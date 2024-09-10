# Filtrage des alignements

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

