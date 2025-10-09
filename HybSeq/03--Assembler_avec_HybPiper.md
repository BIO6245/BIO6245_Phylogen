# Assemblage des données HybSeq

[HybPiper](https://github.com/mossmatters/HybPiper) est le pipeline le plus 
populaire pour assembler des données génétiques acquises avec la méthode 
HybSeq.

Dans ce tutoriel, vous allez apprendre à utiliser **HybPiper**.

Le tutoriel assume que vous avez déjà fait le contrôle de la qualité des 
séquences, et donc que vos séquences sont rognées. Il est mal avisé 
d'assembler des données qui n'ont pas passé le contrôle qualité; cela peut 
créer des erreurs d'assemblage et réduire l'efficacité de cette étape de 
l'analyse.

---

## Assembler des données HybSeq avec HybPiper

Assembler les données de tous les échantillons du jeu de données test:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
READS_PATH=/scratch/$USER/HybSeqTest/reads/trim
WD=/scratch/$USER/HybSeqTest
TARGETS=/data/hybseqRefs/combined_Mega353_Carex554.fa
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"
CPUS=4
MEM_PER_CPU=2G

mkdir -p $WD
cd $WD

## Créer une batch file pour l'assemblage des données avec HybPiper
echo '#!/bin/bash' > hybpiper.sbatch
echo "#SBATCH --job-name=hybpiper
#SBATCH --output=hybpiper-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CPUS
#SBATCH --mem-per-cpu=$MEM_PER_CPU
#SBATCH --time=$TIME

READ1=\$(ls $READS_PATH/*trim_R1.fastq.gz | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
NAME=\$(basename \$READ1 _trim_R1.fastq.gz)

hybpiper assemble \\
  --cpu $CPUS \\
  -t_dna $TARGETS \\
  -r $READS_PATH/\${NAME}_trim_R*.fastq.gz \\
  --prefix \$NAME \\
  --bwa" >> hybpiper.sbatch

## Déterminer le nombre d'échantillons à analyser
NFILES=$(ls -1 $READS_PATH/*trim_R1.fastq.gz | wc -l)

## envoyer les tâches d'assemblage à SLURM
conda activate hybpiper
sbatch --mail-user=$EMAIL --array=1-$NFILES hybpiper.sbatch

```

Pendant que vous attendez les résultats de cette analyse, lisez le wiki de HybPiper pour mieux comprendre les étapes de cette analyse.

---

### Contrôle qualité de l'assemblage HybPiper

La commande `hybpiper stats` donnera quelques statistiques récapitulatives sur 
l'assemblage HybPiper. Cela nous oblige à spécifier une liste de séquences de 
sondes et une liste de noms d'échantillons. La commande 
`hybpiper recovery_heatmap` prend le résultat de `hybpiper stats` et génère 
une carte thermique (heatmap) pour évaluer visuellement dans quelle mesure 
chaque gène a été récupéré dans chaque échantillon (en termes de longueur du 
contig le plus long).

Voici un script pour générer des statistiques en utilisant `hybpiper stats`:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
TARGETS=/data/hybseqRefs/combined_Mega353_Carex554.fa

cd $WD

## Créer une liste des échantillons assemblés avec HybPiper
ls -d Carex*/ C_*/ | cut -f1 -d'/' > samplelist.txt

## Exécuter hybpiper_stats
conda activate hybpiper
sbatch --job-name=hybpiper_stats \
  --output=hybpiper_stats.out \
  --time=2:00:00 \
	--cpus-per-task=1 \
	--mem-per-cpu=4G \
  --wrap "
		hybpiper stats -t_dna $TARGETS gene samplelist.txt
		hybpiper recovery_heatmap --heatmap_dpi 300 --heatmap_filetype pdf seq_lengths.tsv
		"

```

Une fois cela fait, télécharger et visualiser le heatmap qui est sauvegardé 
dans le fichier `recovery_heatmap.pdf`.

- **Question:** Comment interprète-t-on ce graphique?

- **Question:** Quels échantillons ont le mieux fonctionnés (plus de séquences 
assemblées de meilleure qualité)? Lesquels ont moins bien fonctionnés?

- **Question:** Quelles peuvent être les raisons d'une moins bonne 
récupération des séquences chez un ou quelques échantillons? Trouver au moins <deux raisons techniques, et deux raisons biologiques.
