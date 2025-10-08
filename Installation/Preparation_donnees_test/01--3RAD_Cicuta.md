# Préparations des données test 3RAD de *Cicuta*

Je vais utiliser des données de séquençages 3RAD issues du projet MSc de 
Thomas Villeneuve sur plusieurs taxons de *Cicuta*. 

Le but est de préparer un jeu de données qui représente bien les données 
brutes de séquençage Illumina d'une bibliothèque 3RAD. Il faut donc que ce ne 
soit pas démultiplexé, et conserver les index et les sites de restriction. 
Toutefois, il faut que ce soit plus petit et que ça contienne moins de loci, 
pour que l'analyse soit rapide.

Voici les étapes à faire:
  - Aligner toutes les lectures à un génome de référence (*Oenanthe*);
	- Sélectionner seulement les lectures qui sont alignées à 1 seul chromosome.

Le résultat conservera les lectures dans leur état brut, mais il y aura moins 
de données à gérer et aussi beaucoup moins de loci. En alignant à un génome 
de référence, on élimine aussi les lectures provenant des échantillons 
d'autres taxons qui ont été combiné avec les échantillons de *Cicuta*.

Voici le code à lancer:  
```bash
WD=/scratch/$USER/create_3rad_test_cicuta
ORIGINAL_READS=/data/sequenceData/3rad/3rad_20250310_plate1-moreSeq/*S1_L002*
BARCODE_FILE=/data/sequenceData/3rad/3rad_20250310_plate1/ipyrad_barcodes.txt
SAMPLE_FILTER=Cicuta
REF_GENOME=/data/genomes/Oenanthe_javanica_cv_FuqinNo1/Ojavanica_cv_FuqinNo1_v1.0.fna 
TARGET_REGION=CM109969.1
CPUS=4
MEM_PER_CPU=4G
TIME=14-00:00:00

## Créer et naviguer vers le dossier de travail
mkdir -p $WD
cd $WD

## Créer un lien symbolique vers le génome de référence
ln -s $REF_GENOME ref.fa

## Lancer la job d'alignement sur SLURM
## effectuer l'analyse
sbatch \
  --job-name=bwa_align \
	--output=bwa_align.out \
	--time=$TIME \
	--cpus-per-task=$CPUS \
	--mem-per-cpu=$MEM_PER_CPU \
	--wrap="bwa index ref.fa;	\
	  wait; \
    bwa mem -t $CPUS ref.fa $ORIGINAL_READS \
		  | samtools view -bS - > aligned.bam; \
		samtools sort aligned.bam -o aligned.sorted.bam; \
		samtools index aligned.sorted.bam; \
		samtools view -b aligned.sorted.bam \"$TARGET_REGION\" > aligned.subset.bam; \
		samtools fastq \
		  -1 Cicuta_test_3rad_R1.fastq.gz \
		  -2 Cicuta_test_3rad_R2.fastq.gz \
			-0 /dev/null -s /dev/null \
			-n aligned.subset.bam"

```
