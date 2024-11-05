# Estimation d'un arbre d'espèce à partir d'arbres de gènes

Ce tutoriel vous montrera comment estimer un arbre d'espèces à partir des données provenant de 
plusieurs gènes différents, via l'estimation d'arbres de gènes. Il existe deux grandes approches: 
1. Co-estimation des arbres de gènes et de l'arbre d'espèce (p.ex., **\*BEAST** (=**starBEAST**)
2. Estimation des arbres de gènes individuels, puis estimation d'un arbre d'espèce avec une 
méthode par abrégé (p.ex., **ASTRAL**, **svdQuartets**, etc.)

## Co-estimation avec \*BEAST

### Installer BEAST 2

Téléchargez et installez le logiciel **BEAST 2** qui est disponible gratuitement à cette adresse: 
[https://www.beast2.org/](https://www.beast2.org/). 

Dans Windows, pour lancer un module de **BEAST 2** il suffit de dézipper le fichier `.zip`, de 
naviguer à l'intérieur du dossier `BEAST/`, et de double-cliquer sur un des fichiers `.exe`. Par 
exemple, double-cliquer sur `BEAST/BEAUti.exe` pour lancer le module BEAUti.

Dans Mac, il faut installer le programme à l'aide du fichier `.dmg`.

Dans Linux, il faut décompresser l'archive .tar.gz, puis lancer un des fichiers binaires du dossier 
`BEAST/bin` à partir du terminal.

### Préparer l'analyse \*BEAST

Pour lancer une analyse dans **BEAST 2**, il y a deux étapes:
1. Préparer un fichier `.xml` dans le module BEAUti, qui indiquera les paramètres de l'analyse.
2. Exécuter ce fichier `.xml` dans le module BEAST, qui fera l'analyse.

- Commencez par télécharger les trois alignements nommés `Gaufres_gene*.nex` dans le dossier 
[BIO6245_Phylogen/Sanger/fichiers/test.nex](fichiers/test.nex).

- Ensuite, ouvrez le module BEAUti.

- Cliquez sur File -> Template -> StarBeast3.

- Importez les trois alignements `Gaufres_gene*.nex` que vous venez de télécharger (File -> 
Import Alignement). 

- Vous allez voir apparaître chacun de ces alignements sur une ligne distincte dans l'onglet 
"Partitions". Si deux gènes sont physiquement attachés (qu'il n'existe pas de recombinaison 
possible entre ces gènes, par exemple s'ils appartiennent tous deux au génome mitochondrial), il 
faut leur attribuer le même modèle d'arbre.

- Liez le modèle d'arbre des gènes #26 et #29 en sélectionnant le même modèle d'arbre dans la 
colonne Tree de l'onglet Partition (par exemple, sélectionnez "Gaufres_gene26" pour les deux).

- Dans l'onglet "Taxon Sets", il faut indiquer l'espèce à laquelle appartient chaque échantillon 
séquencé, car on peut avoir séquencé plusieurs individus provenant de la même espèce. Le plus 
rapide est de cliquer sur le bouton "Guess", puis d'automatiquement attribuer le nom de l'espèce 
basé sur la structure du nom qu'on a donné aux séquences. Utilisez les éléments 2 et 3 séparés 
par des *underscores* (`_`).

- Puisque les gènes #26 et #29 sont issus du génome mitochondrial, naviguez à l'onglet "Gene 
Ploidy" et changez la ploidie de leur arbre à 1.0. Conservez la ploidie à 2.0 pour l'arbre 
du gène nucléaire #47.

- Dans l'onglet "Site model", ajustez les paramètres pour estimer un modèle GTR+Gamma pour 
chacun des trois gènes.

- Dans l'onglet "Gene Clock Model", assurez-vous que les cases de tous les gènes sont cochées 
pour estimer le taux de mutation individuellement pour chaque gène (qui peut évoluer à une 
vitesse différente selon son rôle fonctionnel, etc.).

- Dans l'onglet "Species Clock Model", spécifiez un "Species Tree Relaxed Clock", qui est 
beaucoup plus réaliste qu'une horloge moléculaire stricte (le défaut).

- Dans l'onglet "Priors", sélectionnez le Birth Death Model comme a priori pour le paramètre 
Tree.t:Species. Cela permet d'estimer à la fois le taux de spéciation et le taux d'extinction, 
ce qui est plus réaliste que l'a priori Yule qui assume l'absence totale d'extinction.

- Dans l'onglet "MCMC", ajustez la chaîne MCMC comme vous le voulez. C'est la même chose que pour 
les analyses Bayesiennes dans MrBayes (référez-vous au tutoriel de MrBayes).

- Une fois que tous les paramètres sont bien sélectionnés, sauvegardez le fichier de paramètres 
avec File -> Save As. Nommez-le `Gaufres_starBeast.xml`.

### Analyse Bayesienne \*BEAST

- Ouvrez le module BEAST

- Choisissez `Gaufres_starBEAST.xml` comme "Input File".

- Lancez 2 chaînes ("Instances" = 2) qui utilisent chacune 2 processeurs ("Threads" = 2) et 
cliquez sur "Run".

- Vous allez pouvoir suivre le progrès des chaînes à l'écran.

- Pendant que l'analyse a lieu, ouvrez le fichier `starbeast3.log` dans **Tracer** pour déterminer 
si les chaînes MCMC ont roulé suffisamment longtemps pour bien estimer tous les paramètres du 
modèle.

- Pendant que l'analyse a lieu, ouvrez le module DensiTree de **BEAST 2**. Dans ce module, 
ouvrez successivement les fichiers `species.trees`, `Gaufres_gene26.trees` et 
`Gaufres_gene49.trees` pour visualiser l'estimation de l'arbre d'espèce et des arbres de gènes 
par \*BEAST.

- Lorsque vous avez roulé les chaînes MCMC suffisamment longtemps, ouvrez le module TreeAnnotator 
de **BEAST 2**. Dans ce module, déterminez le pourcentage des itérations MCMC à mettre à la 
poubelle (burnin), puis sélectionnez `species.trees` comme "Input Tree File" et nommez le 
output tree file `best.starBeast.species.tre`. 

- Fermez le module TreeAnnotator et ouvrez le fichier `best.starBeast.species.tre` dans 
FigTree pour visuler votre arbre d'espèce estimé par \*BEAST.

