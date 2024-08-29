# Alignement des séquences

## Loci nucléaires assemblés par HybPiper

### Alignement des séquences

Récupérons d'abord les séquences d'exons, d'introns et de supercontigs avec HybPiper afin de pouvoir les 
aligner ultérieurement :
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
TARGETS=/data/hybseqRefs/GoFlag_targets.fa

## Aller chercher les séquences des exons de chaque échantillon
mkdir -p $WD/seqs/exon
cd $WD/seqs/exon
conda activate hybpiper
nohup hybpiper retrieve_sequences \
  -t_aa $TARGETS \
  --sample_names $WD/samplelist.txt \
  --fasta_dir . \
  --hybpiper_dir $WD \
  dna > retrieve_seqs.log &

## Aller chercher les séquences des supercontigs de chaque échantillon
mkdir -p $WD/seqs/supercontig
cd $WD/seqs/supercontig
conda activate hybpiper
nohup hybpiper retrieve_sequences \
  -t_aa $TARGETS \
  --sample_names $WD/samplelist.txt \
  --fasta_dir . \
  --hybpiper_dir $WD \
  supercontig > retrieve_seqs.log &

## Aller chercher les séquences des introns de chaque échantillon
mkdir -p $WD/seqs/intron
cd $WD/seqs/intron
conda activate hybpiper
nohup hybpiper retrieve_sequences \
  -t_aa $TARGETS \
  --sample_names $WD/samplelist.txt \
  --fasta_dir . \
  --hybpiper_dir $WD \
  intron > retrieve_seqs.log &

```

Une fois que les séquences ont été récupérées avec HybPiper, retirer les loci représentés par <4 
échantillons puisque ces loci ne contiennent pas d'information phylogénétique importante:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest

## Trouver les loci avec <4 échantillons et les supprimer
cd $WD/seqs
for i in $(ls ./*/*.fasta ./*/*.FNA)
  do
    NSAMPLES=$(grep '>' $i | wc -l)
    if [ $NSAMPLES -lt 4 ]
      then
        echo "$i contains $NSAMPLES samples, which is <4, so it was removed!"
        rm $i
    else
	  echo "$i contains $NSAMPLES samples."
    fi
  done >> filter_min4.log

```

Une fois l'étape précédente terminée, examiner le résultat de ce filtrage en exécutant 
`more filter_min4.log`.

- **Question:** Pourquoi est-ce que ça prend au minimum 4 échantillons dans un alignement pour pouvoir 
estimer un arbre de relations phylogénétiques?

- **Question:** Est-ce que la majorité des données sont exoniques ou introniques? Pourquoi?


Puisque très peu de données introniques ont été récupérées, ce n'est pas très utile d'aligner ces données 
ou celles des supercontigs. Nous allons donc continuer le travail uniquement avec les données exoniques.







!!! CONTINUER ICI !!!








Aligner tous les loci restants après le filtrage précédent avec MAFFT et effectuer des estimations rapides 
de l'arbre phylogénétique avec FastTree:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## aligner les exons
###

SEQS_PATH=$WD/seqs/exon
TREES_PATH=$WD/trees/exon
mkdir -p $SEQS_PATH/align
mkdir -p $TREES_PATH/fastTree
cd $SEQS_PATH/align

## create a SLURM batchfile to align with MAFFT and run FastTree on each locus in an array
echo '#!/bin/bash' > mafft-fasttree.sbatch
echo "#SBATCH --job-name=mafft-fasttree
#SBATCH --output=mafft-fasttree-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

SEQIN=\$(ls $SEQS_PATH/*.fasta $SEQS_PATH/*.FNA | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENENAME=\$(basename \$SEQIN .FNA)

mafft --genafpair --maxiterate 1000 \$SEQIN > \$GENENAME.fasta

FastTree -nt -gtr \$GENENAME.fasta > $TREES_PATH/fastTree/\$GENENAME.tre" >> mafft-fasttree.sbatch

## Déterminer le nombre de loci à analyser dans des sous-tâches séparées de SLURM
NFILES=$(ls -1 {$SEQS_PATH/*.fasta,$SEQS_PATH/*.FNA} 2>/dev/null | wc -l)

## Envoyer les tâches d'alignement à SLURM
conda activate base
sbatch --mail-user=$EMAIL --array=1-$NFILES mafft-fasttree.sbatch

```

### Corriger les noms des échantillons dans les alignements

Les noms de séquence dans les alignements contiennent un commentaire lié au nombre de contigs qui ont été 
cousus pour créer la séquence finale. Ceux-ci doivent être supprimés, sinon le même échantillon porte un 
nom différent dans chaque locus, en fonction du nombre de contigs qui ont été assemblés pour créer la 
séquence finale de chaque locus.  
```
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest

###
## corriger les noms d'échantillons des exons
###

cd $WD/seqs/exon/align

## conserver une copie des alignements bruts
mkdir -p ./raw
cp *.fasta ./raw/

## homogénéiser les noms des échantillons à travers les loci
sed -i 's/ multi_.*/ /g' *.fasta
sed -i 's/ single_.*/ /g' *.fasta

```


!!! Continuer à ajuster le code à partir d'ici !!!



## Filtering low-confidence regions of alignments

First, we will use TrimAl to remove regions of alignments that are too gappy, 
```bash
WD=/home/bourret/scratch/Villaverde_reanalysis
EMAIL=votre.courriel@umontreal.ca
WINDOW_SIZE=3
MIN_NONGAP_PERCENT=0.1
MIN_SIMILARITY=0.1
MIN_OVERLAP=0.5

###
## Filter exon alignments
###

ALIGNS_PATH=$WD/seqs/exon/align

mkdir -p $ALIGNS_PATH/filter/trimal
cd $ALIGNS_PATH/filter

## creating a batchfile to trim alignments
echo '#!/bin/bash' > trimal.sbatch
echo "#SBATCH --job-name=trimal
#SBATCH --output=trimal-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=0-3:00:00

ALIGNIN=\$(ls $ALIGNS_PATH/*.fasta | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENENAME=\$(basename \$ALIGNIN .fasta)

trimal \\
  -in \$ALIGNIN \\
  -out ./trimal/\$GENENAME.fasta \\
  -w $WINDOW_SIZE \\
  -gapthreshold $MIN_NONGAP_PERCENT \\
  -simthreshold $MIN_SIMILARITY \\
  -resoverlap $MIN_OVERLAP \\
  -seqoverlap $MIN_OVERLAP" >> trimal.sbatch

## run trimAl
module purge
module load StdEnv/2020
module load trimal/1.4
NFILES=$(ls -1 $ALIGNS_PATH/*.fasta | wc -l)
sbatch --mail-user=$EMAIL --array=1-$NFILES trimal.sbatch


###
## Filter intron alignments
###

ALIGNS_PATH=$WD/seqs/intron/align

mkdir -p $ALIGNS_PATH/filter/trimal
cd $ALIGNS_PATH/filter

## creating a batchfile to trim alignments
echo '#!/bin/bash' > trimal.sbatch
echo "#SBATCH --job-name=trimal
#SBATCH --output=trimal-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=0-3:00:00

ALIGNIN=\$(ls $ALIGNS_PATH/*.fasta | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENENAME=\$(basename \$ALIGNIN .fasta)

trimal \\
  -in \$ALIGNIN \\
  -out ./trimal/\$GENENAME.fasta \\
  -w $WINDOW_SIZE \\
  -gapthreshold $MIN_NONGAP_PERCENT \\
  -simthreshold $MIN_SIMILARITY \\
  -resoverlap $MIN_OVERLAP \\
  -seqoverlap $MIN_OVERLAP" >> trimal.sbatch

## run trimAl
module purge
module load StdEnv/2020
module load trimal/1.4
NFILES=$(ls -1 $ALIGNS_PATH/*.fasta | wc -l)
sbatch --mail-user=$EMAIL --array=1-$NFILES trimal.sbatch



###
## Filter supercontig alignments
###

ALIGNS_PATH=$WD/seqs/supercontig/align

mkdir -p $ALIGNS_PATH/filter/trimal
cd $ALIGNS_PATH/filter

## creating a batchfile to trim alignments
echo '#!/bin/bash' > trimal.sbatch
echo "#SBATCH --job-name=trimal
#SBATCH --output=trimal-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=0-3:00:00

ALIGNIN=\$(ls $ALIGNS_PATH/*.fasta | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENENAME=\$(basename \$ALIGNIN .fasta)

trimal \\
  -in \$ALIGNIN \\
  -out ./trimal/\$GENENAME.fasta \\
  -w $WINDOW_SIZE \\
  -gapthreshold $MIN_NONGAP_PERCENT \\
  -simthreshold $MIN_SIMILARITY \\
  -resoverlap $MIN_OVERLAP \\
  -seqoverlap $MIN_OVERLAP" >> trimal.sbatch

## run trimAl
module purge
module load StdEnv/2020
module load trimal/1.4
NFILES=$(ls -1 $ALIGNS_PATH/*.fasta | wc -l)
sbatch --mail-user=$EMAIL --array=1-$NFILES trimal.sbatch


```


When this is done, the result is smaller alignments that may contain fewer sequences, because some 
samples will have all their alignment positions filtered out by trimal for some loci. Thus, we again 
need to remove the loci that contain <4 samples (after trimal filtering).  
```bash
WD=/home/bourret/scratch/Villaverde_reanalysis

## find loci with data for <4 samples, and remove them (they cannot be used to estimate a tree)
cd $WD/seqs
for i in $(ls ./*/align/filter/trimal/*.fasta)
  do
    NSAMPLES=$(grep '>' $i | wc -l)
    if [ $NSAMPLES -lt 4 ]
      then
        echo "$i contains $NSAMPLES samples, which is <4, so it was removed!"
        rm $i
    else
	  echo "$i contains $NSAMPLES."
    fi
  done >> filter_min4_after_trimal.log

```


Finally, use TAPER to mask zones in one sample that are abnormally divergent from other samples.  
```bash
SRC=/project/def-bourret/shared/progs
WD=/home/bourret/scratch/Villaverde_reanalysis
EMAIL=votre.courriel@umontreal.ca
FILTERING_CUTOFF=1
MASKING_CHAR=N

###
## Filter exon alignments
###

ALIGNS_PATH=$WD/seqs/exon/align/filter

mkdir -p $ALIGNS_PATH/taper
cd $ALIGNS_PATH

## creating a batchfile to trim alignments
echo '#!/bin/bash' > TAPER.sbatch
echo "#SBATCH --job-name=TAPER
#SBATCH --output=TAPER-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=1G
#SBATCH --time=0-3:00:00

ALIGNIN=\$(ls $ALIGNS_PATH/trimal/*.fasta | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENENAME=\$(basename \$ALIGNIN .fasta)

julia $SRC/TAPER-1.0.0/correction_multi.jl \\
  --mask $MASKING_CHAR \\
  --cutoff $FILTERING_CUTOFF \\
  \$ALIGNIN > ./taper/\$GENENAME.fasta" >> TAPER.sbatch


## run TAPER
module purge
module load StdEnv/2023
module load julia/1.10.0
NFILES=$(ls -1 $ALIGNS_PATH/trimal/*.fasta | wc -l)
sbatch --mail-user=$EMAIL --array=1-$NFILES TAPER.sbatch

```

!!! Finish writing the TAPER script for introns and supercontigs in the above !!!

