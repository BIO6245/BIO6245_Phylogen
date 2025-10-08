# Estimation d'un arbre d'espèces

Ce tutoriel vous montrera comment estimer un arbre d'espèces à partir des données provenant de 
plusieurs gènes différents, via l'estimation d'arbres de gènes. Il existe deux grandes approches: 
1. Co-estimation des arbres de gènes et de l'arbre d'espèce (p.ex., **\*BEAST** (=**starBEAST**)
2. Estimation des arbres de gènes individuels, puis estimation d'un arbre d'espèce avec une 
méthode par abrégé (p.ex., **ASTRAL**, **svdQuartets**, etc.)

Malheureusement, la co-estimation des arbres de gènes et de l'arbre d'espèce avec \*BEAST demande 
beaucoup trop de ressources de calcul pour être faisable avec un jeu de données génomiques comme 
le HybSeq. C'est efficace uniquement pour un jeu de données de 10 à 50 gènes maximum. Pour des 
centaines de gènes, il faut utiliser des méthodes par abrégé.

---

## Estimation par abrégé avec ASTRAL

L'estimation d'un arbre par une méthode par abrégé comme ASTRAL se fait en deux étapes. Tout 
d'abord, il faut estimer les phylogénies des gènes individuels. Ensuite, il faut fournir ces 
phylogénies à ASTRAL pour estimer l'arbre d'espèce à partir de toutes ces différentes phylogénies.

Pour plus d'information sur ASTRAL, visitez le github [ici](https://github.com/chaoszhang/ASTER).

Notez que ASTRAL est une des multiples méthodes d'estimation phylogénétique disponible dans le 
paquet ASTER, qui contient aussi d'autres approches d'analyse phylogénétique.

### Estimation des arbres de gènes individuels avec RAxML

Le code ci-dessous estimera individuellement la phylogénie de chacun des exon qui a été récupéré 
dans le jeu de données HybSeq. Chaque analyse se fera dans une job SLURM séparée à l'aide d'un 
"job array".  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_CATFASTA=/opt/catfasta2phyml
SRC_RAXML=/opt/RAxML-8.2.12
WD=/scratch/$USER/HybSeqTest/trees/exon/raxml
ALIGNMENTS=/scratch/$USER/HybSeqTest/seqs/exon/align/trimal/*.fasta
MODEL=GTRCAT
BOOTSTRAP_REPS=20
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"
THREADS=2


## préparer l'espace de travail
mkdir -p $WD
cd $WD

## Créer un batchfile pour SLURM, dans lequel on estime l'arbre de chaque exon
echo '#!/bin/bash' > raxml-genetrees.sbatch
echo "#SBATCH --job-name=raxml-genetrees
#SBATCH --output=raxml-genetrees-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$THREADS
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

## aller chercher l'alignement numéro "SLURM_ARRAY_TASK_ID"
ALIGNIN=\$(ls $ALIGNMENTS | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENENAME=\$(basename \$ALIGNIN .fasta)

## pour cet alignement, transformer le format .fasta en .phy
$SRC_CATFASTA/catfasta2phyml.pl \\
  --concatenate \$ALIGNIN \\
  1> \$GENENAME.phy \\
  2> \$GENENAME.partitions

## corriger le formattage du fichier de partitions
sed -i \"s/^.*\\//DNA, /g\" \$GENENAME.partitions
sed -i \"s/\\.fasta//g\" \$GENENAME.partitions

## ensuite, estimer l'arbre de gène avec RAxML
$SRC_RAXML/raxmlHPC-PTHREADS-SSE3 \\
    -f a \\
    -T $THREADS \\
	-p 1234 \\
	-x 1234 \\
	-m $MODEL \\
	-# $BOOTSTRAP_REPS \\
    -s \$GENENAME.phy \\
    -q \$GENENAME.partitions \\
	-n \$GENENAME" >> raxml-genetrees.sbatch

## Déterminer le nombre de loci à analyser dans des sous-tâches séparées de SLURM
NFILES=$(ls -1 $ALIGNMENTS | wc -l)

## Envoyer les tâches d'alignement à SLURM
sbatch --mail-user=$EMAIL --array=1-$NFILES raxml-genetrees.sbatch

```

Après avoir lancé les tâches ci-dessus, attendre qu'elles finissent toutes. Vérifier quelques 
arbres de gènes manuellement en les téléchargeant sur votre ordinateur et en les visualisant à 
l'aide de FigTree. Sont-ils tous identiques? Y a-t-il des éléments de ressemblance?

### Estimation de l'arbre d'espèces avec ASTER

Il existe plusieurs "variétés" d'algorithmes ASTRAL, chacun avec des avantages et inconvénients:
- ASTRAL: l'algorithme standard, qui prend en compte seulement la topologie des arbres de gènes, 
sans évaluer les longueurs de branches et le support statistique.
- ASTRAL-Pro: permet d'utiliser des arbres de gènes qui contiennent des duplications (arbres de 
famille de gènes avec plusieurs copies dans chaque espèce).
- weighted ASTRAL ou wASTRAL: comme l'algorithme standard, mais pondère la valeur de chaque 
quartet en fonctionne de la longueur des branches et/ou support bootstrap associé, ce qui fait 
que les clades qui ont un faible support dans les arbres de gènes auront moins d'influence sur le 
résultat final de l'arbre d'espèce. Ça donne des résultats plus fiables.
- CASTER: méthode qui part des alignements directement.
- WASTER: méthode qui part des séquences non alignées.

Nous allons utiliser la méthode wASTRAL, qui est la plus fiable lorsqu'on a estimé des arbres de 
gènes avec des valeurs de support bootstrap.



```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_ASTER=/opt/ASTER/bin
WD=/scratch/$USER/HybSeqTest/trees/exon/wastral
ARBRES=/scratch/$USER/HybSeqTest/trees/exon/raxml/RAxML_bipartitions.*
OUTGROUP=Polystichum_speciosissimum_SRR14320998
OUTPUT_PREFIX=Dryopteris
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"


## créer le dossier où on sauvegardera les résultats
mkdir -p $WD
cd $WD

## concaténer tous les arbres dans un seul fichier
cat $ARBRES > geneTrees.tre

## effectuer l'analyse
$SRC_ASTER/wastral \
  -i geneTrees.tre \
  -o $OUTPUT_PREFIX.species.tre \
  --root $OUTGROUP

```

Téléchargez l'arbre d'espèces qui a été estimé et visualisez-le avec FigTree. Comparez-le avec 
l'arbre d'espèces estimé dans des analyses récentes du genre *Dryopteris*. Est-ce que votre 
estimation parait fiable?

