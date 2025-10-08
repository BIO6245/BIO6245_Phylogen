# Gérer la paralogie dans les données HybSeq

## Déterminer le nombre de potentiels paralogues

Parfois, certains gènes visés dans une étude HybSeq sont dupliqués dans un ou 
plusieurs échantillons. Si les gènes ont été dupliqués il y a peu de temps, 
c'est possible que les deux paralogues issus de cet événement de duplication 
n'aient pas divergés suffisamment pour être séparés lors de l'analyse. Les 
lectures Illumina issues de ces deux paralogues seront alors alignés sur la 
même séquence de référence, et ils seront assemblés comme des contigs 
différents par HybPiper (car ils ne sont pas exactement identiques).

Par défaut, si HybPiper a assemblé plus d'un contig pour un gène, alors il 
sélectionnera le "meilleur" contig comme étant celui qui a le plus de 
couverture de séquençage (10X plus de couverture que tous les autres), ou qui 
est le plus long (>=75% de la séquence ciblée présente). Si plusieurs contigs 
ont une haute couverture et longueur, alors il est fort probable que ces 
contigs correspondent à différents haplotypes ou paralogues d'un gène 
dupliqué.

Pour vérifier combien de gènes possèdent des paralogues ou haplotypes dans 
chaque échantillon, exécuter ces commandes:   
```bash
WD=/scratch/$USER/HybSeqTest
TARGETS=/data/hybseqRefs/combined_Mega353_Carex554.fa

cd $WD

## Créer un rapport indiquant le nb potentiel de paralogues / échantillon
conda activate hybpiper
sbatch --job-name=paralog_retriever \
  --output=paralog_retriever.out \
  --time=2:00:00 \
  --wrap "
		hybpiper paralog_retriever samplelist.txt -t_dna $TARGETS
		"

```

Une fois cette étape terminée, téléchargez le fichier `paralog_report.tsv` sur 
votre ordinateur local et examinez-le dans Excel.  
  - Combien de loci sont présents en une seule copie dans tous les échantillons?  
	- Combien de loci contiennent possiblement des paralogues?  
  - Existe-t-il des loci présentant des paralogues dans chaque échantillon, ce 
	qui suggérerait une duplication génique ancienne partagée par toutes les 
	espèces ?  

---

## Visualiser les arbres de gènes incluant les potentiels paralogues

HybPiper sélectionne pour chaque locus un contig qu’il considère comme le 
"meilleur représentant", en se basant sur la couverture, la longueur et la 
similarité avec la séquence de sonde. Cependant, il est préférable de vérifier 
manuellement tous les arbres géniques afin de s’assurer qu’aucune erreur n’a été commise par HybPiper.  

Pour chaque locus, alignez tous les contigs récupérés par *paralog retriever* 
à l’aide de [MAFFT](https://mafft.cbrc.jp/alignment/software/about.html), puis 
générez rapidement une phylogénie génique à l’aide de 
[FastTree](http://www.microbesonline.org/fasttree/):    
```bash
WD=/scratch/$USER/HybSeqTest
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"
CPUS=1
MEM_PER_CPU=2G

mkdir -p $WD/trees/paralogs
cd $WD/trees/paralogs

## pour chaque locus, aligner avec mafft et estimer phylo avec FastTree
echo '#!/bin/bash' > quickParaTree.sbatch
echo "#SBATCH --job-name=quickParaTree
#SBATCH --output=quickParaTree-%a.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CPUS
#SBATCH --mem-per-cpu=$MEM_PER_CPU
#SBATCH --time=$TIME

GENE_FASTA=\$(ls $WD/paralogs_all/*.fasta | head -n \$SLURM_ARRAY_TASK_ID | tail -1)
GENE_NAME=\$(basename \$GENE_FASTA .fasta)

## aligner avec mafft => estimer arbre avec FastTree
mafft --localpair \$GENE_FASTA | fasttree -nt -gtr > \$GENE_NAME.tre
" >> quickParaTree.sbatch

## déterminer combien de gènes à analyser
NFILES=$(ls -1 $WD/paralogs_all/*.fasta | wc -l)

## run the loop in the background, save screen output to log file
sbatch --mail-user=$EMAIL --array=1-$NFILES quickParaTree.sbatch

```

Attendre que cette analyse termine. Une fois terminée, combiner tous ces 
arbres de gènes en un seul fichier pour faciliter la visualisation:  
```bash
WD=/scratch/$USER/HybSeqTest

cd $WD/trees/paralogs

## créer un fichier pour contenir les arbres de tous les loci
echo "#NEXUS
begin trees;" > allTrees.nex

## boucle pour ajouter chaque arbre de gène dans ce fichier .nex
for TREE in $(ls *.tre)
	do
		GENE_NAME=$(basename $TREE .tre)
		
		## si arbre existe dans le fichier $TREE, combiner avec les autres
		if [ -s $TREE ]; then
			echo "tree $GENE_NAME = $(<$TREE)" >> allTrees.nex

		## autrement, il n'y a rien dans ce fichier, supprimer
		else
			rm $TREE
			continue
		fi
		
	done
	
## terminer le fichier par "end;", nécessaire pour format .nex
echo "end;" >> allTrees.nex

```

Une fois cela fait, télécharger `allTrees.nex` sur votre machine locale et 
passer visuellement à travers tous les arbres en utilisant 
using [FigTree](http://tree.bio.ed.ac.uk/software/figtree/) ou un autre 
programme de visualisation. Porter particulièrement attention aux loci qui 
ont été identifiés comme ayant de nombreux potentiels paralogues pour de 
nombreux échantillons.

  - **Question:** Y a-t-il de l'évidence que des contigs alléliques ont été 
	assemblés?

  - **Question:** Y a-t-il de l'évidence que des paralogues ont été assemblés?

  - **Question:** Comment distinguerait-on des allèles de paralogues dans les 
	arbres de gènes que vous visualisez?

---

## Éliminer les paralogues avec ParaGone

[ParaGone](https://github.com/chrisjackson-pellicle/ParaGone) est un 
pipeline populaire pour éliminer les paralogues dans des séquences assemblées 
avec HybPiper.

Exécuter les commandes ci-dessous pour éliminer les paralogues dans vos 
séquences en utilisant ParaGone:  
```bash
WD=/scratch/$USER/HybSeqTest
PARALOGS_FILES=$WD/paralogs_all/*.fasta
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"
CPUS=8
MEM_PER_CPU=4G
OUTGROUPS="--internal_outgroup Carex_baileyi_ELB615_S66_L003 \
--internal_outgroup Carex_intumescens_ELB196 \
--internal_outgroup Carex_vesicaria_ELB251_S50_L003"

mkdir -p $WD/paragone
cd $WD/paragone

## créer un dossier avec un lien vers les séquences de gènes/paralogues
mkdir -p $WD/paragone/paralogs_seqs
ln -s $PARALOGS_FILES $WD/paragone/paralogs_seqs/

## supprimer les fichiers si ils sont vides ou contiennent <4 séquences
for i in $(ls $WD/paragone/paralogs_seqs/*.fasta $WD/paragone/paralogs_seqs/*.FNA)
  do
    NSAMPLES=$(grep '>' $i | wc -l)
    if [ $NSAMPLES -lt 4 ]; then
      echo "$i contains $NSAMPLES samples, which is <4, so it was removed!"
      rm $i
    else
			echo "$i contains $NSAMPLES samples."
    fi
  done >> filter_min4.log

## créer un batchfile pour l'analyse ParaGone
echo '#!/bin/bash' > paragone.sbatch
echo "#SBATCH --job-name=paragone
#SBATCH --output=paragone.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$CPUS
#SBATCH --mem-per-cpu=$MEM_PER_CPU
#SBATCH --time=$TIME

paragone check_and_align \\
	$WD/paragone/paralogs_seqs \\
	$OUTGROUPS \\
	--pool $CPUS \\
	--threads 1
	
paragone alignment_to_tree \\
	04_alignments_trimmed_cleaned \\
	--pool $CPUS \\
	--threads 1 \\
	--use_fasttree

paragone qc_trees_and_extract_fasta 04_alignments_trimmed_cleaned \\
	--cut_deep_paralogs_internal_branch_length_cutoff 0.04

paragone align_selected_and_tree \\
	04_alignments_trimmed_cleaned \\
	--use_fasttree \\
	--pool $CPUS \\
	--threads 1

paragone prune_paralogs \\
	--mo \\
	--rt \\
	--mi

paragone final_alignments \\
	--mo \\
	--rt \\
	--mi \\
	--pool $CPUS \\
	--threads 1 \\
	--keep_intermediate_files" >> paragone.sbatch

## exécuter paragone
conda activate paragone
sbatch --mail-user=$EMAIL paragone.sbatch

```

Pendant que l'analyse roule, profitez-en pour lire le 
[wiki de ParaGone](https://github.com/chrisjackson-pellicle/ParaGone/wiki/Tutorial). 
Vous remarquerez que le pipeline inclue des étapes d'alignement des 
séquences, de nettoyage de ces séquences, de génération d'arbres de gènes, 
ainsi que d'extraction des clades de ces arbres correspondant à des paralogues 
différents. Il existe 4 méthodes différentes à l'étape d'extraction, chacune 
avec des avantages et inconvénients.

  - **Question**: Quelles sont les 4 méthodes d'extraction?
	
  - **Question**: Quels sont les avantages et inconvénients de ces 4 méthodes?
	
	- **Question**: Pourquoi la méthode *MO* est préférable dans la majorité des 
	cas lorsque des hors-groupes sont inclus dans les séquences à analyser?

