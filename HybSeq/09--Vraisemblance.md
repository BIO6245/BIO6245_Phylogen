# Sélection des modèles, partitions et arbres de distance

Ce tutoriel vous guidera dans l’utilisation d'**IQ-Tree** et de **RAxML** pour l'analyse de 
maximum de vraisemblance (*maximum likelihood*) sur une matrice de données concaténées ou 
sur un seul gène. Il assume qu'on sait déjà quel est le meilleur modèle de substitution.

---

## Maximum de vraisemblance dans IQ-Tree

**IQ-Tree** offre une approche rapide et flexible pour la sélection de modèles et le 
partitionnement. Il permet de faire des analyses de maximum de vraisemblance sur à peu 
près n'importe quel modèle d'évolution et de partitionnement imaginable. Cette flexibilité 
vient avec un coût: il est plus lent que RAxML, particulièrement avec les jeux de données 
qui sont un peu plus grands.

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

Une fois cela fait, voici du code pour faire une analyse de maximum de vraisemblance avec 
**IQ-Tree**:  
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
  --job-name=iqtree.search_bootstrap \
  --output=iqtree.search_bootstrap.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=4 \
  --mem-per-cpu=2G \
  --wrap="$SRC_IQ/iqtree2 -s $OUTPUT_PREFIX.phy -p $OUTPUT_PREFIX.nex -nt 4 -b $BOOTSTRAP_REPS"

```

### Description des paramètres dans l'analyse IQ-Tree

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

### Fichiers de sortie de IQ-Tree

Lorsque vous exécutez IQ-Tree avec le script fourni, plusieurs fichiers de sortie sont générés. Ces fichiers contiennent des informations sur l'arbre de maximum de vraisemblance, les partitions, les réplicats bootstrap et les modèles de substitution utilisés. Voici une description des principaux fichiers de sortie créés par IQ-Tree dans ce script :

1. **`Dryopteris.treefile`** :
   - Ce fichier contient l'arbre de maximum de vraisemblance (ML) inféré par IQ-Tree. C'est le résultat principal de l'analyse, avec les longueurs de branches optimisées et les topologies déterminées en fonction du modèle de substitution choisi.

2. **`Dryopteris.log`** :
   - Un journal détaillé de l'exécution du programme IQ-Tree. Il comprend des informations sur les options choisies pour l'exécution, les modèles de substitution appliqués à chaque partition, les résultats intermédiaires et les statistiques sur la convergence de l'arbre.

3. **`Dryopteris.iqtree`** :
   - Ce fichier fournit des informations complètes sur les paramètres de l'analyse. Il inclut la log-vraisemblance de l'arbre final, des informations sur les partitions, le modèle de substitution utilisé pour chaque partition, et les réplicats bootstrap. Ce fichier est utile pour examiner les détails des résultats et pour la reproductibilité de l'analyse.

4. **`Dryopteris.bionj`** :
   - Un arbre NJ (Neighbor-Joining) initial utilisé par IQ-Tree pour commencer l'optimisation de l'arbre de maximum de vraisemblance. Cet arbre est généré avant le calcul ML final, et sert souvent de point de départ pour la recherche de la meilleure topologie.

5. **`Dryopteris.model.gz`** :
   - Un fichier compressé contenant des informations détaillées sur le modèle de substitution pour chaque partition. Ce fichier peut être utile si vous souhaitez réutiliser les mêmes modèles pour une autre analyse ou examiner de manière approfondie les paramètres de chaque modèle.

6. **`Dryopteris.ckp.gz`** :
   - Un fichier de checkpoint qui permet à IQ-Tree de reprendre une analyse interrompue, par exemple à cause d'une limite de temps dépassée. Si l'analyse est interrompue, IQ-Tree peut être relancé en utilisant ce fichier pour continuer depuis ce point.

7. **`Dryopteris.splits.nex`** :
   - Ce fichier contient les informations sur les clades supportés par les réplicats bootstrap. Il peut être utilisé pour visualiser les bipartitions dans un logiciel comme SplitsTree ou pour des analyses supplémentaires sur les partitions d'arbre.

8. **`Dryopteris.bootstrap`** :
   - Ce fichier contient les arbres bootstrap générés au cours de l'analyse. Ces arbres sont utilisés pour estimer le support bootstrap des branches dans l'arbre final de maximum de vraisemblance. Chaque arbre correspond à un réplicat bootstrap différent.

9. **`Dryopteris.contree`** :
   - L'arbre de consensus basé sur les réplicats bootstrap. Il contient les valeurs de support bootstrap annotées sur les branches. Cet arbre donne une idée de la robustesse des clades et des relations phylogénétiques observées dans l'arbre ML.

10. **`iqtree.search_bootstrap.log`** :
   - Un fichier journal généré par SLURM qui contient toutes les informations relatives à l'exécution du travail (temps d'exécution, ressources utilisées, messages d'erreurs, etc.). Ce fichier est utile pour diagnostiquer les problèmes de calcul.

---

## Maximum de vraisemblance dans RAxML

**RAxML** est moins flexible que IQ-Tree, mais très efficace à faire ce qu'il fait le mieux: 
estimer très rapidement un arbre de maximum de vraisemblance en utilisant un modèle GTR ou 
GTR+Gamma. Il est aussi rapide pour le calcul de support bootstrap. En conséquence, il s'agit du 
programme d'analyse de maximum de vraisemblance le plus populaire à ce jour.

Prérequis:
- Avoir un fichier en format phylip (.phy) contenant la concaténation de tous les loci;  
- Avoir un fichier associé qui donne les informations sur les partitions optimales 
(déterminées lors de notre sélection de modèle);  
- Connaître le modèle le plus approprié pour chacune de nos partitions;
- Il suffit d'avoir suivi les instructions du 
[tutoriel sur la sélection de modèle](HybSeq/07--Selection_modele.md) pour avoir ces 
fichiers.  

Les fichiers d'intérêt sont donc `filtered_concat.partitions.best_scheme` et 
`filtered_concat.phy`, dans votre dossier `/scratch/$USER/HybSeqTest/seqs/exon/align/concat`.

- **Question**: Examinez ces fichiers à l'aider de la commande `more`, et assurez-vous que vous comprenez comment ils sont formattés.  

Une fois cela fait, voici du code pour faire une analyse de maximum de vraisemblance avec 
**RAxML**:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_RAXML=/opt/RAxML-8.2.12
WD=/scratch/$USER/HybSeqTest/trees/exon/align/concat/raxml
ALIGNMENT=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/filtered_concat.phy
PARTITIONS=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/filtered_concat.partitions.best_scheme
MODEL=GTRCAT
OUTPUT_PREFIX=Dryopteris
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

### Questions/Exercices pour les étudiants

1. **Interprétation des fichiers d'entrée** :
   - Ouvrez les fichiers `filtered_concat.phy` et `filtered_concat.partitions.best_scheme` avec la commande `more`. Expliquez leur rôle dans l'analyse et comment ils sont formatés.

2. **Modèles de substitution** :
   - Pourquoi utilise-t-on le modèle **GTRCAT** dans cette analyse ? Quelles sont les différences entre les modèles GTRCAT et GTRGAMMA ?
   - Si vous deviez utiliser un autre modèle de substitution, comment pourriez-vous modifier le script ?

3. **Bootstrap** :
   - Qu'est-ce qu'un réplicat bootstrap, et pourquoi est-il important dans une analyse phylogénétique ?
   - Que signifient des valeurs bootstrap élevées dans un arbre phylogénétique ?

4. **Parallélisation** :
   - Pourquoi utiliser plusieurs threads (`-T 4`) dans cette analyse ? Comment cela affecte-t-il le temps d'exécution ?

5. **Pratique** :
   - Modifiez le script pour exécuter 500 réplicats bootstrap au lieu de 100. Quelle différence observez-vous dans le fichier de sortie en termes de valeurs de support bootstrap ?
   - Analysez le fichier `RAxML_info.Dryopteris`. Quels paramètres de l'analyse y sont répertoriés et pourquoi sont-ils importants pour comprendre les résultats ?


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
   
   
## !!! Ignorer le code ci-dessous !!!   

!!! Écrire analyse avec RAxML !!!

6. **Comparaison des logiciels** :
   - Comparez l'approche de IQ-TREE avec celle de RAxML. Quels sont les avantages et inconvénients de chaque logiciel pour une analyse de maximum de vraisemblance ?