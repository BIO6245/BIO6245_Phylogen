# Assemblage des données HybSeq

[HybPiper](https://github.com/mossmatters/HybPiper) est le pipeline le plus populaire pour assembler des 
données génétiques acquises avec la méthode HybSeq.

## Assembler des données HybSeq avec HybPiper

Assembler les données de tous les échantillons du jeu de données test:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
READS_PATH=/scratch/$USER/HybSeqTest/reads/trim
WD=/scratch/$USER/HybSeqTest
TARGETS=/data/hybseqRefs/GoFlag_targets.fa
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

mkdir -p $WD
cd $WD

## Créer une batch file pour l'assemblage des données avec HybPiper
echo '#!/bin/bash' > hybpiper.sbatch
echo "#SBATCH --job-name=hybpiper
#SBATCH --output=hybpiper-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

READ1=\$(ls $READS_PATH/*trim_R1.fastq.gz | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
NAME=\$(basename \$READ1 _trim_R1.fastq.gz)

hybpiper assemble \\
  --cpu 8 \\
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

Pendant que vous attendez les résultats de cette analyse, lisez le wiki de HybPiper pour mieux comprendre
les étapes de cette analyse.

### Contrôle qualité de l'assemblage HybPiper

La commande `hybpiper stats` donnera quelques statistiques récapitulatives sur l'assemblage HybPiper. 
Cela nous oblige à spécifier une liste de séquences de sondes et une liste de noms d'échantillons. La 
commande `hybpiper recovery_heatmap` prend le résultat de `hybpiper stats` et génère une carte thermique 
(heatmap) pour évaluer visuellement dans quelle mesure chaque gène a été récupéré dans chaque échantillon 
(en termes de longueur du contig le plus long).

Voici un script pour générer des statistiques en utilisant `hybpiper stats`:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
TARGETS=/data/hybseqRefs/GoFlag_targets.fa

cd $WD

## Créer une liste des échantillons assemblés avec HybPiper
ls -d *RR*/ | cut -f1 -d'/' > samplelist.txt


## Exécuter hybpiper_stats
conda activate hybpiper
nohup hybpiper stats -t_dna $TARGETS gene samplelist.txt > hybpiper_stats.log &

```

Une fois la tâche précédente terminée, créer une heatmap de récupération avec `hybpiper recovery_heatmap`:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
TARGETS=/data/hybseqRefs/GoFlag_targets.fa

cd $WD

## Exécuter recovery_heatmap
nohup hybpiper recovery_heatmap --heatmap_dpi 300 --heatmap_filetype pdf seq_lengths.tsv > hybpiper_heatmap.log &

```

Une fois cela fait, télécharger et visualiser le heatmap qui est sauvegardé dans le fichier 
`recovery_heatmap.pdf`.

- **Question:** Comment interprète-t-on ce graphique?

- **Question:** Quels échantillons ont le mieux fonctionnés (plus de séquences assemblées de meilleure 
qualité)? Lesquels ont moins bien fonctionnés?

- **Question:** Quelles peuvent être les raisons d'une moins bonne récupération des séquences chez un ou 
quelques échantillons? Trouver au moins deux raisons techniques, et deux raisons biologiques.








!!!! Ignorer le code ci-dessous !!!!

!!!! Code ci-dessous pas encore roulé et ajusté pour le cours !!!!











### Investigating problems of paralogy

Sometimes, one of the genes targeted by the HybSeq probes is duplicated in one or a few of your samples. 
If the age of the duplication is young enough, it is quite likely that both copies of the gene are similar 
enough to each other and the probe sequence that they will both be "fished" by the probes. The reads of 
both copies are then caught by HybPiper and assembled in different contigs (because they are not identical).

By default, HybPiper identifies the "best" contig for each gene by coverage (10x more coverage than 
other contigs) and length (>=75% of the target sequence is present). If multiple contigs have high 
coverage and length, they likely each represent a different copy of a duplicated gene (i.e., they are 
paralogs), or different alleles of the same gene.

To check how many potential paralogs/alternative alleles are present in each sample, run the script 
below:
```bash
WD=~/scratch/HybSeqTest
TARGETS=/project/def-bourret/shared/hybseqRefs/GoFlag_targets.fa

cd $WD

conda activate hybpiper

## create a report on the number of potential paralogs
## run in the background because it takes a bit of time
nohup hybpiper paralog_retriever samplelist.txt -t_dna $TARGETS > paralog_retriever.log &
tail -f paralog_retriever.log

```

Once done, download the file `paralog_report.tsv` to your local computer, and examine it in excel.
- How many loci are single-copy across all samples?
- How many loci possibly contain paralogs?
- Are there any loci with paralogs in every sample, suggesting an ancient gene duplication shared by 
all species?

HybPiper selects for each locus one contig that it considers the "best representative" for the contig,
 based on coverage, length and similarity to the probe sequence. However, it is best to manually check 
 all the gene trees to make sure that HybPiper has not made any error.

For each locus, align all the contigs retrieved by paralog retriever using 
[MAFFT](https://mafft.cbrc.jp/alignment/software/about.html) and generate a quick gene phylogeny 
using [FastTree](http://www.microbesonline.org/fasttree/):  
```bash
WD=~/scratch/HybSeqTest
EMAIL=votre.courriel@umontreal.ca

mkdir -p $WD/trees/paralogs
cd $WD/trees/paralogs

## loop over all loci, align with mafft, pipe to FastTree for quick rough phylogeny
echo '#!/bin/bash' > quickParaTree.sbatch
echo "#SBATCH --job-name=quickParaTree
#SBATCH --output=quickParaTree.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem-per-cpu=2G
#SBATCH --time=0-3:00:00

echo \"#NEXUS
begin trees;\" > allTrees.nex
for t in \$(ls -d $WD/paralogs_all/*.fasta)
  do
    GENENAME=\$(basename \$t .fasta)
    mafft --localpair \$t | FastTree -nt -gtr > \$GENENAME.tre
    echo \"tree \$GENENAME = \$(<\$GENENAME.tre)\" >> allTrees.nex
    rm \$GENENAME.tre
  done
echo \"end;\" >> allTrees.nex
rm -r maffttmp" >> quickParaTree.sbatch

## run the loop in the background, save screen output to log file
module load StdEnv/2020
module load mafft/7.471
module load fasttree/2.1.11
sbatch --mail-user=$EMAIL quickParaTree.sbatch

```

Once done, download `allTrees.nex` to your local computer and visually sift through the trees 
using [FigTree](http://tree.bio.ed.ac.uk/software/figtree/) or another tree viewer, focusing in 
particular on loci that have many paralogs identified for many samples. 
- Is there evidence that allelic contigs were retrieved? 
- Is there evidence that paralogs were retrieved?


