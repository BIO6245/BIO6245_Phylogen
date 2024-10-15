# Sélection des modèles, partitions et arbres de distance

Ce tutoriel vous guidera dans l’utilisation de **PAUP\*** pour l'estimation d'arbres de distance.

---

## Arbres de distance avec PAUP\*

Lancez le code ci-dessous pour faire une analyse de parcimonie avec PAUP\* en créer un fichier 
contenant toutes les commandes à envoyer à PAUP\*, puis en soumettant ce fichier sur le SLURM.  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/scratch/$USER/HybSeqTest
ALIGNMENT=/scratch/$USER/HybSeqTest/seqs/exon/align/concat/filtered_concat.nex
OUTPUT_PREFIX=filtered
OUTGROUPS="Polystichum_speciosissimum_SRR14320998"
DISTANCE=JC # spécifie le modèle
RATES=gamma # spécifie qu'on veut modéliser de la variation entre les sites
SHAPE=1 # spécifie le paramètre alpha de la distribution gamma de la variation des taux
ESTFREQ=all # spécifie qu'on veut estimer la fréquence de chaque nucléotide
OBJECTIVE=lsfit # spécifie qu'on utiliser les moindres carrés (least-square fit) plutôt que Minimum evolution (ME)
NREPS_SEARCH=100
SEARCH_PARAMS="multre=yes nchuck=5 chucklen=1"
NREPS_BOOTSTRAP=100
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"

###
## Analyse de distance
###

mkdir -p $WD/trees/exon/concat/distance
cd $WD/trees/exon/concat/distance

## Créer un fichier contenant les commandes à envoyer à PAUP*
echo "
begin paup;

	set autoclose=yes;
	set taxlabels=full;
	set increase=auto autoinc=100 autoclose=yes;
	set warnreset=no warntree=no;

	outgroup $OUTGROUPS;

	set criterion=distance;
	dset distance=$DISTANCE rates=$RATES estFreq=$ESTFREQ objective=$OBJECTIVE;
	hs addseq=random nreps=$NREPS_SEARCH $SEARCH_PARAMS;
	descr 1 / plot=phylogram;
	savetrees file=$OUTPUT_PREFIX.$OBJECTIVE.best.tre format=newick brlens=yes;
	
	boots nreps=$NREPS_BOOTSTRAP search=heuristic grpfreq=no treefile=$OUTPUT_PREFIX.$OBJECTIVE.bootfile / addseq=random nreps=3 multre=yes steepest=no nchuck=3 chucklen=1 limitperrep=yes;
	savetrees from=0 to=1 file=$OUTPUT_PREFIX.$OBJECTIVE.bootstrap.tre format=newick savebootp=nodelab maxdec=0;

	quit;
	
end;" > paup_commands.txt

## Ajouter ce bloc de commandes PAUP à l'alignement pour créer un batchfile pour PAUP*
cat $ALIGNMENT paup_commands.txt > paup_batchfile.nex

## Exécuter ce batchfile en mode non-intéractif sur SLURM
sbatch \
  --job-name=paup \
  --output=paup.log \
  --mail-user=$EMAIL \
  --nodes=1 \
  --time=$TIME \
  --cpus-per-task=4 \
  --mem-per-cpu=2G \
  --wrap="/opt/paup4a168_centos64 --noninteractive paup_batchfile.nex"

```

- **Question**: Après que l'analyse ait terminée, examiner le logfile avec la commande 
`more paup.log`. Comprenez-vous chaque élément à l'intérieur de ce logfile?  
- **Question**: Comment la phylogénie estimée par la méthode de distance se compare à celle 
estimée par la parcimonie?  
- **Question**: Est-ce que le modèle d'évolution spécifié est le plus approprié pour le jeu de 
données, basée sur l'analyse de sélection de modèle qu'on a fait dans le tutoriel précédent? Si 
non, quel serait le meilleur modèle?  
- **Question**: Changez le modèle utilisé pour l'analyse, et rouler une deuxième fois l'analyse. 
Les résultats sont-ils différents? En quoi?  
- **Question**: Changez le critère d'optimisation à Minimum Evolution (plutôt que least-square), 
et roulez à nouveau l'analyse. Qu'est-ce qui change?  
- **Question**: Comment faire pour estimer un arbre avec le neighbor-joining? Ajustez le code 
pour le faire et re-roulez l'analyse.  