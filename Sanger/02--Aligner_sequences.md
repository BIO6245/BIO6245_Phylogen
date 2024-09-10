# Aligner séquences Sanger

## Aligner avec MEGA

Le programme [MEGA](https://www.megasoftware.net/) offre gratuitement l'accès à plusieurs algorithmes 
importants d'alignement et d'analyse phylogénétique, le tout organisé dans une interface graphique intuitive. 
Nous allons donc utiliser ce programme pour tester l'effet de différents algorithmes automatisés d'alignement 
des séquences.

Ouvrez le fichier `sequence.fasta` dans MEGA et sélectionnez l'option "Align" (plutôt que "Analyze"). Les 
séquences téléchargez de Genbank ne sont pas alignées. Vous devriez remarquer cela facilement en les 
examinant: vous constaterez que les différentes couleurs correspondant aux quatre différentes bases (A, C, 
T, G) ne sont pas du tout les mêmes sur une même colonne.

Dans les onglets en haut de la fenêtre "Alignment Explorer", sélectionnez "Display" -> "Toggle Conserved 
Sites" -> "at 50% Level". Désormais, les nucléotides de chaque séquence qui sont identiques au consensus 
de chaque colonne (le consensus étant le nucléotide le plus abondant dans sa colonne) ne sont pas colorés. 
Cela permet de plus rapidement remarquer les positions qui sont mal alignées par rapport à la majorité des 
autres séquences. Vous devriez remarquer qu'au moins quelques uns des échantillons ont des nucléotides 
différents du consensus dans presque toutes les colonnes: c'est parce que la séquence de ces échantillons ne 
débute pas au même endroit que les autres séquences.

En cliquant sur une position d'une séquence et en ajoutant ensuite des espaces, il est possible de décaler 
la séquence au complet ou une partie de la séquence vers la droite. Essayer d'ajuster l'alignement 
manuellement en faisant cela. On peut aussi redécaler vers la gauche avec la touche backspace (<-).

L'alignement manuel est plutôt long et pénible. Essayez alors un algorithme d'alignement automatique. Dans 
l'onglet "Alignment", sélectionnez "Align By ClustalW", puis pesez sur OK. 

- **Question:** Regardez le résultat aligné. Est-ce que le nombre de positions qui diffèrent du consensus 
a changé?

Sauvegardez l'alignement en sélectionnant "Data" ->  "Export Alignment" -> "Nexus/PAUP Format", puis changez 
le nom du fichier pour "alignment1.nex".

Ensuite, réalignez vos séquences en utilisant un nouvel algorithme ou des nouveaux paramètres. Par exemple, 
vous pourriez sélectionner "Alignment" -> "Align by MUSCLE et conserver les paramètres par défaut, ou bien 
vous pourriez sélectionner encore "Alignment" -> "Align by ClustalW", mais changer le Gap Opening Penalty 
de 15,00 à 100. Sauvegardez ce nouvel alignment en format NEXUS sous le nom de "alignment2.nex". 

- **Question:** Comparez visuellement le résultat à celui de l'alignement précédent. Y a-t-il des différences 
notables? Si oui, lesquelles, et pourquoi?


## Vérifier les résultats de l'alignement par l'analyse phylogénétique

Fermez les fenêtres ouvertes de MEGA, puis ouvrez une nouvelle fenêtre de MEGA. Ouvrez le fichier 
`alignment1.nex` dans MEGA. Faites des arbres phylogénétiques en utilisant le bouton "PHYLOGENY" et en 
sélectionnant les algorithmes "Neighbor-Joining", puis "Maximum Parsimony", en conservant les paramètres 
par défaut pour chacun. Pour chaque arbre, enracinez sur l'échantillon de *Polystichum* en cliquant-droit 
sur la branche juste en-dessous de l'étiquette *Polystichum*, et en sélectionnant "Root Tree".

- **Question:** Est-ce que les résults diffèrent entre les deux algorithmes utilisés pour estimer la 
phylogénie? Si oui, avez-vous une idée pourquoi?

Fermez les fenêtres actives de MEGA, puis ouvrez le fichier `alignment2.nex` dans MEGA. Refaites des arbres 
en utilisant les mêmes algorithmes. 

- **Question:** Est-ce que les résultats sont différents de ceux du premier alignement?

