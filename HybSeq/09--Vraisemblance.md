# Sélection des modèles, partitions et arbres de distance

Ce tutoriel vous guidera dans l’utilisation d'**IQ-Tree** et de **RAxML** pour l'analyse de 
maximum de vraisemblance (*maximum likelihood*) sur une matrice de données concaténées ou 
sur un seul gène. Il assume qu'on sait déjà quel est le meilleur modèle de substitution.

---

## Maximum de vraisemblance dans IQ-Tree

**IQ-Tree** offre une approche rapide et flexible pour la sélection de modèles et le 
partitionnement. Il permet de tester plusieurs partitions et modèles pour chaque partition.

Prérequis:
- Avoir un fichier en format phylip (.phy) contenant la concaténation de tous les loci;  
- Avoir un fichier associé qui donne les informations sur les partitions optimales 
(déterminées lors de notre sélection de modèle);  
- Connaître le modèle le plus approprié pour chacune de nos partitions;
- Il suffit d'avoir suivi les instructions du 
[tutoriel sur la sélection de modèle](HybSeq/07--Selection_modele.md) pour avoir ces 
fichiers.  

Les fichiers d'intérêt sont donc `filtered_concat.partitions.best_scheme.nex` et `filtered_concat.phy`, dans votre dossier `/scratch/$USER/HybSeqTest/seqs/exon/align/concat`.

- **Question**: Examinez ces fichiers à l'aider de la commande `more`, et assurez-vous que vous comprenez comment ils sont formattés.  

Une fois cela fait, voici du code pour faire une analyse de sélection des modèles et des 
partitions avec **IQ-Tree**:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_IQ=/opt/iqtree-2.3.6-Linux-intel/bin
WD=/scratch/$USER/HybSeqTest/trees/exon/align/concat/iqtree
ALIGNMENT=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/filtered_concat.phy
PARTITIONS=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/filtered_concat.partitions.best_scheme.nex
OUTPUT_PREFIX=Dryopteris
BOOTSTRAP_REPS=100
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## Maximum de vraisemblance avec IQ-Tree
###

mkdir -p $WD
cd $WD

ln $ALIGNMENT $OUTPUT_PREFIX.phy
ln $PARTITIONS $OUTPUT_PREFIX.nex

## Exécuter ce batchfile en mode non-intéractif sur SLURM
sbatch \
  --job-name=modelSelect \
  --output=iqtree.modelSelect.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=4 \
  --mem-per-cpu=2G \
  --wrap="$SRC_IQ/iqtree2 -s $OUTPUT_PREFIX.phy -p $OUTPUT_PREFIX.nex -nt 4 -b $BOOTSTRAP_REPS"

```

1. **`SRC_IQ`** : Spécifie l'emplacement du binaire IQ-TREE sur le système. Ici, il est situé à 
`/opt/iqtree-2.3.6-Linux-intel/bin`.
   
2. **`WD`** : Définit le répertoire de travail où les résultats seront stockés. Dans ce cas, c'est `/scratch/$USER/HybSeqTest/trees/exon/align/concat/iqtree`.

3. **`ALIGNMENT`** : Fichier d'alignement au format **PHYLIP** contenant la concaténation de tous 
les loci. Ce fichier est essentiel pour l'analyse phylogénétique.

4. **`PARTITIONS`** : Fichier définissant les partitions et les modèles de substitution optimaux 
pour chaque partition. Il a été généré lors de l'étape de sélection du modèle.

5. **`OUTPUT_PREFIX`** : Préfixe utilisé pour nommer les fichiers de sortie. Ici, les fichiers de 
sortie auront le préfixe `Dryopteris`.

6. **`BOOTSTRAP_REPS`** : Le nombre de réplicats bootstrap à effectuer. Ici, 100 réplicats sont 
spécifiés pour évaluer la robustesse de l'arbre.

7. **`EMAIL`** : Adresse e-mail de l'utilisateur pour recevoir les notifications lorsque le 
travail est terminé ou en cas d'erreur.

8. **`TIME`** : Le temps maximum alloué pour le travail sur le système SLURM, ici 12 heures.

9. **`-s $OUTPUT_PREFIX.phy`** : Spécifie le fichier d'alignement à utiliser dans l'analyse.

10. **`-p $OUTPUT_PREFIX.nex`** : Indique le fichier de partitions à utiliser dans l'analyse.

11. **`-nt 4`** : Utilise 4 threads pour accélérer l'analyse.

12. **`-b $BOOTSTRAP_REPS`** : Effectue 100 réplicats bootstrap pour estimer la robustesse des 
branches de l'arbre.

### Questions/Exercices pour les étudiants

1. **Interprétation des fichiers d'entrée** :
   - Examinez les fichiers `filtered_concat.partitions.best_scheme.nex` et `filtered_concat.phy` à 
   l'aide de la commande `more`. Expliquez comment ils sont structurés et leur rôle dans l'analyse.

2. **Examen du résultat** :
   - Téléchargez le fichier de sortie `*.nex.contree` et `*.nex.treefile` qui contiennent les 
   résultats phylogénétiques de l'analyse. Examinez-les dans **FigTree**. Comment se 
   comparent-ils avec les résultats des analyses de distance et de parcimonie sur le même jeu 
   de données?

2. **Optimisation du modèle de substitution** :
   - Expliquez pourquoi il est important de sélectionner le bon modèle de substitution pour chaque 
   partition.
   
3. **Analyse de bootstrap** :
   - Expliquez le rôle du bootstrap dans une analyse de maximum de vraisemblance. Pourquoi est-il 
   nécessaire d'effectuer des réplicats bootstrap ?
   - Que représente une valeur bootstrap élevée dans un arbre phylogénétique ? Proposez un seuil 
   pour considérer une branche comme statistiquement significative.
   
4. **Parallélisation des tâches** :
   - Pourquoi est-il avantageux d'utiliser plusieurs threads (`-nt 4`) dans une analyse IQ-TREE ? 
   Comment la parallélisation affecte-t-elle la durée de l'analyse ?
   
5. **Pratique** :
   - Modifiez le script SLURM pour augmenter le nombre de réplicats bootstrap à 500. Quelle est 
   l'impact de cette modification sur le temps d'exécution prévu ?
   - Utilisez les fichiers de sortie `*.log` pour vérifier si l'analyse s'est terminée avec succès. 
   Quels indicateurs vous permettent de confirmer cela ?
   
   
## !!! Ignorer le code ci-dessous !!!   

!!! Écrire analyse avec RAxML !!!

6. **Comparaison des logiciels** :
   - Comparez l'approche de IQ-TREE avec celle de RAxML. Quels sont les avantages et inconvénients de chaque logiciel pour une analyse de maximum de vraisemblance ?