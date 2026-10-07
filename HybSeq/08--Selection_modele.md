# Sélection des modèles, partitions et arbres de distance

Ce tutoriel vous guidera dans l’utilisation d'**IQ-Tree** pour la sélection des
 modèles de substitution et le partitionnement des données qui serviront dans
 les analyses phylogénétiques probabilistes (distance, maximum de vraisemblance
 et bayesienne).
 
La sélection du modèle de substitution est le processus par lequel on estime un
 arbre approximatif (par exemple, avec Neighbor-Joining) puis on estime la
 vraisemblance de différents modèles sur cet arbre et notre jeu de données. En 
 gros, sur cet arbre, on détermine pour différents modèles (JC, GTR,  GTR+G, 
 ...) la vraisemblance, qui indique à quel point le modèle est supporté si on 
 assume que cet arbre est le bon (il est généralement approximativement
 correct). Le modèle qui obtient la plus grande vraisemblance, après avoir
 corrigé pour le nombre de paramètres (corrections AIC ou BIC), est "le 
 meilleur" et sera utilisé pour les analyses subséquentes.
 
Le partitionnement, c'est le processus par lequel on attribue des modèles
 différents à différents sous-groupes de données. Par exemple, vos données ici
 contiennent des séquences de plusieurs gènes. Chaque gène n'évolue pas à la
 même vitesse, donc il serait préférable d'estimer des paramètres de modèles
 différents pour chaque gène.
 
Toutefois, lorsque vous avez un très grand nombre de gènes (plusieurs centaines
 ici), estimer un modèle différent pour chaque gène prend beaucoup de temps, et
 n'est pas très efficace car certains gènes se ressemblent beaucoup en terme de
 taux d'évolution (gènes qui évoluent à la même vitesse). En conséquence, la
 sélection de modèles et de partitions va tenter de combiner les gènes qui ont
 un taux d'évolution similaire en une grande "partition" de données. Une fois
 qu'on aura sélectionné la façon de partitionner idéale, et le modèle idéal pour
 chaque partition, on pourra faire des analyses subséquentes (vraisemblance,
 bayesienne).
 

---

## Sélection des modèles et du partitionnement dans IQ-Tree

**IQ-Tree** offre une approche rapide et flexible pour la sélection de modèles
 et le partitionnement. Il permet de tester plusieurs partitions et modèles pour chaque partition.

Prérequis:
- Avoir un fichier en format phylip (.phy) contenant la concaténation de tous les loci;  
- Avoir un fichier associé qui donne les informations sur les partitions;  
- Il suffit d'avoir suivi les instructions du 
[tutoriel sur la parcimonie](HybSeq/07--Parcimonie.md) pour avoir ces fichiers.  

Les fichiers d'intérêt sont donc `filtered_concat.phy` et
`filtered_concat.partitions`, dans votre dossier
 `HybSeqTest/align/taper/trimal/concat`.

- **Question**: Examinez ces fichiers à l'aider de la commande `more`, et
 assurez-vous que vous comprenez comment ils sont formattés.  

Une fois cela fait, voici du code pour faire une analyse de sélection des modèles et des 
partitions avec **IQ-Tree**:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_IQ=/opt/iqtree-2.3.6-Linux-intel/bin
WD=/scratch/$USER/HybSeqTest/align/taper/trimal/concat/
ALIGNMENT=filtered_concat.phy
PARTITIONS=filtered_concat.partitions
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## Sélection de modèle avec IQ-Tree
###

cd $WD

## Exécuter ce batchfile en mode non-intéractif sur SLURM
sbatch \
  --job-name=modelSelect \
  --output=iqtree.modelSelect.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=4 \
  --mem-per-cpu=2G \
  --wrap="$SRC_IQ/iqtree2 -s $ALIGNMENT -p  $PARTITIONS -m TESTMERGEONLY -nt 4"

```

- **Question**: Quels sont les fichiers créés par l'analyse de sélection de
 modèles et de partitions de IQ-Tree? Que contiennent-ils?  
- **Question**: Combien de partitions ont été sélectionnées?  
- **Question**: Quel est le modèle le plus complexe sélectionné parmi toutes ces
 partitions?  

