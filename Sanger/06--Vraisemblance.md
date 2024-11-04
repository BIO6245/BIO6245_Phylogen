# Maximum de vraisemblance

Ce tutoriel vous guidera dans l’utilisation d'**IQ-Tree** et de **RAxML** pour l'analyse de 
maximum de vraisemblance (*maximum likelihood*) sur une matrice de données concaténées provenant 
d'un seul gène. Il assume qu'on sait déjà quel est le meilleur modèle de substitution.

---

## Maximum de vraisemblance dans IQ-Tree

**IQ-Tree** permet de faire des analyses de maximum de vraisemblance sur à peu près n'importe quel 
modèle d'évolution et de partitionnement imaginable. Cette flexibilité vient avec un coût: il est 
plus lent que RAxML, particulièrement avec les jeux de données qui sont un peu plus grands.

Prérequis:
- Avoir un fichier en format phylip (.phy) ou nexus (.nex) contenant l'alignement d'un seul gène;
- Avoir un fichier associé qui donne les informations sur les partitions optimales 
(déterminées lors de notre sélection de modèle);  
- Connaître le modèle le plus approprié pour chacune de nos partitions;
- Il suffit d'avoir suivi les instructions du 
[tutoriel sur la sélection de modèle](Sanger/05--Selection_modele-distance.md) pour avoir ces 
fichiers.  

Les fichiers d'intérêt sont donc `partitions.txt.best_scheme.nex` et `test.phy`, dans votre dossier
`/home/$USER`.

- **Question**: Examinez ces fichiers à l'aider de la commande `more`, et assurez-vous que vous 
comprenez comment ils sont formattés.  

Une fois cela fait, voici du code pour faire une analyse de maximum de vraisemblance avec 
**IQ-Tree**:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_IQ=/opt/iqtree-2.3.6-Linux-intel/bin
WD=/home/$USER/test_iqtree
ALIGNMENT=/home/$USER/test.nex
PARTITIONS=/home/$USER/partitions.txt.best_scheme.nex
OUTPUT_PREFIX=test
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
  --job-name=iqtree_search_bootstrap \
  --output=iqtree_search_bootstrap.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=4 \
  --mem-per-cpu=2G \
  --wrap="$SRC_IQ/iqtree2 -s $OUTPUT_PREFIX.phy -p $OUTPUT_PREFIX.nex -nt 4 -b $BOOTSTRAP_REPS"

```

### Description des paramètres dans l'analyse IQ-Tree

Voici une description ajustée des paramètres utilisés dans le code IQ-Tree:

1. **`SRC_IQ`** : Chemin vers le binaire IQ-TREE sur le système, défini ici comme 
`/opt/iqtree-2.3.6-Linux-intel/bin`.

2. **`WD`** : Répertoire de travail où les résultats seront sauvegardés. Dans cet exemple, il 
s'agit de `/home/$USER/test_iqtree`.

3. **`ALIGNMENT`** : Chemin vers le fichier d'alignement en entrée, spécifié ici comme 
`/home/$USER/test.nex`.

4. **`PARTITIONS`** : Fichier de partitions indiquant les modèles de substitution optimaux pour 
chaque partition. Ce fichier est situé à `/home/$USER/partitions.txt.best_scheme.nex`.

5. **`OUTPUT_PREFIX`** : Préfixe pour les fichiers de sortie, défini comme `test` dans cet exemple, 
ce qui signifie que les fichiers de sortie porteront ce préfixe.

6. **`BOOTSTRAP_REPS`** : Nombre de réplicats bootstrap à exécuter, fixé ici à 100, pour évaluer la 
robustesse des branches de l’arbre.

7. **`EMAIL`** : Adresse courriel de l’utilisateur pour recevoir des notifications une fois le 
travail terminé.

8. **`TIME`** : Durée maximale allouée pour le travail sur SLURM, ici configurée pour 12 heures.

9. **`-s $OUTPUT_PREFIX.phy`** : Paramètre indiquant le fichier d’alignement à utiliser dans 
l’analyse.

10. **`-p $OUTPUT_PREFIX.nex`** : Paramètre spécifiant le fichier de partitions pour l’analyse.

11. **`-nt 4`** : Utilisation de 4 threads pour accélérer l’analyse IQ-Tree.

12. **`-b $BOOTSTRAP_REPS`** : Effectue 100 réplicats bootstrap pour estimer la robustesse des 
branches de l’arbre phylogénétique.

### Fichiers de sortie de IQ-Tree

Voici les descriptions ajustées des fichiers de sortie générés par IQ-Tree:

1. **`test.treefile`** :
   - Ce fichier contient l’arbre de maximum de vraisemblance inféré par IQ-Tree avec les longueurs 
   de branches et la topologie optimisées selon le modèle choisi. C'est le résultat principal de 
   l’analyse phylogénétique.

2. **`test.log`** :
   - Un journal d’exécution détaillé de l’analyse, incluant les paramètres utilisés, les modèles de 
   substitution appliqués à chaque partition et des statistiques sur la convergence de l’arbre.

3. **`test.iqtree`** :
   - Ce fichier présente les détails de l’analyse finale, comme la log-vraisemblance de l’arbre, 
   les informations sur les partitions et les modèles de substitution. Il est essentiel pour 
   examiner la reproductibilité de l’analyse.

4. **`test.bionj`** :
   - Un arbre initial de type NJ (Neighbor-Joining) utilisé comme point de départ pour optimiser 
   l’arbre de maximum de vraisemblance final.

5. **`test.model.gz`** :
   - Un fichier compressé contenant les informations sur les modèles de substitution pour chaque 
   partition, utile pour réutiliser les modèles dans des analyses similaires ou approfondir 
   l’examen des paramètres.

6. **`test.ckp.gz`** :
   - Un fichier de checkpoint permettant de reprendre l’analyse en cas d’interruption. IQ-TREE peut 
   utiliser ce fichier pour continuer à partir de l’endroit où l’analyse s'est arrêtée.

7. **`test.splits.nex`** :
   - Ce fichier contient les informations sur les clades supportés par les réplicats bootstrap. Il 
   peut être visualisé dans des logiciels de phylogénie comme SplitsTree.

8. **`test.bootstrap`** :
   - Les arbres bootstrap générés pour estimer la robustesse des branches de l’arbre final. Chaque 
   arbre correspond à un réplicat bootstrap distinct.

9. **`test.contree`** :
   - L’arbre de consensus basé sur les réplicats bootstrap, avec les valeurs de support des 
   branches. Il fournit un aperçu de la stabilité des relations observées dans l’arbre ML.

10. **`iqtree.search_bootstrap.log`** :
   - Un journal SLURM contenant les informations sur l’exécution, comme le temps, les ressources 
   utilisées, et les messages d’erreurs éventuels.

---


!!! Le code ci-dessous n'est pas terminé !!!
!!! En attendant, vous pouvez suivre le tutoriel sur le maximum de vraisemblance dans le dossier HybSeq !!!
!!! Ce tutoriel vous montrera comment utiliser RAxML (c'est la même chose que pour les données Sanger) !!!


## Maximum de vraisemblance dans RAxML

**RAxML** est moins flexible que IQ-Tree, mais très efficace à ce qu'il fait le mieux: estimer très 
rapidement un arbre de maximum de vraisemblance en utilisant un modèle GTR ou GTR+Gamma. Il est 
aussi rapide pour le calcul de support bootstrap. Ces caractéristiques en font le programme 
d'analyse de maximum de vraisemblance le plus populaire à ce jour.

Prérequis: les mêmes que pour l'analyse de maximum de vraisemblance avec IQ-Tree.

**RAxML** ne fonctionne qu'avec des fichiers de type phylip (`.phy`) en entrée. En conséquence, il faut tout d'abord 
transformer le fichier de type nexus (`.nex`) en fichier de type phylip. Nous allons faire ça avec la fonction `export` de 
PAUP\*. 
```bash
SRC_PAUP=/opt/


```

Ensuite, on lance l'analyse RAxML.
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_RAXML=/opt/RAxML-8.2.12
WD=/scratch/$USER/HybSeqTest/trees/exon/align/concat/raxml
ALIGNMENT=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/filtered_concat.phy
PARTITIONS=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/filtered_concat.partitions.best_scheme
MODEL=GTRCAT
OUTPUT_PREFIX=test
BOOTSTRAP_REPS=100
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## Maximum de vraisemblance avec RAxML
###

mkdir -p $WD
cd $WD

ln $ALIGNMENT $OUTPUT_PREFIX.phy
cp $PARTITIONS $OUTPUT_PREFIX.part
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
    $ARGUMENTS_FOR_RAXML \
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

### Description des paramètres dans l'analyse RAxML

1. **`SRC_RAXML`** : Indique l'emplacement du programme RAxML (version 8.2.12) sur le système. Cela permet de spécifier le chemin exact pour exécuter RAxML.

2. **`WD`** : Répertoire de travail où les fichiers de sortie seront enregistrés. Ici, il s'agit de `/scratch/$USER/HybSeqTest/trees/exon/align/concat/raxml`.

3. **`ALIGNMENT`** : Fichier d'alignement au format **PHYLIP** contenant la concaténation de tous les loci. Ce fichier est essentiel pour construire l'arbre de maximum de vraisemblance.

4. **`PARTITIONS`** : Fichier de partitions optimales, contenant les informations sur la manière dont les données sont partitionnées en différents blocs et le modèle de substitution associé à chaque partition. Ce fichier est copié et modifié pour être compatible avec RAxML (via `sed`).

5. **`MODEL`** : Modèle de substitution utilisé dans l'analyse. Ici, le modèle **GTRCAT** (une variante du modèle GTR avec approximation CAT pour la distribution du taux de substitution parmi les sites) est utilisé, car il est plus rapide et convient bien pour les grandes analyses phylogénétiques.

6. **`OUTPUT_PREFIX`** : Préfixe utilisé pour nommer les fichiers de sortie générés par RAxML. Ici, le préfixe est `Dryopteris`, donc tous les fichiers de sortie commenceront par ce nom.

7. **`BOOTSTRAP_REPS`** : Nombre de réplicats bootstrap à effectuer pour évaluer la robustesse des branches de l'arbre. Ici, 100 réplicats sont spécifiés.

8. **`EMAIL`** : Adresse e-mail de l'utilisateur pour recevoir des notifications à la fin de l'exécution du travail (ou en cas d'erreur).

9. **`TIME`** : Temps maximum alloué pour l'exécution du travail. Ici, 12 heures ont été allouées.

10. **`-f a`** : Option pour indiquer que RAxML doit exécuter à la fois une analyse de maximum de vraisemblance (ML) et une analyse bootstrap en une seule exécution.

11. **`-T 4`** : Indique le nombre de threads à utiliser pour cette analyse. Ici, 4 threads sont spécifiés pour accélérer l'exécution.

12. **`-p 1234`** : Graine pour le générateur de nombres aléatoires utilisé dans la recherche de l'arbre ML. Cela permet de rendre l'analyse reproductible.

13. **`-x 1234`** : Graine pour le générateur de nombres aléatoires utilisé dans l'analyse bootstrap. Cela garantit que les réplicats bootstrap sont reproductibles.

14. **`-m $MODEL`** : Spécifie le modèle de substitution choisi (ici, GTRCAT).

15. **`-# $BOOTSTRAP_REPS`** : Définit le nombre de réplicats bootstrap à exécuter (100 dans cet exemple).

16. **`-s $OUTPUT_PREFIX.phy`** : Indique le fichier d'alignement à utiliser pour l'analyse.

17. **`-q $OUTPUT_PREFIX.part`** : Spécifie le fichier de partitions à utiliser.

18. **`-n $OUTPUT_PREFIX`** : Définit le nom des fichiers de sortie, qui commenceront par le préfixe `Dryopteris`.

### Fichiers de sortie de RAxML

1. **`RAxML_info.Dryopteris`** : Fichier d'information qui contient un résumé de l'analyse, y compris les paramètres utilisés, le modèle de substitution, les partitions, et des détails sur les réplicats bootstrap.

2. **`RAxML_bestTree.Dryopteris`** : Contient l'arbre de maximum de vraisemblance optimal trouvé par RAxML.

3. **`RAxML_bootstrap.Dryopteris`** : Ce fichier contient les 100 réplicats bootstrap générés lors de l'analyse.

4. **`RAxML_bipartitions.Dryopteris`** : Arbre de consensus des réplicats bootstrap avec les valeurs de support bootstrap annotées sur chaque branche.

5. **`RAxML_bipartitionsBranchLabels.Dryopteris`** : Semblable à `RAxML_bipartitions.Dryopteris`, mais avec des étiquettes supplémentaires pour les branches internes, représentant les valeurs bootstrap.

6. **`raxml.log`** : Fichier journal qui enregistre toutes les étapes du calcul et toute erreur éventuelle pendant l'exécution.

---

## Questions et exercices

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
   
