# Analyse de parcimonie sur les données concaténées

Les données HybSeq utilisées ici correspondent à quelques centaines de gènes nucléaires récupérés dans 
sept espèces différentes de fougères du genre *Dryopteris* et d'un genre apparenté, *Polystichum*. Tous 
ces gènes ont une histoire évolutive différente. Toutefois, si l'ensemble de ces gènes est analysé en 
bloc et qu'on estime une phylogénie globale pour ce bloc, cette phylogénie devrait être une bonne 
approximation de l'arbre d'espèces. 

Cette approche est nommée "analyse par concaténation", car elle procède tout d'abord par la 
combinaison (=concaténation) des alignements des différents gènes en une seule matrice, suivi de 
l'analyse de cette matrice "concaténée" comme si c'était l'alignement d'un seul gène.

Ici, nous allons effectuer une analyse de parcimonie sur la matrice concaténée.


## Concaténer les alignements des différents gènes

Tout d'abord, il est nécessaire de concaténer les alignements des différents gènes qui sont enregistrés 
dans `/scratch/$USER/HybSeqTest/seqs/exon/align`. Pour ce faire, nous allons utiliser le code ci-dessous 
à exécuter sur le serveur:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca


###
## Script principal
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
  1> concat.phy \
  2> concat.partitions"

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
  1> concat.fasta \
  2> /dev/null"

```

After the previous jobs finish, fix the partition file to make it readable by IQ-Tree and RAxML.  
```bash
WD=/home/bourret/scratch/Villaverde_reanalysis

###
## Fix partition file for exons
###

ALIGN_PATH=$WD/seqs/exon/align/concat

cd $ALIGN_PATH

## add the "DNA, " required at start of each line of partition file for RAxML
sed -i 's/^/DNA, /g' concat.partitions

## remove file path and ending from partition names
sed -i 's/DNA, .*\//DNA, /g' concat.partitions
sed -i 's/.fasta//g' concat.partitions

```

## Run a quick-and-dirty FastTree analysis on concatenated dataset

Run FastTree on the concatenated alignment to have a quick look at the expected results we can get from 
this matrix:  
```bash
WD=/home/bourret/scratch/Villaverde_reanalysis
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-3:00:00"
THREADS=4

###
## run FastTree on concatenated exon dataset
###

ALIGN_PATH=$WD/seqs/exon/align/concat/concat.fasta
TREES_PATH=$WD/trees/exon/concat/fasttree
mkdir -p $TREES_PATH
cd $TREES_PATH

## create a SLURM batchfile to align with MAFFT and run FastTree on each locus in an array
echo '#!/bin/bash' > fasttree.sbatch
echo "#SBATCH --job-name=fasttree
#SBATCH --output=fasttree.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$THREADS
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

DATASET_NAME=\$(basename $ALIGN_PATH .fasta)

FastTree -nt -gtr $ALIGN_PATH > \${DATASET_NAME}_fast.tre" >> fasttree.sbatch

## send the FastTree task to SLURM
conda deactivate
module purge
module load StdEnv/2020
module load fasttree/2.1.11
sbatch --mail-user=$EMAIL fasttree.sbatch

```


### Model and partition selection

We will use IQ-tree to select the best model and partition scheme for the concatenate dataset.  
```bash
SRC=/project/def-bourret/shared/progs
WD=/home/bourret/scratch/Villaverde_reanalysis/trees/exon
EMAIL=etienne.leveille-bourret@umontreal.ca
TIME="0-12:00:00"
THREADS=16
ALIGNMENT=/home/bourret/scratch/Villaverde_reanalysis/seqs/exon/align/concat/concat.phy
PARTITIONS=/home/bourret/scratch/Villaverde_reanalysis/seqs/exon/align/concat/concat.partitions

mkdir -p $WD/concat
cd $WD/concat

## create a SLURM batchfile to perform model selection with IQ-tree
echo '#!/bin/bash' > iqtree-modselect.sbatch
echo "#SBATCH --job-name=iqtree-modselect
#SBATCH --output=iqtree-modselect.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$THREADS
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

mkdir -p ./iqtree_modselect
cd ./iqtree_modselect
ln -s $ALIGNMENT .
ln -s $PARTITIONS .
iqtree \\
  -nt $THREADS \\
  -s *.phy \\
  -spp *.partitions \\
  -m TESTMERGEONLY \\
  -mset raxml \\
  -rcluster 10

" >> iqtree-modselect.sbatch

## submit the iq-tree model selection job
module purge
module load StdEnv/2020 intel/2020.1.217
module load iq-tree/2.0.7
sbatch --mail-user=$EMAIL iqtree-modselect.sbatch

```


!!! code below not run and finished yet !!!


Because we are only working on a "test" dataset, we can proceed with phylogenetic analysis of the whole 
concatenated alignment, without having to worry too much about alignment and assembly errors. Here is a 
script to run a [RAxML](https://cme.h-its.org/exelixis/web/software/raxml/) analysis, partitioned by gene.
 **Please change the email address**. To know what all of these commands are doing in RAxML, you will need 
 to [read the manual](https://cme.h-its.org/exelixis/php/countManualNew.php):
```bash
WD=/home/$USER/scratch/HybSeqTest/align/concat
EMAIL=etienne.leveille-bourret@umontreal.ca
ALIGNMENT=all_supercontig.phy
PARTITIONS=all_supercontig.partitions
THREADS=16
ARGUMENTS_FOR_RAXML="-f a -p 1234 -x 1234 -# 100 -m GTRCAT -n supercontig"

cd $WD

## load required modules
conda deactivate
module purge
module load StdEnv/2020
module load nixpkgs/16.09 intel/2018.3 openmpi/3.1.4
module load raxml/8.2.11

## send a raxml job to the cluster
sbatch --job-name=raxml_test \
  --output=raxml_test.out \
  --mail-user=$EMAIL \
  --mail-type=end \
  --time=03:00:00 \
  --mem-per-cpu=1G \
  --cpus-per-task=$THREADS \
  --wrap  "raxmlHPC \
    $ARGUMENTS_FOR_RAXML \
    -T $THREADS \
    -s $ALIGNMENT \
    -q $PARTITIONS"
```
