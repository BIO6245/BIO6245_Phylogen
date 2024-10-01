# Sélection des modèles, partitions et arbres de distance

Ce tutoriel vous guidera dans l’utilisation d'**IQ-Tree** pour la sélection des 
modèles de substitution et le partitionnement des données qui serviront dans les analyses 
phylogénétiques probabilistes.

---

## Sélection des modèles et du partitionnement dans IQ-Tree

**IQ-Tree** offre une approche rapide et flexible pour la sélection de modèles et le 
partitionnement. Il permet de tester plusieurs partitions et modèles pour chaque partition.

Prérequis:
- Avoir un fichier en format phylip (.phy) contenant la concaténation de tous les loci;  
- Avoir un fichier associé qui donne les informations sur les partitions;  
- Il suffit d'avoir suivi les instructions du 
[tutoriel sur la parcimonie](HybSeq/06--Parcimonie.md) pour avoir ces fichiers.  

Les fichiers d'intérêt sont donc `filtered_concat.phy` et `filtered_concat.partitions`, dans 
votre dossier `HybSeqTest/seqs/exon/align/concat`.

- **Question**: Examinez ces fichiers à l'aider de la commande `more`, et assurez-vous que vous 
comprenez comment ils sont formattés.  

Une fois cela fait, voici du code pour faire une analyse de sélection des modèles et des 
partitions avec **IQ-Tree**:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_IQ=/opt/iqtree-2.3.6-Linux-intel/bin
WD=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/
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

- **Question**: Quels sont les fichiers créés par l'analyse de sélection de modèles et de 
partitions de IQ-Tree? Que contiennent-ils?  
- **Question**: Combien de partitions ont été sélectionnées?  
- **Question**: Quel est le modèle le plus complexe sélectionné parmi toutes ces partitions?  

