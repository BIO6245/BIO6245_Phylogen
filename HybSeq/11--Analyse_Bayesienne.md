# Analyse phylogénétique Bayesienne

## Analyse bayesienne dans MrBayes

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
 `/scratch/$USER/HybSeqTest/align/taper/trimal/concat/`.

- **Question**: Examinez ces fichiers à l'aider de la commande `more`, et
 assurez-vous que vous comprenez comment ils sont formattés.  

Une fois cela fait, voici du code pour faire une analyse de maximum de
 vraisemblance avec **MrBayes**:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
SRC_MB=/opt/MrBayes-mpi/src
WD=/scratch/$USER/HybSeqTest
ALIGNMENT=$WD/align/taper/trimal/concat/filtered_concat.nex
PARTITIONS=$WD/align/taper/trimal/concat/filtered_concat.partitions.best_scheme
OUTPUT_PREFIX=Carex
CHAIN_LENGTH=100000
NCPUS=4
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## Créer un fichier contenant les commandes à lancer dans MrBayes
###

## créer un dossier avec un lien vers l'alignement et les partitions
mkdir -p $WD/trees/concat/mrbayes
cd $WD/trees/concat/mrbayes

ln $ALIGNMENT $OUTPUT_PREFIX.nex
ln $PARTITIONS $OUTPUT_PREFIX.part

## Créer des partitions lisibles avec MrBayes
NPARTS=$(grep -c '=' $OUTPUT_PREFIX.part)
awk -F' = ' '{gsub(/,/, "", $2); printf "  charset p%d = %s;\n", NR, $2}' \
  $OUTPUT_PREFIX.part > mrbayes.part
echo "  partition iqtreeBest = $NPARTS: $(seq -s, -f 'p%g' 1 "$NPARTS");" >> mrbayes.part

## Créer un fichier contenant les commandes à envoyer à MrBayes
echo "
begin mrbayes;" > mrbayes_commands.txt
cat mrbayes.part >> mrbayes_commands.txt
echo "
  set autoclose=yes seed=12345 swapseed=12345;
  
  set partition = iqtreeBest;
  unlink revmat=(all) statef=(all) shape=(all);
  lset applyto=(all) nst=6 rates=gamma;
  showmodel;
  
  mcmcp ng=$CHAIN_LENGTH printf=1000 diagnf=1000 diagnst=maxstddev nch=4 savebr=yes filename=$OUTPUT_PREFIX;
  
  mcmc;
  
end;" >> mrbayes_commands.txt

## Ajouter les commandes à exécuter par MrBayes à la fin de l'alignement .nex
cat $ALIGNMENT $OUTPUT_PREFIX.nex mrbayes_commands.txt > mrbayes_batchfile.nex

## Exécuter MrBayes sur SLURM
sbatch \
  --job-name=mrbayes \
  --output=mrbayes.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --ntasks=$NCPUS \
  --cpus-per-task=1 \
  --mem-per-cpu=2G \
  --wrap="mpirun.openmpi -np $NCPUS $SRC_MB/mb-mpi mrbayes_batchfile.nex"


```