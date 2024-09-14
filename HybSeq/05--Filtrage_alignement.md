# Filtrage des alignements

Plusieurs sources d'erreur peuvent survenir lors du séquençage, de l'assemblage et de l'alignement 
qui peuvent résulter en des erreurs d'alignement, où les nucléotides d'une espèce ne sont pas du 
tout aligner avec les nucléotides homologues d'une autre espèce.

Quelques exemples d'erreur:  
- Si l'ADN d'une espèce est de plus faible qualité, il peut résulter une plus faible couverture de 
séquençage, menant à un plus grand nombre d'erreurs de séquençage. En conséquence, certaines 
"mutations" observées chez cette espèce peuvent être seulement dû à des erreurs de séquençage.  
- Dans une région particulièrement riche en mutations de type insertion et délétion, il peut être 
plus difficile pour les algorithmes d'alignement automatisés de créer un alignement fiable.  
- Les mutations de type inversion causent presque automatiquement des erreurs d'alignement, car la 
séquence ACTG inversée devient GTCA, interprétée comme 4 mutations différentes dans un alignement.
- Si des éléments transposables sont insérés au centre d'une séquence, ou si le locus séquencé a 
changé de position dans le génome, ça peut mener à des nucléotides présents dans la séquence d'une 
espèce qui n'ont pas d'équivalent dans la séquence des autres espèces.  
- etc.  

En conséquence, il est préférable d'examiner les alignements à l'oeil pour déceler les erreurs, 
leur fréquence et leur source probable.

Une fois la fréquence et la source des erreurs identifiée, on peut soit:  
1. laisser les erreurs s'ils sont peu fréquentes et ne risquent pas de confondre le résultat 
final;  
2. corriger à la main les erreurs (lorsque la quantité de données est assez faible);  
3. retirer les séquences problématiques ou les "masquer" en les transformant en "?", des "-" ou 
des "N";  
4. utiliser des programmes pour retirer et/ou masquer les zones de séquences problématiques.  

Nous allons ici tester quelques programmes qui filtrent les alignements en retirer les colonnes 
et les zones qui semblent être moins bien alignées.  


## Masquer les zones qui sont anormalement divergentes à l'aide de TAPER

[TAPER](https://github.com/chaoszhang/TAPER) est une approche qui identifie des zones de 
divergence anormalement élevée par rapport au reste de la séquence de la même espèce. Ces zones 
sont ensuite masquées en les transformant en "N" (ambiguïté).  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca
TIME="0-3:00:00"
FILTERING_CUTOFF=3
MASKING_CHAR=N


###
## Filter les alignements d'exons
###

ALIGNS_PATH=$WD/seqs/exon/align

mkdir -p $ALIGNS_PATH/taper
cd $ALIGNS_PATH

echo "for i in \$(ls $ALIGNS_PATH/*.fasta)
  do
    GENENAME=\$(basename \$i .fasta)
	echo \"Running TAPER on \$GENENAME\"
    julia -t 1 $SRC/TAPER-1.0.0/correction_multi.jl \\
	  --mask $MASKING_CHAR \\
	  --cutoff $FILTERING_CUTOFF \\
      \$i > ./taper/\$GENENAME.fasta
  done" > TAPER.sh

## rouler TAPER en background
nohup bash TAPER.sh > taper.log &

## suivre le progrès de la tâche
tail -f taper.log

```



!!! CONTINUER À PARTIR D'ICI !!!




## Filtrer les régions des alignements qui contiennent trop de caractères indéterminés

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



!!! Ignorer le texte ci-bas !!!

!!! Code qui ne fonctionne pas, à régler avec Nicolas !!!

TAPER ne roule pas bien sur SLURM. Le problème semble provenir de julia
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca
TIME="0-3:00:00"
THREADS=1
FILTERING_CUTOFF=3
MASKING_CHAR=N


###
## Filtrer les alignements d'exons
###

ALIGNS_PATH=$WD/seqs/exon/align

mkdir -p $ALIGNS_PATH/taper
cd $ALIGNS_PATH

## creating a batchfile to trim alignments
echo '#!/bin/bash' > TAPER.sbatch
echo "#SBATCH --job-name=TAPER
#SBATCH --output=TAPER-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$THREADS
#SBATCH --mem-per-cpu=1G
#SBATCH --time=$TIME

ALIGNIN=\$(ls $ALIGNS_PATH/*.fasta | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENENAME=\$(basename \$ALIGNIN .fasta)

julia -t $THREADS $SRC/TAPER-1.0.0/correction_multi.jl \\
  --mask $MASKING_CHAR \\
  --cutoff $FILTERING_CUTOFF \\
  \$ALIGNIN > ./taper/\$GENENAME.fasta" >> TAPER.sbatch


## run TAPER
NFILES=$(ls -1 $ALIGNS_PATH/*.fasta | wc -l)
sbatch --mail-user=$EMAIL --array=1-$NFILES TAPER.sbatch

```