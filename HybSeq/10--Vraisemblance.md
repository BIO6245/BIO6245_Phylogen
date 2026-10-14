# Analyse de maximum de vraisemblance

Ce tutoriel vous guidera dans l’utilisation d'**IQ-Tree** et de **RAxML** pour
 l'analyse de maximum de vraisemblance (*maximum likelihood*) sur une matrice de
 données concaténées ou sur un seul gène. Il assume qu'on sait déjà quel est le
 meilleur modèle de substitution.

---

## Maximum de vraisemblance dans IQ-Tree

**IQ-Tree** offre une approche rapide et flexible pour la sélection de modèles
 et le partitionnement. Il permet de faire des analyses de maximum de
 vraisemblance sur à peu près n'importe quel modèle d'évolution et de
 partitionnement imaginable. Cette flexibilité vient avec un coût: il est plus
 lent que RAxML, particulièrement avec les jeux de données qui sont un peu plus
 grands.

Prérequis:
- Avoir un fichier en format phylip (.phy) contenant la concaténation de tous
 les loci;  
- Avoir un fichier associé qui donne les informations sur les partitions
 optimales (déterminées lors de notre sélection de modèle);  
- Connaître le modèle le plus approprié pour chacune de nos partitions;
- Il suffit d'avoir suivi les instructions du
 [tutoriel sur la sélection de modèle](HybSeq/08--Selection_modele.md) pour
 avoir ces fichiers.  

Les fichiers d'intérêt sont donc `filtered_concat.partitions.best_scheme.nex` et
 `filtered_concat.phy`, dans votre dossier 
 `/scratch/$USER/HybSeqTest/seqs/exon/align/concat`.

- **Question**: Examinez ces fichiers à l'aider de la commande `more`, et
 assurez-vous que vous comprenez comment ils sont formattés.  

Une fois cela fait, voici du code pour faire une analyse de maximum de
 vraisemblance avec **IQ-Tree**:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_IQ=/opt/iqtree-2.3.6-Linux-intel/bin
WD=/scratch/$USER/HybSeqTest
ALIGNMENT=$WD/align/taper/trimal/concat/filtered_concat.nex
PARTITIONS=$WD/align/taper/trimal/concat/filtered_concat.partitions.best_scheme.nex
OUTPUT_PREFIX=Carex
BOOTSTRAP_REPS=20
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## Maximum de vraisemblance avec IQ-Tree
###

mkdir -p $WD/trees/concat/iqtree
cd $WD/trees/concat/iqtree

ln $ALIGNMENT $OUTPUT_PREFIX.phy
ln $PARTITIONS $OUTPUT_PREFIX.part

## Exécuter IQ-TREE sur SLURM
sbatch \
  --job-name=iqtree.search_bootstrap \
  --output=iqtree.search_bootstrap.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=4 \
  --mem-per-cpu=2G \
  --wrap="$SRC_IQ/iqtree2 \
    -s $OUTPUT_PREFIX.phy \
    -p $OUTPUT_PREFIX.part \
    -nt 4 \
    -b $BOOTSTRAP_REPS"

```

Pour une description des paramètres et fichiers de sortie, voir le tutoriel sur
 le maximum de vraisemblance dans le dossier "Sanger".

---

## Maximum de vraisemblance dans RAxML

**RAxML** est moins flexible que IQ-Tree, mais très efficace à faire ce qu'il
 fait le mieux: estimer très rapidement un arbre de maximum de vraisemblance en
 utilisant un modèle GTR ou GTR+Gamma. Il est aussi rapide pour le calcul de
 support bootstrap. En conséquence, il s'agit du programme d'analyse de maximum
 de vraisemblance le plus populaire à ce jour.

Prérequis:
- Avoir un fichier en format phylip (.phy) contenant la concaténation de tous
 les loci;  
- Avoir un fichier associé qui donne les informations sur les partitions
 optimales 
(déterminées lors de notre sélection de modèle);  
- Connaître le modèle le plus approprié pour chacune de nos partitions;
- Il suffit d'avoir suivi les instructions du 
 [tutoriel sur la sélection de modèle](HybSeq/07--Selection_modele.md) pour
 avoir ces fichiers.  

Les fichiers d'intérêt sont donc `filtered_concat.partitions.best_scheme` et
 `filtered_concat.phy`, dans votre dossier
 `/scratch/$USER/HybSeqTest/align/taper/trimal/concat/`.

- **Question**: Examinez ces fichiers à l'aider de la commande `more`, et
 assurez-vous que vous comprenez comment ils sont formattés.  

Une fois cela fait, voici du code pour faire une analyse de maximum de
 vraisemblance avec **RAxML**:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_RAXML=/opt/RAxML-8.2.12
WD=/scratch/$USER/HybSeqTest
ALIGNMENT=$WD/align/taper/trimal/concat/filtered_concat.phy
PARTITIONS=$WD/align/taper/trimal/concat/filtered_concat.partitions.best_scheme
MODEL=GTRCAT
OUTPUT_PREFIX=Carex
BOOTSTRAP_REPS=20
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## Maximum de vraisemblance avec RAxML
###

mkdir -p $WD/trees/concat/raxml
cd $WD/trees/concat/raxml

ln $ALIGNMENT $OUTPUT_PREFIX.phy
cp $PARTITIONS $OUTPUT_PREFIX.part

## le début de chaque ligne dans le fichier de partitions doit être "DNA" pour
## que RAxML fonctionne
sed -i 's/DNAF,/DNA,/g' $OUTPUT_PREFIX.part

## Exécuter ce batchfile en mode non-intéractif sur SLURM
sbatch \
  --job-name=raxml \
  --output=raxml.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=4 \
  --mem-per-cpu=2G \
  --wrap="$SRC_RAXML/raxmlHPC-PTHREADS-SSE3 \
    -f a \
    -T 4 \
    -p 1234 \
    -x 1234 \
    -m $MODEL \
    -# $BOOTSTRAP_REPS \
    -s $OUTPUT_PREFIX.phy \
    -q $OUTPUT_PREFIX.part \
    -n $OUTPUT_PREFIX"

```

Pour une description des paramètres et fichiers de sortie, voir le tutoriel sur
 le maximum de vraisemblance dans le dossier "Sanger".

---

## Questions et exercices

1. **Interprétation des fichiers d'entrée** :
   - Examinez les fichiers `filtered_concat.partitions.best_scheme.nex` et
   `filtered_concat.phy` à l'aide de la commande `more`. Expliquez comment ils
   sont structurés et leur rôle dans l'analyse.

2. **Examen du résultat** :
   - Téléchargez le fichier de sortie `*.nex.contree` et `*.nex.treefile` qui
   contiennent les résultats phylogénétiques de l'analyse. Examinez-les dans
   **FigTree**. Comment se comparent-ils avec les résultats des analyses de
   distance et de parcimonie sur le même jeu de données?

2. **Optimisation du modèle de substitution** :
   - Expliquez pourquoi il est important de sélectionner le bon modèle de
   substitution pour chaque partition.   
   
3. **Analyse de bootstrap** :
   - Expliquez le rôle du bootstrap dans une analyse de maximum de
   vraisemblance. Pourquoi est-il nécessaire d'effectuer des réplicats
   bootstrap?
   - Que représente une valeur bootstrap élevée dans un arbre phylogénétique?
   Proposez un seuil pour considérer une branche comme statistiquement
   significative.
   
4. **Parallélisation des tâches** :
   - Pourquoi est-il avantageux d'utiliser plusieurs threads (`-nt 4`) dans une
   analyse IQ-TREE? Comment la parallélisation affecte-t-elle la durée de
   l'analyse?
   
5. **Pratique** :
   - Modifiez le script SLURM pour augmenter le nombre de réplicats bootstrap à
   500. Quelle est l'impact de cette modification sur le temps d'exécution
   prévu?
   - Utilisez les fichiers de sortie `*.log` pour vérifier si l'analyse s'est
   terminée avec succès. Quels indicateurs vous permettent de confirmer cela?
