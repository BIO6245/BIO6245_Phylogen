# Contrôle qualité des données Illumina

Les données de type Illumina contiennent souvent de nombreux types d'erreurs 
et d'artefacts, notamment des données de mauvaise qualité, ainsi que des 
données provenant des adaptateurs d'ADN synthétiques qui sont ajoutés lors de 
la préparation de la bibliothèque.

Ici, l'objectif sera d'apprendre à utiliser des outils standards pour évaluer 
la qualité des données que nous avons en main, et aussi pour éliminer les 
séquences qui sont probablement erronées, de façon à réduire la complexité des 
analyses subséquentes tout en augmentant la qualité des résultats obtenus.

Nous utiliserons des données test générées par Étienne Lacroix-Carignan et 
Maurane Bourgouin en 2024-2025 sur des échantillons de *Carex*. Les données 
test sont localisées à cet endroit sur le serveur *aphidzen*: 
`/data/testdata/HybSeq_Carex/`.

---

## Rouler FastQC sur les données brutes

Les étapes suivantes se feront toutes sur l'espace `/scratch` étant donné 
qu'il ne s'agit pas de données (qui doivent être entreposées sur `/data`), 
mais d'analyses de données qui doivent se faire sur le `/scratch`. Pour ne pas 
avoir à faire des copies des données Illumina, nous allons faire un "lien" 
vers ces données avec la commande `ln -s`. Nous allons ensuite effectuer un 
premier contrôle qualité à l'aide de [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).

Lancer les commandes ci-dessous pour rouler FastQC sur les données test 
*Carex*:   
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_FASTQC=/opt/FastQC
WD=/scratch/$USER/HybSeqTest
READS_PATH=/data/testdata/HybSeq_Carex

mkdir -p $WD/reads
cd $WD/reads

## faire des liens symboliques (ln -s) vers les fichiers de données Illumina
ln -s $READS_PATH/*.fastq.gz .

## s'assurer que tous les fichiers terminent par _R01 ou _R02
rename s/_001// *.fastq.gz
rename s/_1.fastq.gz/_R1.fastq.gz/ *.fastq.gz
rename s/_2.fastq.gz/_R2.fastq.gz/ *.fastq.gz

## créer dossier pour premier contrôle qualité des données Illumina brutes
mkdir -p $WD/reads/qc_before
cd $WD/reads/qc_before

## déterminer combien de fichiers .fastq.gz il faut analyser
NFILES=$(ls -1 $WD/reads/*.fastq.gz | wc -l)

## lancer les analyses FastQC sur SLURM
sbatch --job-name=fastQC \
  --output=fastQC-%a.out \
	--array=1-$NFILES \
  --time=4:00:00 \
  --wrap "
	  SAMPLE=\$(ls $WD/reads/*.fastq.gz | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
		$SRC_FASTQC/fastqc --outdir . \$SAMPLE"

```

Pendant que l'analyse FastQC progresse, vous pouvez lire la documentation sur 
le site web de FastQC vous aider à interpréter les résultats de cette analyse.

Ensuite, lorsque FastQC a terminé d'analyser chaque échantillon, il faut 
télécharger les fichiers sortie de FastQC et les visualer sur son ordinateur 
local pour évaluer la qualité des données brutes.

Pour télécharger les données sur votre machine locale, **exécutez la** 
**commande ci-dessous à partir de votre machine locale, et non pas du** 
**serveur.** N'oubliez pas d'ajuster les chemins vers les fichiers.  
```bash
## Rouler le code ci-dessous à partir de votre machine locale

## Ajuster les variables ci-dessous de façon appropriée
CLUSTER_USERNAME=elbourret
LOCAL=/mnt/c/Users/Etienne/Downloads
REMOTE=/scratch/$CLUSTER_USERNAME/HybSeqTest/reads/qc_before

rsync --progress --recursive $CLUSTER_USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE $LOCAL/

```

Une fois les fichiers FastQC téléchargés, visualisez-les. Chaque fichier 
.fastq.gz a été analysé séparément par FastQC pour créer un rapport sur la 
qualité des données. Visualisez chaque rapport un par un. À l'intérieur de 
chaque rapport, regardez chacune des sections. **Vous devez bien comprendre**  
**ce que signifie chacune des sections. SVP, ne vous gênez pas de poser des** 
**questions au professeur pour bien comprendre, même si la question a déjà** 
**été posée.**

Notez que si vous êtes sur Windows, il faut remplacer les "backslash" (\\) 
dans votre chemin avec des "forward slash" (\/). De plus, le chemin "C:/" doit 
être remplacé par "/mnt/c/". Finalement, notez que les espaces dans les noms 
des dossiers et fichiers causent problème sur tous les terminal Linux/Mac. Il 
y a deux solutions pour les espaces: soit ne jamais utiliser d'espaces dans 
vos noms de fichier, ou soit entourer les espaces par des guillemets simples 
('), ou précéder chaque espace par un "backslash" (\\) lorsque vous spécifiez 
un chemin vers un dossier ou fichier.  
Par exemple:  
- Le chemin Windows: `C:\Users\Moi\Documents\BIO 6245\ficher de travail.txt`  
- Doit être remplacé par: `/mnt/c/Users/Moi/Documents/BIO' '6245/fichier' 'de' 'travail.txt`  
- Ou bien par: `/mnt/c/Users/Moi/Documents/BIO\ 6245/fichier\ de\ travail.txt`

---

## Dédupliquer, rogner et filtrer les données Illumina

Les séquences Illumina (ci-après nommées "lectures", traduction de "reads") 
contiennent des erreurs qu'il faut éliminer pour éviter d'introduire ces 
erreurs dans les données finales que nous analyseront dans ce cours.

Pour préparer les échantillons pour le séquençage Illumina, on utilise 
généralement de nombreux cycles d'amplification PCR. Cela mène à des lectures 
exactement identiques, qu'on nomme "duplicats", qui n'apportent pas 
d'information supplémentaire et peuvent causer certains biais lors de 
l'analyse. Ces duplicats PCR seront retirés, par un processus qu'on nomme la 
"déduplication", à l'aide du programme 
[htStream_SuperDeduper](https://s4hts.github.io/HTStream/).

On doit ensuite retirer les nucléotides de faible qualité dans les lectures 
restantes, ce qu'on nomme le "rognage". Le processus utilise les scores de 
qualité de séquence offerts par Illumina pour éliminer les bases qui ont une 
plus faible qualité. On obtient alors des lectures qui sont plus courtes, 
mais demeilleure qualité en moyenne. Toutefois, si certaines lectures sont de 
très mauvaises qualité, le rognage réduira tellement la longueur de la 
lecture qu'elle ne sera plus d'aucune utilité. Par exemple, si une lecture ne 
fait que 3-4pb de longueur, ce ne sera pas utilisable car impossible à aligner 
étant donné quec'est trop court. Il faut alors "filtrer" les lectures qui sont 
trop courtes après le rognage. Nous utiliserons le très populaire programme 
[trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) pour le rognage 
et la filtration des lectures Illumina.

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
READ1=\$(ls $READS_PATH/*_R1*.fastq.gz | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
NAME=\$(basename \$READ1 _R1.fastq.gz)
READ2=\$(echo \"\${NAME}_R2.fastq.gz\")
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
NFILES=$(ls -1 $READS_PATH/*1.fastq.gz | wc -l)

## Soumettre ce fichier batch à SLURM
sbatch --mail-user=$EMAIL --array=1-$NFILES QCnTrim.sbatch

```

Une fois cela fait, rouler encore une fois FastQC sur les fichiers dédupliqués 
(dans le dossier `./dedup/`) et sur les fichiers finaux qui sont aussi rognés 
(dans le dossier `./trim/`).

- **Question:** Quelles sont les différences entre les données brutes, les 
données dédupliquées, et les données finales qui sont dédupliquées, rognées 
et filtrées?

- **Question:** Était-ce utile de dédupliquer, rogner et filtrer les données? 
Pourquoi?

