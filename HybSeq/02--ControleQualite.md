# Contrôle qualité des données Illumina

## Rouler FastQC sur les données brutes

Les étapes suivantes se feront toutes sur l'espace `/scratch` étant donné qu'il ne s'agit pas de données 
(qui doivent être entreposées sur `/data`), mais d'analyses de données qui doivent se faire sur le 
`/scratch`. Pour ne pas avoir à faire des copies des données Illumina, nous allons faire un "lien" 
vers ces données avec la commande `ln -s`. Nous allons ensuite effectuer un premier contrôle qualité 
à l'aide de [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
READS_PATH=/data/$USER/HybSeqTest/reads

mkdir -p $WD/reads
cd $WD/reads

## faire des liens symboliques (ln -s) vers les fichiers de données Illumina
ln -s $READS_PATH/*.fastq.gz .

## faire un premier contrôle qualité des données Illumina brutes
mkdir -p $WD/reads/qc_before
nohup /opt/FastQC/fastqc \
  *.fastq.gz \
  --outdir qc_before > ./qc_before/fastqc.log &

## suivre le progrès de FastQC
tail -f ./qc_before/fastqc.log

```

Pendant que l'analyse FastQC progresse, vous pouvez lire la documentation sur le site web de FastQC 
vous aider à interpréter les résultats de cette analyse.

Ensuite, lorsque FastQC a terminé d'analyser chaque échantillon, il faut télécharger les fichiers sortis 
de FastQC et les visualer sur son ordinateur local pour évaluer la qualité des données brutes.

Pour télécharger les données sur votre machine locale, **exécutez la commande ci-dessous à partir de** 
**votre machine locale, et non pas du serveur.** N'oubliez pas d'ajuster les chemins vers les fichiers.  
```bash
## Rouler le code ci-dessous à partir de votre machine locale

## Ajuster les variables ci-dessous de façon appropriée
USERNAME=elbourret
LOCAL=/mnt/c/Users/p0948315/Downloads
REMOTE=/scratch/$USERNAME/HybSeqTest/reads/qc_before

rsync --progress --recursive $USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE $LOCAL/

```

Une fois les fichiers FastQC téléchargés, visualisez-les. Chaque fichier .fastq.gz a été analysé 
séparément par FastQC pour créer un rapport sur la qualité des données. Visualisez chaque rapport un 
par un. À l'intérieur de chaque rapport, regardez chacune des sections. **Vous devez bien comprendre** 
**ce que signifie chacune des sections. SVP, ne vous gênez pas de poser des questions au professeur pour** 
**bien comprendre, même si la question a déjà été posée.**

## Dédupliquer, rogner et filtrer les données Illumina

Les séquences Illumina (ci-après nommées "lectures", traduction de "reads") contiennent des erreurs 
qu'il faut éliminer pour éviter d'introduire ces erreurs dans les données finales que nous analyseront 
dans ce cours.

Pour préparer les échantillons pour le séquençage Illumina, on utilise généralement de nombreux cycles 
d'amplification PCR. Cela mène à des lectures exactement identiques, qu'on nomme "duplicats", qui 
n'apportent pas d'information supplémentaire et peuvent causer certains biais lors de l'analyse. Ces
duplicats PCR seront retirés, par un processus qu'on nomme la "déduplication", à l'aide du programme 
[htStream_SuperDeduper](https://s4hts.github.io/HTStream/).

On doit ensuite retirer les nucléotides de faible qualité dans les lectures restantes, ce qu'on nomme 
le "rognage". Le processus utilise les scores de qualité de séquence offerts par Illumina pour éliminer 
les bases qui ont une plus faible qualité. On obtient alors des lectures qui sont plus courtes, mais de
meilleure qualité en moyenne. Toutefois, si certaines lectures sont de très mauvaises qualité, le rognage 
réduira tellement la longueur de la lecture qu'elle ne sera plus d'aucune utilité. Par exemple, si une 
lecture ne fait que 3-4pb de longueur, ce ne sera pas utilisable car impossible à aligner étant donné que
c'est trop court. Il faut alors "filtrer" les lectures qui sont trop courtes après le rognage. Nous 
utiliserons le très populaire programme [trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) pour 
le rognage et la filtration des lectures Illumina.

Voici le code pour effectuer toutes ces étapes (déduplication, rognage, filtration) sur tous les échantillons:
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
READS_PATH=/scratch/$USER/HybSeqTest/reads
EMAIL=votre.adresse@umontreal.ca
ADAPTER=TruSeq3-PE-2.fa

cd $READS_PATH

## Créer un fichier batch qui sera utilisé pour soumettre une tâche à SLURM
echo '#!/bin/bash' > QCnTrim.sbatch
echo "#SBATCH --job-name=QCnTrim
#SBATCH --output=QCnTrim-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=8G
#SBATCH --time=0-10:00:00

## Initialiser les variables internes du fichier batch
READ1=\$(ls $READS_PATH/*_1.fastq.gz | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
NAME=\$(basename \$READ1 _1.fastq.gz)
READ2=\$(echo \"\${NAME}_2.fastq.gz\")
WD=\$(pwd)

## Créer des sous-dossiers pour contenir les données dédupliquées...
mkdir -p \$WD/dedup

## ...et pour contenir les données rognées
mkdir -p \$WD/trim

## Filtrer les duplicats PCR (=dédupliquer) avec la fonction SuperDeduper de HTStream
$SRC/HTStream/build/hts_SuperDeduper/hts_SuperDeduper \\
  -1 \$READ1\\
  -2 \$READ2 \\
  -s 1 \\
  -l 99 \\
  --stats-file \$WD/dedup/dedup_stats_\$SLURM_ARRAY_TASK_ID.log \\
  -f \$WD/dedup/\${NAME}_dedup
  
## Rogner les séquences d'adapteur et les nucléotides de faible qualité
cd \$WD/trim
java -jar $SRC/Trimmomatic-0.39/trimmomatic-0.39.jar PE -phred33 \\
   \$WD/dedup/\${NAME}_dedup_R1.fastq.gz \\
   \$WD/dedup/\${NAME}_dedup_R2.fastq.gz \\
   \${NAME}_trim_R1.fastq.gz \${NAME}_unpaired_R1.fastq.gz \\
   \${NAME}_trim_R2.fastq.gz \${NAME}_unpaired_R2.fastq.gz \\
   ILLUMINACLIP:/opt/Trimmomatic-0.39/adapters/$ADAPTER:2:30:10:8:TRUE \\
   LEADING:3 \\
   TRAILING:3 \\
   SLIDINGWINDOW:4:15 \\
   MINLEN:35" >> QCnTrim.sbatch

## Déterminer combien de fichiers il faut filtrer et rogner
NFILES=$(ls -1 $READS_PATH/*_1.fastq.gz | wc -l)

## Soumettre ce fichier batch à SLURM
sbatch --mail-user=$EMAIL --array=1-$NFILES QCnTrim.sbatch

```

Une fois cela fait, rouler encore une fois FastQC sur les fichiers dédupliqués (dans le dossier `./dedup/`) 
et sur les fichiers finaux qui sont aussi rognés (dans le dossier `./trim/`). 

- **Question:** Quelles sont les différences entre les données brutes, les données dédupliquées, et les 
données finales qui sont dédupliquées, rognées et filtrées?

- **Question:** Était-ce utile de dédupliquer, rogner et filtrer les données? Pourquoi?

