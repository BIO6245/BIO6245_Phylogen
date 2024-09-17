# Alignement des séquences

## Loci nucléaires assemblés par HybPiper

### Récupérer les séquences assemblées par HybPiper

Récupérons les séquences d'exons avec HybPiper afin de pouvoir les 
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

```

---

### Éliminer les alignements contenant moins de 4 séquences

Une fois que les séquences ont été récupérées avec HybPiper, retirer les loci représentés 
par <4 échantillons puisque ces loci ne contiennent pas d'information phylogénétique importante:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest

## Trouver les loci avec <4 échantillons et les supprimer
cd $WD/seqs
for i in $(ls ./exon/*.FNA)
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

Vous pouvez aussi examiner spécifiquement les lignes où on a filtré des gènes en exécutant 
`grep 'removed' filter_min4.log`.

- **Question:** Pourquoi est-ce que ça prend au minimum 4 échantillons dans un alignement pour 
pouvoir estimer un arbre de relations phylogénétiques?

---

## Alignement des séquences

Aligner tous les loci restants après le filtrage précédent avec 
[MAFFT](https://mafft.cbrc.jp/alignment/software/algorithms/algorithms.html), un programme qui 
utilise des algorithmes d'alignement efficaces et assez précis. En même temps, il nous est possible 
d'estimer des arbres phylogénétiques rapidement avec 
[FastTree](http://www.microbesonline.org/fasttree/), tout en gardant en tête que ces arbres ne sont 
pas très précis.
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

---

### Corriger les noms des échantillons dans les alignements

Les noms de séquence dans les alignements contiennent un commentaire lié au nombre de contigs qui 
ont été combinés pour créer la séquence finale. Ceux-ci doivent être supprimés, sinon le même 
échantillon porte un nom différent dans chaque locus, en fonction du nombre de contigs qui ont 
été assemblés pour créer la séquence finale de chaque locus.  
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

Une fois tout ça fait, télécharger quelques alignements (fichiers `.fasta`) et les examiner à 
l'oeil sur votre propre ordinateur avec MEGA. 

- **Question:** Est-ce qu'il semble y avoir des erreurs? 

- **Question:** Si oui, qu'est-ce qui pourrait en être la cause, et comment les corriger?

Ensuite, télécharger quelques arbres (fichiers `.tre`) et les examiner sur votre propre ordinateur 
avec le programme [FigTree](http://tree.bio.ed.ac.uk/software/figtree/).

- **Question:** Est-ce que tous les arbres de gènes ont la même topologie?

- **Question:** S'il y a des différences, qu'est-ce qui pourrait être en cause? Trouvez au moins 
un exemplede processus biologique et un exemple d'erreur analytique qui pourrait causer des 
différences de topologie.
