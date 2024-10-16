# Aligner séquences Sanger

Ce tutoriel vous montrera à aligner des séquences Sanger à l'aide de ces programmes: 
  - [MEGA](https://www.megasoftware.net/), interface permettant de générer des alignements avec 
  les algorithmes CLUSTAL et MUSCLE, ainsi que de visualiser les alignements;  
  - [MAFFT](https://mafft.cbrc.jp/alignment/software/), qui est très rapide et généralement plus 
  fiable que les algorithmes précédents, mais  qui n'est pas disponible en interface graphique sur 
  MEGA (on doit passer par la ligne de commandes);  
  - [OMM_MACSE](https://github.com/ranwez/MACSE_V2_PIPELINES), qui est de loin l'approche la plus 
  précise pour l'alignement des nucléotides provenant de gènes (codants ou pseudogénisés), mais qui 
  est plus lente que MAFFT.

---

## Aligner avec MEGA

Le programme [MEGA](https://www.megasoftware.net/) offre gratuitement l'accès à plusieurs 
algorithmes importants d'alignement et d'analyse phylogénétique, le tout organisé dans une 
interface graphique intuitive. Nous allons donc utiliser ce programme pour tester l'effet de différents algorithmes automatisés d'alignement des séquences.

Ouvrez le fichier `sequence.fasta` dans MEGA et sélectionnez l'option "Align" (plutôt que 
"Analyze"). Les séquences téléchargez de Genbank ne sont pas alignées. Vous devriez remarquer 
cela facilement en les examinant: vous constaterez que les différentes couleurs correspondant 
aux quatre différentes bases (A, C, T, G) ne sont pas du tout les mêmes sur une même colonne.

Dans les onglets en haut de la fenêtre "Alignment Explorer", sélectionnez "Display" -> "Toggle 
Conserved Sites" -> "at 50% Level". Désormais, les nucléotides de chaque séquence qui sont 
identiques au consensus de chaque colonne (le consensus étant le nucléotide le plus abondant dans 
sa colonne) ne sont pas colorés. Cela permet de plus rapidement remarquer les positions qui sont 
mal alignées par rapport à la majorité des autres séquences. Vous devriez remarquer qu'au moins  
quelques uns des échantillons ont des nucléotides différents du consensus dans presque toutes les 
colonnes: c'est parce que la séquence de ces échantillons ne débute pas au même endroit que les  
autres séquences.

En cliquant sur une position d'une séquence et en ajoutant ensuite des espaces, il est possible de 
décaler la séquence au complet ou une partie de la séquence vers la droite. Essayer d'ajuster 
l'alignement manuellement en faisant cela. On peut aussi redécaler vers la gauche avec la touche 
backspace (<-).

L'alignement manuel est plutôt long et pénible. Essayez alors un algorithme d'alignement automatique.

Dans l'onglet "Alignment", sélectionnez "Align By ClustalW", puis pesez sur OK.

- **Question:** Regardez le résultat aligné. Est-ce que le nombre de positions qui diffèrent du 
consensus a changé?

Sauvegardez l'alignement en sélectionnant "Data" ->  "Export Alignment" -> "Nexus/PAUP Format", puis 
changez le nom du fichier pour "alignment1.nex".

Ensuite, réalignez vos séquences en utilisant un nouvel algorithme ou des nouveaux paramètres. Par 
exemple, vous pourriez sélectionner "Alignment" -> "Align by MUSCLE et conserver les paramètres par 
défaut, ou bien vous pourriez sélectionner encore "Alignment" -> "Align by ClustalW", mais changer 
le Gap Opening Penalty de 15,00 à 100. Sauvegardez ce nouvel alignment en format NEXUS sous le nom 
de "alignment2.nex".

- **Question:** Comparez visuellement le résultat à celui de l'alignement précédent. Y a-t-il des 
différences notables? Si oui, lesquelles, et pourquoi?

---

## Vérifier les résultats de l'alignement par l'analyse phylogénétique

Fermez les fenêtres ouvertes de MEGA, puis ouvrez une nouvelle fenêtre de MEGA. Ouvrez le fichier 
`alignment1.nex` dans MEGA. Faites des arbres phylogénétiques en utilisant le bouton "PHYLOGENY" 
et en sélectionnant les algorithmes "Neighbor-Joining", puis "Maximum Parsimony", en conservant les 
paramètres par défaut pour chacun. Pour chaque arbre, enracinez sur l'échantillon de *Polystichum* 
en cliquant-droit sur la branche juste en-dessous de l'étiquette *Polystichum*, et en sélectionnant 
"Root Tree".

- **Question:** Est-ce que les résults diffèrent entre les deux algorithmes utilisés pour estimer 
la phylogénie? Si oui, avez-vous une idée pourquoi?

Fermez les fenêtres actives de MEGA, puis ouvrez le fichier `alignment2.nex` dans MEGA. Refaites 
des arbres en utilisant les mêmes algorithmes. 

- **Question:** Est-ce que les résultats sont différents de ceux du premier alignement?

---

## Alignement des indels

Les logiciels d'alignement de séquence automatisés, comme **MAFFT**, **ClustalW**, ou **MUSCLE**, 
sont des outils puissants mais peuvent parfois commettre des erreurs dans l'alignement de 
séquences. Ces erreurs se produisent principalement pour deux raisons:  

1. **Complexité des indels (insertions/délétions)**: Les régions où des indels sont présents sont 
souvent difficiles à aligner correctement. Les logiciels essaient de minimiser le nombre de gaps 
dans l'alignement, mais peuvent mal interpréter la position ou l'ampleur des indels, entraînant des 
erreurs d'alignement.
  
2. **Séquences divergentes** : Lorsque les séquences sont très différentes (éloignées 
évolutivement), les logiciels ont du mal à identifier des homologies exactes entre les résidus, ce 
qui peut mener à des alignements incorrects ou non optimaux.

---

### Utilisation du score de parcimonie pour ajuster les alignements

Le **score de parcimonie** est un critère objectif qui peut être utilisé pour ajuster les 
alignements manuellement et améliorer leur qualité. Ce principe repose sur l'idée que l'alignement 
le plus simple (c'est-à-dire celui qui postule le moins de changements évolutifs) est souvent le 
plus probable.

Voici comment ce critère peut aider à ajuster un alignement:  

1. **Correction des indels** : Les indels (gaps) peuvent être réajustés en maximisant la 
parcimonie, c'est-à-dire en plaçant les indels de manière à réduire le nombre d'événements 
d'insertion et de délétion nécessaires. Cela se fait en essayant de regrouper les indels en un 
seul événement évolutif au lieu de multiples petites modifications.

2. **Simple Gap Coding** : Une méthode courante est de coder les indels comme des caractères 
binaires (présent = 1, absent = 0) et d’inclure ces informations dans l’analyse. Cela permet de 
prendre en compte les indels de façon plus rigoureuse et d'améliorer l'alignement.

3. **Recalcul du score de parcimonie** : Après ajustement des indels et réarrangement des 
séquences, il est possible de recalculer le score de parcimonie. Un score plus bas indique 
généralement un alignement plus probable du point de vue évolutif.

#### Exercice d'ajustement d'un alignement basé sur le critère de parcimonie

- Téléchargez l'alignement nommé `test.nex` dans le dossier 
[BIO6245_Phylogen/Sanger/fichiers/test.nex](fichiers/test.nex).  
- Chargez cet alignement dans MEGA et effectuez une analyse de parcimonie. Notez le score (le 
meilleur arbre trouvé devrait être 8 mutations de long).
- Ouvrez `test.nex` dans le programme AliView et tentez d'ajuster l'alignement à la main.
- Sauvegardez le nouvel alignement sous le nom de `test2.nex`.
- Chargez `test2.nex` dans MEGA, ré-effectuez l'analyse de parcimonie. Est-ce que le score s'est 
amélioré? Notez qu'un score plus bas est meilleur, car ça signifie qu'un plus petit nombre de 
mutations sont nécessaires pour expliquer les données.
- Si le score ne s'est pas amélioré, réessayez d'ajuster l'alignement jusqu'à temps que le score 
s'améliore.

---

## Aligner avec MAFFT

### Transférer le fichier .fasta

**Transférez le fichier Fasta** [`sequence.fasta`] que vous aviez obtenu depuis Genbank vers le 
serveur de calcul à l'aide de la commande `rsync`. Utilisez cette commande dans un 
terminal (sur votre ordinateur, pas le serveur):  
```bash
## Rouler le code ci-dessous à partir de votre machine locale

## Ajuster les variables ci-dessous de façon appropriée
CLUSTER_USERNAME=Votre_nom_utilisateur_serveur
LOCAL=/Chemin/vers/sequence.fasta
REMOTE=/home/$CLUSTER_USERNAME

rsync --progress $LOCAL $CLUSTER_USERNAME@aphidzen.irbv.umontreal.ca:$REMOTE/
```

Notez que si vous êtes sur Windows, il faut remplacer les "backslash" (\\) dans votre chemin avec 
des "forward slash" (\/). De plus, le chemin "C:/" doit être remplacé par "/mnt/c/". Finalement, 
notez que les espaces dans les noms des dossiers et fichiers causent problème sur tous les 
terminal Linux/Mac. Il y a deux solutions pour les espaces: soit ne jamais utiliser d'espaces dans 
vos noms de fichier, ou soit entourer les espaces par des guillemets simples ('), ou précéder 
chaque espace par un "backslash" (\\) lorsque vous spécifiez un chemin vers un dossier ou fichier. 
Par exemple:  
- Le chemin Windows: `C:\Users\Moi\Documents\BIO 6245\ficher de travail.txt`  
- Doit être remplacé par: `/mnt/c/Users/Moi/Documents/BIO' '6245/fichier' 'de' 'travail.txt`  
- Ou bien par: `/mnt/c/Users/Moi/Documents/BIO\ 6245/fichier\ de\ travail.txt`

### Alignement

Une fois que le fichier `sequence.fasta` a été transféré vers le serveur, effectuer l'alignement 
avec MAFFT en roulant cette commande. MAFFT est exécuté avec l'option `--auto`, qui choisit 
automatiquement les meilleurs paramètres pour l'alignement. Le fichier de sortie est nommé 
`sequence.fasta.aligned.fasta`:  
```bash
## Ajuster les variables ci-dessous de façon appropriée
WD=/home/$USER/
SEQIN=sequence.fasta
EMAIL=votre.courriel@umontreal.ca
TIME="0-12:00:00"
THREADS=1

###
## aligner les séquences
###

mkdir -p $WD
cd $WD

## create a SLURM batchfile to align with MAFFT
echo '#!/bin/bash' > mafft.sbatch
echo "#SBATCH --job-name=mafft
#SBATCH --output=mafft.out
#SBATCH --mail-type=end
#SBATCH --nodes=1
#SBATCH --cpus-per-task=$THREADS
#SBATCH --mem-per-cpu=2G
#SBATCH --time=$TIME

mafft --thread $THREADS --auto $SEQIN > $SEQIN.aligned.fasta" >> mafft.sbatch

## Envoyer la tâche d'alignement à SLURM
conda activate base
sbatch --mail-user=$EMAIL mafft.sbatch

```

1. **Interprétation des options de MAFFT** : Que signifie l'option `--auto` dans MAFFT, et pourquoi est-elle utile pour l'alignement de séquences?
   
2. **Exécution de tâches sur SLURM** : Quel est l'intérêt d'utiliser SLURM pour soumettre des tâches sur un serveur de calcul ? Quels avantages cela offre-t-il par rapport à une exécution locale?

3. **Exercice 1** : Modifiez le script pour utiliser 4 CPU au lieu de 1 pour paralléliser l'alignement avec MAFFT. Quelle commande ajouteriez-vous dans le script SLURM pour y parvenir?

4. **Exercice 2** : Une fois l'alignement terminé, utilisez la commande `more` pour examiner le fichier `sequence.fasta.aligned.fasta`. Combien de séquences ont été alignées, et quelle est la longueur de l'alignement final?

5. **Exercice 3** : Téléchargez le résultat `sequence.fasta.aligned.fasta` sur votre ordinateur local 
en utilisant la commande rsync. Ensuite, ouvrez l'alignement avec MEGA ou AliView, et comparez-le avec 
votre alignement fait dans MEGA avec CLUSTAL ou MUSCLE. Y a-t-il des différences? Quel alignement 
semble être le plus fiable?

---

## Aligner avec OMM_MACSE

!!! Continuer ici !!!

