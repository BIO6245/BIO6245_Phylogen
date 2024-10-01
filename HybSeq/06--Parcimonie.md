# Analyse de parcimonie sur les données concaténées

Les données HybSeq utilisées ici correspondent à quelques centaines de gènes nucléaires récupérés 
dans sept espèces différentes de fougères du genre *Dryopteris* et d'un genre apparenté, 
*Polystichum*. Tous ces gènes ont une histoire évolutive différente. Toutefois, si l'ensemble de 
ces gènes est analysé en bloc, la phylogénie estimée devrait représenter une bonne 
approximation de l'arbre d'espèces. 

Cette approche est nommée "analyse par concaténation", car elle procède tout d'abord par la 
combinaison (=concaténation) des alignements des différents gènes en une seule matrice, suivi de 
l'analyse de cette matrice "concaténée" comme si c'était l'alignement d'un seul très gros gène.

Ici, nous allons effectuer une analyse de parcimonie sur la matrice concaténée.

---

## Concaténer les alignements

Nous avons des alignements bruts et des alignements filtrés. Les deux seront concaténés pour créer 
deux matrices différentes, de façon évaluer si la filtration a un effet sur le résultat avec ce 
jeu de données.
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca


###
## Concaténer les alignements bruts, avant filtrage
###

ALIGN_PATH=$WD/seqs/exon/align

mkdir -p $ALIGN_PATH/concat
cd $ALIGN_PATH/concat

## Créer un alignement concaténé en format phyml avec catfasta2phyml
TIME="0-3:00:00"
sbatch \
  --job-name=catfasta2phyml \
  --output=catfasta2phyml.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=1 \
  --mem-per-cpu=8G \
  --wrap="$SRC/catfasta2phyml/catfasta2phyml.pl \
  --concatenate $ALIGN_PATH/*.fasta \
  1> raw_concat.phy \
  2> raw_concat.partitions"

## Créer un alignement concaténé en format fasta avec catfasta2phyml
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-3:00:00"
sbatch \
  --job-name=catfasta2phyml \
  --output=catfasta2phyml.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=1 \
  --mem-per-cpu=8G \
  --wrap="$SRC/catfasta2phyml/catfasta2phyml.pl \
  --fasta \
  --concatenate $ALIGN_PATH/*.fasta \
  1> raw_concat.fasta \
  2> /dev/null"

## Créer un alignement concaténé en format nexus avec clustalW
clustalw \
  -convert \
  -infile=raw_concat.fasta \
  -output=nexus \
  -outfile=raw_concat.nex

###
## Concaténer les alignements après filtrage
###

ALIGN_PATH=$WD/seqs/exon/align/trimal

mkdir -p $WD/seqs/exon/align/concat
cd $WD/seqs/exon/align/concat

## Créer un alignement concaténé en format phyml avec catfasta2phyml
TIME="0-3:00:00"
sbatch \
  --job-name=catfasta2phyml \
  --output=catfasta2phyml.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=1 \
  --mem-per-cpu=8G \
  --wrap="$SRC/catfasta2phyml/catfasta2phyml.pl \
  --concatenate $ALIGN_PATH/*.fasta \
  1> filtered_concat.phy \
  2> filtered_concat.partitions"

## Créer un alignement concaténé en format fasta avec catfasta2phyml
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-3:00:00"
sbatch \
  --job-name=catfasta2phyml \
  --output=catfasta2phyml.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=1 \
  --mem-per-cpu=8G \
  --wrap="$SRC/catfasta2phyml/catfasta2phyml.pl \
  --fasta \
  --concatenate $ALIGN_PATH/*.fasta \
  1> filtered_concat.fasta \
  2> /dev/null"

## Créer un alignement concaténé en format nexus avec clustalW
clustalw \
  -convert \
  -infile=filtered_concat.fasta \
  -output=nexus \
  -outfile=filtered_concat.nex

```

Attendre que les tâches précédentes aient toutes terminées. Une fois cela fait, corriger les noms 
des gènes dans les fichiers d'alignement (par défaut, ils incluent le chemin vers l'alignement  
.fasta de chaque gène). Exécuter ces commandes:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest

## ajuster les noms des gènes dans les fichiers de partition
cd $WD/seqs/exon/align/concat
sed -i "s/^.*\//DNA, /g" raw_concat.partitions
sed -i "s/\.fasta//g" raw_concat.partitions
sed -i "s/^.*\//DNA, /g" filtered_concat.partitions
sed -i "s/\.fasta//g" filtered_concat.partitions

```

---

### Analyse de parcimonie

Lancez le code ci-dessous pour faire une analyse de parcimonie avec PAUP\* en créer un fichier 
contenant toutes les commandes à envoyer à PAUP\*, puis en soumettant ce fichier dans une tâche 
sur SLURM:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
ALIGNMENT=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/filtered_concat.nex
OUTPUT_PREFIX=filtered
OUTGROUPS="Polystichum_speciosissimum_SRR14320998"
NREPS_SEARCH=100
SEARCH_PARAMS="multre=yes nchuck=5 chucklen=1"
NREPS_BOOTSTRAP=100
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## Analyse de parcimonie
###

mkdir -p $WD/trees/exon/concat/parsimony
cd $WD/trees/exon/concat/parsimony

## Créer un fichier contenant les commandes à envoyer à PAUP*
echo "
begin paup;

	set autoclose=yes;
	set taxlabels=full;
	set increase=auto autoinc=100 autoclose=yes;
	set warnreset=no warntree=no;

	outgroup $OUTGROUPS;

	hs addseq=random nreps=$NREPS_SEARCH $SEARCH_PARAMS;
	filter best=yes permdel=yes;
	condense collapse=max;
	descr 1;
	savetrees file=$OUTPUT_PREFIX.pars_bests.tre format=nex brlens=yes;
	contre /majrule=no treefile=$OUTPUT_PREFIX.pars_strict.tre;
	
	boots nreps=$NREPS_BOOTSTRAP search=heuristic grpfreq=no treefile=$OUTPUT_PREFIX.pars_bootfile.tre / addseq=random nreps=3 multre=yes steepest=no nchuck=3 chucklen=1 limitperrep=yes;
	savetrees from=0 to=1 file=$OUTPUT_PREFIX.pars_boot.tre format=nex savebootp=nodelab maxdec=0;

	quit;
	
end;" > paup_commands.txt

## Ajouter ce bloc de commandes PAUP à l'alignement pour créer un batchfile pour PAUP*
cat $ALIGNMENT paup_commands.txt > paup_batchfile.nex

## Exécuter ce batchfile en mode non-intéractif sur SLURM
sbatch \
  --job-name=paup \
  --output=paup.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=4 \
  --mem-per-cpu=2G \
  --wrap="/opt/paup4a168_centos64 --noninteractive paup_batchfile.nex"

```

Examiner les résultats avec la commande `more paup.log`. 

- **Question**: Est-ce que l'arbre estimé est similaire 
aux phylogénies déjà publiées sur *Dryopteris*?  
- **Question**: Est-ce que les branches sont bien supportées? Qu'est-ce qu'un bon support?    
- **Question**: Quelle branche a reçu le moins bon support?

---

### Exercices

1. Ré-exécutez l'analyse de parcimonie sur l'alignement concaténé brut, avant le filtrage des 
données. Assurez-vous de modifier aussi le nom des fichiers de sortie. En quoi les résultats 
diffèrent-ils de l'analyse sur l'alignement concaténé filtré?

2. Tentez de modifier les paramètres de la recherche heuristique et de l'analyse bootstrap. 
Qu'est-ce que les modifications ont comme résultats? Qu'est-ce que les différents paramètres 
veulent dire?

