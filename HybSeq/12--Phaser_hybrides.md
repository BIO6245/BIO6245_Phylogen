# Phaser les allèles ou sous-génomes des hybrides ou allopolyploïdes


## Approche du phasage par polarisation

Cette approche inventée initialement par 
[Leal et al. 2023](https://doi.org/10.1093/sysbio/syad009) est rapide et 
efficace pour phaser les allèles des hybrides et allopolyploïdes d'origine 
récente, lorsqu'on a les séquences d'au moins une des lignées parentales. 
Toutefois, elle est sensible aux données manquantes et aux erreurs 
d'alignement. Il est donc préférable d'avoir des données de grande qualité, 
quitte à en éliminer une certaine quantité (avec du filtrage plus sévère).

Pour cette approche, il faut préparer un alignement où les allèles 
alternatives sont codées comme des ambiguïtés IUPAC. Voici les étapes pour 
faire cela:  

1. Assembler les séquences d'une allèle représentative de chaque locus 
(p.ex. avec HybPiper);  

2. Optionnellement: filtrer les données pour s'assurer de la qualité (p.ex. 
avec Paragone et trimAl);  
	
3. Utiliser les allèles ainsi assemblées (et filtrées) comme séquences de 
référence pour le re-mappage des données brutes (p.ex., avec bwa-mem);  
	
4. Inférer les allèles alternatives à l'aide des données ainsi mappées 
(p.ex. avec GATK);  
	
5. Générer un alignement où toutes les allèles sont représentées à l'aide de 
codes d'ambiguïté IUPAC.  
	
Une fois qu'on a cet alignement avec allèles codées IUPAC, on peut le 
soumettre au processus de polarisation itérative, qui fonctionne ainsi:  

1. Générer un arbre aléatoire;  
	
2. Pour chaque échantillon ***i***, déterminer l'échantillon ***j*** le plus 
proche dans l'arbre;  
	
3. Polariser la séquence de ***i*** en fonction de ***j***, c'est à dire que 
si on a un S à la position 1 de ***i***, qui signifie les allèles C/G, 
et qu'on a C à la même position chez ***j***, on va générer la séquence G 
(le complément de C dans l'ensemble C/G);  
	
4. La séquence ainsi polarisée est **le plus différent possible** de celle 
de ***j***, ainsi, si ***j*** est un des parents de ***i***, la séquence 
polarisée devrait normalement ressembler à celle de l'autre parent;  
	
5. On va polariser les allèles de ***i*** qui n'ont pas pu être polarisés 
à l'étape précédente (car ils étaient absents ou ambigus chez ***j***) en 
fonction de la séquence du prochain échantillon le plus proche dans l'arbre, 
et ainsi de suite, de façon à ce qu'à la fin on ait iterativement polarisé 
***i*** en fonction de tous les échantillons de l'arbre, par ordre de 
proximité phylogénétique;  
	
6. On fait cette polarisation itérative pour tous les échantillons;  
	
7. L'alignement ainsi polarisé est soumis à l'analyse phylogénétique;  
	
8. Le nouvel arbre produit sert de base pour répéter la polarisation 
itérative (étapes 2 à 7);  
	
9. On répète les étapes précédentes (2 à 8) jusqu'à stabilisation (état 
où chaque échantillon hybride ou allopolyploïde alternera entre la séquence 
d'un parent et de l'autre parent);  
	
10. Lorsqu'on atteint la stabilisation, on détermine les séquences phasées à 
l'aide des arbres n et n-1, où n est le numéro du dernier arbre produit.  
	
Les sections suivantes indiquent comment effectuer tout cela.


## Nettoyer les séquences de référence avant le mappage

```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
INPUT_ALIGNMENTS=$WD/paragone/26_MO_final_alignments_trimmed
REF_FILE_ENDING=.1to1ortho.selected_stripped.aln.trimmed.fasta
TIME="0-12:00:00"
CPUS=1
MEM_PER_CPU=8G

# dossier pour remappage données Illumina sur séquences de ParaGone
mkdir -p $WD/remap
cd $WD/remap

## dossier pour séquences de référence pour le remappage
mkdir -p $WD/remap/ref_seqs/raw
mkdir -p $WD/remap/ref_seqs/clean
cd $WD/remap/ref_seqs/clean
ln -s $INPUT_ALIGNMENTS/*$REF_FILE_ENDING ../raw/

## nettoyage des séquences de référence avec SimpleAlignmentCleaner
NFILES=$(ls -1 ../raw/*$REF_FILE_ENDING | wc -l)
conda activate bio
sbatch \
  --job-name=SimpleAliClean \
  --output=SimpleAliClean-%a.log \
	--array=1-$NFILES \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=$CPUS \
  --mem-per-cpu=$MEM_PER_CPU \
  --wrap="
		INFILE=\$(ls ../raw/*$REF_FILE_ENDING | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
		NAME=\$(basename \$INFILE $REF_FILE_ENDING)
		python3 $SRC/SimpleAlignmentCleaner*.py \
			-i \$INFILE \
			-o \$NAME.clean \
			--outformat fasta \
			--quantile_threshold 0.9"

```

### Mapper, identifier les variants et les filtrer dans chaque échantillon

Mappage des lectures Illumina sur les exons assemblés par HybPiper avec bwa, 
suivi de l'identification des variants alléliques et filtration de ces 
variants en utilisant GATK:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
READS_PATH=$WD/reads/trim
SAMPLE_LIST_PATH=$WD/samplelist.txt
REF_FOLDER=$WD/remap/ref_seqs/clean
REF_FILE_ENDING=.final.fasta
EMAIL=votre.courriel@umontreal.ca
TIME="7-00:00:00"
CPUS=1
MEM_PER_CPU=4G

# dossier pour remappage données Illumina sur séquences de ParaGone
mkdir -p $WD/remap
cd $WD/remap

## dossier pour séquences de référence pour le remappage
mkdir -p $WD/remap/ref_seqs

## faire une copie des séquences de référence en éliminant les indels (-)
for i in $REF_FOLDER/*$REF_FILE_ENDING
	do
		NAME=$(basename $i $REF_FILE_ENDING)
		awk '/^>/ {if (seq) print seq; print; seq=""} /^[^>]/ {gsub(/-/, ""); seq=seq$0} END {if (seq) print seq}' $i > $WD/remap/ref_seqs/$NAME.fasta
	done

## Créer un lien vers la liste des échantillons à analyser
ln -s $WD/samplelist.txt .

## Créer le batch file pour SLURM
echo '#!/bin/bash' > bwa-gatk.sbatch
echo "#SBATCH --job-name=bwa-gatk
#SBATCH --output=bwa-gatk-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CPUS
#SBATCH --mem-per-cpu=$MEM_PER_CPU
#SBATCH --time=$TIME

BASE_DIR=\$(pwd)

## Déterminer quel échantillon analyser dans cette tâche

SAMPLE=\$(cat \$BASE_DIR/samplelist.txt | head -n \$SLURM_ARRAY_TASK_ID | tail -1)

## Créer un dossier pour cet échantillon et y naviguer
mkdir -p \$BASE_DIR/\$SAMPLE
cd \$BASE_DIR/\$SAMPLE

## Créer une liste avec le nom d'échantillon (nécessaire pour seqtk)
echo \"\$SAMPLE\" > tmp.list

## Boucle qui va chercher séquence d'échantillon de chaque locus 
## et le mets dans un seul fichier .fasta, avec le nom de séquence 
## qui inclut le nom du gène
for ALIGNMENT in $WD/remap/ref_seqs/*.fasta
  do
    LOCUS=\$(basename \$ALIGNMENT .fasta)
		seqtk subseq \$ALIGNMENT tmp.list >> \$SAMPLE.fasta
		sed -i \"s/\$SAMPLE.*/\$LOCUS/g\" \$SAMPLE.fasta
		sed -i \"s/\$SAMPLE multi_.*/\$LOCUS/g\" \$SAMPLE.fasta
    sed -i \"s/\$SAMPLE single_.*/\$LOCUS/g\" \$SAMPLE.fasta
  done

## Supprimer la liste avec le nom d'échantillon
rm tmp.list

## Mapper les données Illumina de l'échantillon sur les séquences références de cet échantillon
bwa index \$SAMPLE.fasta
bwa mem -t $CPUS \$SAMPLE.fasta \\
  $READS_PATH/\${SAMPLE}_trim_R1.fastq.gz \\
  $READS_PATH/\${SAMPLE}_trim_R2.fastq.gz | 
  samtools view -bS - | 
  samtools sort -o \$SAMPLE.bam -
  
## Créer les index et références nécessaires pour gatk
java -jar $SRC/picard.jar AddOrReplaceReadGroups \\
    I=\$SAMPLE.bam \\
    O=\$SAMPLE.rg.bam \\
    RGID=group1 \\
    RGLB=lib1 \\
    RGPL=illumina \\
    RGPU=unit1 \\
    RGSM=\$SAMPLE
		
samtools index \$SAMPLE.rg.bam
samtools faidx \$SAMPLE.fasta
java -jar $SRC/picard.jar CreateSequenceDictionary \\
  -R \$SAMPLE.fasta \\
  -O \$SAMPLE.dict

## Identifier les SNPs avec gatk
$SRC/gatk-4.6.0.0/gatk HaplotypeCaller \\
  -R \$SAMPLE.fasta \\
  -I \$SAMPLE.rg.bam \\
  -O \$SAMPLE.vcf

## Filtrer les variants de mauvaise qualité
GATK_FILTER=\"DP < 3 || QD < 2.0 || FS > 60.0 || MQ < 40.0 || SOR > 3.0 || MQRankSum < -12.5 || ReadPosRankSum < -8.0\"
$SRC/gatk-4.6.0.0/gatk VariantFiltration \\
    -R \$SAMPLE.fasta \\
    -V \$SAMPLE.vcf \\
    -O \$SAMPLE.filt.vcf \\
    --filter-expression \"\$GATK_FILTER\" \\
    --filter-name \"StandardFilters\"

## Créer une séquence consensus avec des codes IUPAC pour les positions hétérozygotes
bgzip -c \$SAMPLE.filt.vcf > \$SAMPLE.filt.vcf.gz
tabix -p vcf \$SAMPLE.filt.vcf.gz
bcftools consensus \\
  --iupac-codes \\
  --samples \$SAMPLE \\
  --fasta-ref \$SAMPLE.fasta \\
  \$SAMPLE.filt.vcf.gz > \$SAMPLE.hetero.fasta" >> bwa-gatk.sbatch


## Déterminer combien d'échantillons à analyser
NFILES=$(wc -l < samplelist.txt)

## Soumettre ces les tâches de mappage et identification des variants
sbatch --mail-user=$EMAIL --array=1-$NFILES bwa-gatk.sbatch

```

Le résultat est un dossier dans lequel chaque échantillon contient un fichier 
nommé `nom.hetero.fasta` qui contient la séquence de tous les gènes assemblés 
par HybPiper, mais avec les positions hétérozygotes codées avec les codes 
d'ambiguïté IUPAC.


### Créer des listes de séquences avec hétérozygotes par gène

Code pour concaténer toutes les séquences avec hétérozygotes IUPAC générés 
dans la section précédente dans un seul fichier `.fasta` par locus:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
TARGETS=/data/hybseqRefs/combined_Mega353_Carex554.fa

## Créer dossier pour les listes de séquences par gène
mkdir -p $WD/remap/seqs_iupac
cd $WD/remap/seqs_iupac

## Créer une liste des gènes
ls -1 $WD/remap/ref_seqs/*.fasta | xargs -n1 basename -s .fasta > genelist.txt

## Pour chaque gène dans la liste de gène, aller chercher les séquences de 
## chaque espèce et les combiner dans un seul fichier fasta
echo "while IGS= read -r GENE
  do
    echo \"Fetching sequences for \$GENE.\"
	while IGS= read -r SAMPLE
	  do
	    echo \"\$GENE\" > tmp.list
	    seqtk subseq ../\$SAMPLE/\$SAMPLE.hetero.fasta tmp.list >> \$GENE.fasta
	    sed -i \"s/\$GENE/\$SAMPLE/g\" \$GENE.fasta
	  done < ../samplelist.txt
  done < genelist.txt" > fetch_seqs.sh

## run the loop in the background
chmod +x fetch_seqs.sh
nohup ./fetch_seqs.sh > fetch_seqs.log &

```

Une fois que les séquences ont été combinées par gène, retirer les loci 
représentés par <4 échantillons puisque ces loci ne contiennent pas 
d'information phylogénétique importante:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest

## Trouver les loci avec <4 échantillons et les supprimer
cd $WD/remap/seqs_iupac
for i in $(ls ./*.fasta)
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

### Alignement des loci avec hétérozygotes IUPAC

Aligner tous les loci restants après le filtrage précédent avec MAFFT, 
nettoyer avec SimpleAlignmentCleaner et estimer un arbre phylogénétique avec 
FastTree:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

SEQS_PATH=$WD/remap/seqs_iupac
TREES_PATH=$WD/remap/seqs_iupac/trees
mkdir -p $SEQS_PATH/align/raw
mkdir -p $SEQS_PATH/align/clean
mkdir -p $TREES_PATH/fastTree
cd $SEQS_PATH/align

## Créer un fichier SLURM batch pour l'analyse mafft + FastTree
echo '#!/bin/bash' > mafft-fasttree.sbatch
echo "#SBATCH --job-name=mafft-fasttree
#SBATCH --output=mafft-fasttree-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=4G
#SBATCH --time=$TIME

SEQIN=\$(ls $SEQS_PATH/*.fasta | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENENAME=\$(basename \$SEQIN .clean.fasta)

mafft --genafpair --maxiterate 1000 \$SEQIN \\
	> $SEQS_PATH/align/raw/\$GENENAME.fasta

python3 $SRC/SimpleAlignmentCleaner*.py \\
	-i $SEQS_PATH/align/raw/\$GENENAME.fasta \\
	-o $SEQS_PATH/align/clean/\$GENENAME.fasta \\
	--outformat fasta \\
	--quantile_threshold 0.9

FastTree \\
	-nt \\
	-gtr $SEQS_PATH/align/clean/\$GENENAME.fasta \\
	> $TREES_PATH/fastTree/\$GENENAME.tre
" >> mafft-fasttree.sbatch

## Déterminer le nombre de loci à analyser
NFILES=$(ls -1 $SEQS_PATH/*.fasta | wc -l)

## Envoyer les tâches d'alignement à SLURM
conda activate bio
sbatch --mail-user=$EMAIL --array=1-$NFILES mafft-fasttree.sbatch

```

### Concaténer les séquences alignées

Concaténer les séquences alignées dans la section précédente en un seul grand alignement:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
ALIGN_PATH=$WD/remap/seqs_iupac/align/clean
ALIGN_FILE_NAMES=*.final.fasta
EMAIL=votre.courriel@umontreal.ca
TIME="0-3:00:00"

## Créer dossier pour séquences concaténées
mkdir -p $ALIGN_PATH/concat
cd $ALIGN_PATH/concat

## Créer un alignement concaténé format phyml avec catfasta2phyml
sbatch \
  --job-name=catfasta2phyml \
  --output=catfasta2phyml.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=1 \
  --mem-per-cpu=8G \
  --wrap="$SRC/catfasta2phyml/catfasta2phyml.pl \
  --concatenate $ALIGN_PATH/$ALIGN_FILE_NAMES \
  --sequential \
  1> concat.phy \
  2> concat.partitions"

## Créer un alignement concaténé format fasta avec catfasta2phyml
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
  --concatenate $ALIGN_PATH/$ALIGN_FILE_NAMES \
  1> concat.fasta \
  2> /dev/null"

sed -i 's!.*/!!' concat.partitions
sed -i 's!\.final.fasta!!' concat.partitions
sed -i 's!\.fasta!!' concat.partitions

```

### Phaser les séquences par polarisation iterative (IterPol)

IterPol est un programme Python basé sur le phasage par polarisation de 
[Leal et al. 2023](https://doi.org/10.1093/sysbio/syad009), mais avec des 
améliorations inédites d'Étienne Léveillé-Bourret. Le code n'est pas encore 
stable et n'est pas publié, mais il est déjà efficace pour phaser les allèles 
d'hybrides et d'allopolyploïdes récents.

Voici du code pour lancer la version la plus récente d'IterPol:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC=/opt
WD=/scratch/$USER/HybSeqTest
ALIGNMENT=$WD/remap/seqs_iupac/align/clean/concat/concat.phy
PREFIX=Carex
TIME=2-00:00:00
CPUS=8
MEM_PER_CPU=4G

mkdir -p $WD/iterpol
cd $WD/iterpol

conda activate dendropy
sbatch --job-name=IterPol \
  --output=IterPol.out \
  --time=2:00:00 \
	--cpus-per-task=$CPUS \
	--mem-per-cpu=$MEM_PER_CPU \
  --wrap "
		python3 $SRC/IterPol/IterPol_v*.py \
			--infile $ALIGNMENT \
			--out_prefix $PREFIX \
			--method paup \
			--threads $CPUS \
			--paup_path $SRC/paup4a168_centos64
			"

```
