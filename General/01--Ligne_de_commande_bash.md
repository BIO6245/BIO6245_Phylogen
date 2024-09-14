# Introduction à la ligne de commande bash

La ligne de commande est un puissant outil informatique permettant d'interagir directement avec un 
système d'exploitation via des instructions écrites, appelées commandes. Elle est souvent utilisée 
pour effectuer des tâches de gestion de fichiers, d'installations de logiciels, ou d'exécutions de 
scripts de manière plus rapide et flexible qu'une interface graphique.

**Savoir utiliser la ligne de commande est une compétence essentielle en bioinformatique et** 
**pour la recherche moderne en phylogénétique**, notamment car c'est généralement la seule façon 
d'intéragir avec les serveurs de calcul et plusieurs programmes importants.

## Qu'est-ce qu'une ligne de commande?

La ligne de commande, aussi appelée _shell_, est un programme qui interprète les commandes saisies 
par l'utilisateur. Chaque commande exécutée dans une ligne de commande indique au système 
d'exploitation de réaliser une action spécifique. Contrairement aux interfaces graphiques où les 
actions sont réalisées en cliquant, la ligne de commande permet une interaction directe et 
textuelle avec l'ordinateur. C'est comme si on discutait directement avec l'ordinateur.

### Shells populaires
- **Bash (Bourne Again Shell)** : le shell le plus couramment utilisé sous Linux et macOS.
- **PowerShell** : un shell plus récent développé par Microsoft, disponible sur Windows.
- **Zsh** : un shell interactif souvent utilisé sur macOS en remplacement de Bash.

---

## Installation et accès à la ligne de commande

### Sur Windows

Windows n'a pas de terminal Linux natif, mais il est possible d'accéder à une ligne de commande 
compatible avec Linux de plusieurs manières:  

1. **Windows Subsystem for Linux (WSL)**  
   WSL permet d'exécuter un environnement Linux directement sous Windows. Pour l'installer :
   - Activez WSL en exécutant la commande suivante dans PowerShell en tant qu'administrateur :
     ```powershell
     wsl --install
     ```
   - Suivez les instructions et choisissez la distribution Linux que vous souhaitez utiliser 
   (par exemple Ubuntu).

   Plus de détails : [Guide d'installation de WSL par Microsoft]
   (https://learn.microsoft.com/fr-fr/windows/wsl/install)

2. **Git Bash**  
   Git Bash est une interface qui permet d'exécuter des commandes Bash sous Windows. Toutefois, 
   son utilisation est déconseillée dans le cours, le WSL était une meilleure approche.

### Sur macOS

macOS est un système basé sur Unix, ce qui signifie qu'il inclut nativement un terminal compatible 
avec les commandes Linux.

1. **Terminal macOS**  
   - Pour accéder à la ligne de commande, ouvrez simplement l'application "Terminal" via Spotlight (`Cmd + Space` et tapez "Terminal").
   - Le terminal utilise **Zsh** ou **Bash** comme interpréteur de commandes par défaut.

2. **Installation de Homebrew (optionnel mais recommandé)**  
   Homebrew est un gestionnaire de paquets qui facilite l'installation de logiciels depuis la ligne de commande. Pour l'installer, exécutez cette commande dans le terminal :
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

---

## Installation de iTerm2 sur macOS

Sur macOS, bien que le terminal natif soit suffisant pour la plupart des utilisateurs, 
**iTerm2** est une alternative plus puissante et flexible qui offre de nombreuses fonctionnalités 
supplémentaires, telles que la gestion de plusieurs onglets, la personnalisation avancée et la 
gestion des sessions.

### Étapes pour installer la plus récente version de iTerm2

1. **Télécharger iTerm2**
   - Rendez-vous sur le site officiel de iTerm2 : [iterm2.com](https://iterm2.com).
   - Cliquez sur le bouton **Download** pour télécharger la version la plus récente.
   
2. **Installation**
   - Une fois le fichier téléchargé (`iTerm2.zip`), double-cliquez dessus pour l'extraire.
   - Faites glisser l'icône `iTerm` dans votre dossier **Applications** pour l'installer.

3. **Lancer iTerm2**
   - Ouvrez **iTerm2** à partir de votre dossier **Applications** ou en utilisant 
   **Spotlight** (`Cmd + Space`, tapez "iTerm").

4. **Configurer iTerm2 (optionnel)**
   - iTerm2 offre de nombreuses options de personnalisation. Par exemple, vous pouvez configurer 
   l'apparence sous `Preferences > Profiles > Colors` et `Preferences > Profiles > Text`.
   - Si vous souhaitez activer l'intégration de zsh ou bash, iTerm2 prend en charge ces shells 
   de manière native, et vous pouvez les configurer selon vos préférences.

---


## Exercices pratiques : Premières commandes Bash

Voici quelques exercices simples pour vous familiariser avec la ligne de commande sous Linux ou 
macOS.

### 1. Afficher le répertoire courant

La commande suivante affiche le chemin complet du répertoire dans lequel vous vous trouvez 
actuellemen:  
```bash
pwd
```

### 2. Lister les fichiers et dossiers

Pour afficher les fichiers et dossiers dans le répertoire courant, utilisez:  
```bash
ls
```

Essayez avec l'option `-l` pour obtenir une liste détaillée:  
```bash
ls -l
```

### 3. Créer un nouveau dossier

Pour créer un nouveau répertoire (par exemple `nouveau_dossier`), utilisez:  
```bash
mkdir nouveau_dossier
```

### 4. Aller dans un autre répertoire

Pour naviguer vers un autre dossier (par exemple `nouveau_dossier` que vous venez de créer), 
tapez:  
```bash
cd nouveau_dossier
```

Pour revenir au répertoire parent, utilisez:  
```bash
cd ..
```

### 5. Créer un fichier et l'éditer

La commande `touch` permet de créer un nouveau fichier vide:  
```bash
touch fichier.txt
```

Pour éditer ce fichier, vous pouvez utiliser un éditeur de texte comme `nano`:  
```bash
nano fichier.txt
```

Tapez du texte, puis appuyez sur `Ctrl + X` pour quitter et sauvegarder.

### 6. Afficher le contenu d'un fichier

Si vous avez un fichier texte avec du contenu, vous pouvez le lire directement dans le terminal 
avec la commande `cat`:  
```bash
cat fichier.txt
```

### 7. Supprimer un fichier ou un dossier

Pour supprimer un fichier, utilisez `rm`:  
```bash
rm fichier.txt
```

Pour supprimer un dossier, utilisez l'option `-r` (récursif):  
```bash
rm -r nouveau_dossier
```

### 8. Variables

En programmation, une **variable** est un espace de stockage qui contient une valeur ou des 
informations. Les variables permettent aux programmes de manipuler des données de manière 
dynamique. Vous pouvez penser à une variable comme à une boîte avec une étiquette, dans laquelle 
vous stockez une valeur, que vous pouvez changer à tout moment.

- **Nom de la variable** : l’étiquette utilisée pour désigner cette boîte.
- **Valeur** : le contenu de cette boîte, c'est-à-dire l'information que la variable stocke.

Imaginez un gobelet avec votre nom dessus. Vous pouvez remplir ce gobelet avec de l'eau, du jus, 
ou le vider complètement. Le gobelet est la **variable**, et le contenu (ou l'absence de contenu) 
est la **valeur**.

En Bash, pour créer une variable, vous n’avez qu’à lui donner un nom et lui assigner une valeur. 
Voici un exemple simple:  
```bash
prenom="Étienne"
```

Ici, la variable s'appelle `prenom` et contient la valeur `"Étienne"`.

Pour utiliser cette variable, vous devez la précéder du symbole `$`:  
```bash
echo "Bonjour, $prenom !"
```

Ce code affichera : `Bonjour, Étienne !`.


### 9. Boucles

Les boucles sont des structures de contrôle qui permettent d'exécuter une série d'instructions 
plusieurs fois, facilitant l'automatisation de tâches répétitives. En Bash, il existe deux types 
de boucles courantes : les boucles `for` et `while`. Ce tutoriel vous apprendra à utiliser ces 
boucles directement dans le terminal et à les inclure dans des fichiers exécutables.

Les boucles en Bash sont un outil puissant pour automatiser des tâches répétitives. Vous pouvez 
les exécuter directement dans le terminal ou dans des fichiers Bash pour créer des scripts 
exécutables. Essayez de combiner différentes boucles pour mieux comprendre leur fonctionnement et 
leur potentiel dans vos projets.

### 10. Boucle `for`

La boucle `for` exécute une série d'instructions pour chaque élément d'une liste ou d'une plage 
définie.

Exemple de base:  
```bash
for i in 1 2 3 4 5
do
    echo "écrit à l'écran $i"
done
```

Ici, la boucle va définir la variable `i=1`, exécuter le contenu de la boucle, puis 
initialiser la variable `i=2`, ré-exécuter le contenu de la boucle avec cette nouvelle variable, 
et ainsi de suite jusqu'à `i=5`. Cela affichera donc "Iteration 1", "Iteration 2", etc., 
jusqu'à "Iteration 5". 

Vous pouvez aussi définir une plage de nombres avec `{}` :

```bash
for i in {1..5}
do
    echo "Nombre: $i"
done
```

Cela produit un résultat identique à celui de l'exemple précédent.

## 11. Boucle `while`

La boucle `while` exécute les instructions tant qu'une condition est vraie.

Exemple de base:  
```bash
i=1
while [ $i -le 5 ]
do
    echo "Compteur: $i"
    ((i++))
done
```

Dans cet exemple, tant que la variable `i` est inférieure ou égale à 5, la boucle continue 
d'exécuter les instructions. Le `-le` à l'intérieur des crochets suivant le `while` signifie 
"less or equal (<=)". On pourrait aussi utiliser `-lt` pour "less than (<)", `-ge` pour "greater 
or equal (>=)", `-eq` pour "equal (=)", `-ne` pour "not equal (!=)", etc.


## 12. Boucle à l'intérieur d'un fichier exécutable

Un script, c'est quand on écrit des commandes dans un fichier texte, qu'on peut ensuite exécuter 
comme un programme. Il est possible de mettre une boucle dans un script Bash et d'exécuter ce 
fichier comme un programme. 

Étapes pour créer et exécuter un fichier contenant une boucle:

1. **Créer un fichier Bash** :
   Utilisez un éditeur de texte comme `nano` pour créer un fichier Bash:  
   ```bash
   nano boucle.sh
   ```

2. **Écrire le script** :
   Voici un exemple de script contenant une boucle `for`. La première ligne, qui débute comme 
   "#!/bin/bash" sert à indiquer qu'il s'agit d'un script contenant des commandes bash:  
   ```bash
   #!/bin/bash
   for i in {1..5}
   do
       echo "Ligne $i"
   done
   ```

3. **Rendre le fichier exécutable** :
   Une fois le fichier sauvegardé, rendez-le exécutable avec la commande suivante:  
   ```bash
   chmod +x boucle.sh
   ```

4. **Exécuter le fichier** :
   Exécutez le script avec cette commande:  
   ```bash
   ./boucle.sh
   ```

Le script affichera "Ligne 1", "Ligne 2", jusqu'à "Ligne 5".

## 13. Boucle imbriquée

Vous pouvez aussi imbriquer des boucles, c'est-à-dire utiliser une boucle à l'intérieur d'une 
autre boucle.

Exemple de boucle imbriquée:  
```bash
for i in {1..3}
do
    for j in {1..2}
    do
        echo "i=$i, j=$j"
    done
done
```

Cela affichera toutes les combinaisons de `i` et `j`, par exemple `i=1, j=1`, `i=1, j=2`, et 
ainsi de suite.

---

## Ressources supplémentaires

- **Linux Command Line Basics** (Tutoriel vidéo) : [Learn Linux Command Line - Full Course]
(https://www.youtube.com/watch?v=ZtqBQ68cfJc)
- **The Linux Command Line** (Livre) : [The Linux Command Line - PDF gratuit]
(https://linuxcommand.org/tlcl.php)
- **Documentations Bash** : [Bash Reference Manual]
(https://www.gnu.org/software/bash/manual/)

En pratiquant ces commandes de base, vous vous familiariserez rapidement avec la ligne de 
commande et serez prêt à explorer des tâches plus complexes.

