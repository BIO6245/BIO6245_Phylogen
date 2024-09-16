# Connexion et utilisation d'un serveur de calcul via ssh

Ce tutoriel vous guidera à travers la connexion au serveur de calcul utilisé pour les travaux 
pratiques (T.P.) du cours **BIO6245 Analyses phylogénétiques**. Nous verrons comment se connecter 
via SSH, naviguer sur le serveur, transférer des fichiers entre votre ordinateur personnel et le 
serveur à l’aide de `rsync`, et réaliser quelques exercices pour mieux comprendre son fonctionnement.


## 1. Qu'est-ce que ssh ?

**SSH (Secure Shell)** est un protocole permettant d'établir une connexion sécurisée à distance 
entre un ordinateur local et un serveur. Cela vous permet d'interagir avec le serveur via une 
ligne de commande pour effectuer des tâches, telles que l'exécution d'analyses ou la gestion de 
fichiers.

---

## 2. Connexion au serveur de calcul

Le serveur de calcul du cours est accessible à l'adresse suivante : **aphidzen.irbv.umontreal.ca**. Pour vous y connecter, vous aurez besoin de l'outil SSH, disponible sur la plupart des systèmes d'exploitation.

### Connexion depuis un système Unix (Mac ou Linux)

Les systèmes Mac et Linux disposent déjà d'une ligne de commande Unix, donc vous pouvez utiliser 
directement `ssh`:  
```bash
ssh UTILISATEUR@aphidzen.irbv.umontreal.ca
```

(Remplacez "UTILISATEUR" par votre nom d'utilisateur.)

### Connexion depuis un système Windows

Windows propose également un environnement Unix via **WSL (Windows Subsystem for Linux)**. Une 
fois WSL installé, ouvrez une session de terminal Unix et tapez la commande suivante:  
```bash
ssh UTILISATEUR@aphidzen.irbv.umontreal.ca
```

---

## 3. Structure du serveur de calcul

Le serveur est organisé en plusieurs partitions, chacune avec une fonction spécifique:  
- `/home` : Espace personnel de 3 To pour chaque utilisateur. Vous y avez accès via `/home/$USER`, 
et vos fichiers ne sont pas partagés avec les autres utilisateurs.  
- `/data` : Espace partagé de 5 To pour entreposer des données. Tous les utilisateurs peuvent y 
accéder.  
- `/scratch` : Espace temporaire de 9,2 To pour les données et calculs. Les fichiers ici sont 
régulièrement supprimés, donc pensez à sauvegarder vos résultats ailleurs.  

---

## 4. Transfert de fichiers entre votre ordinateur et le serveur

### Utilisation de `rsync`

`rsync` est un outil efficace pour synchroniser des fichiers entre votre ordinateur local et un 
serveur distant via SSH. Il est particulièrement utile lorsque vous avez besoin de transférer de 
gros fichiers ou de nombreux fichiers en même temps.

#### Télécharger des fichiers depuis le serveur

Si vous souhaitez télécharger un fichier ou un dossier depuis le serveur vers votre ordinateur, 
utilisez la commande suivante:  
```bash
rsync -avz UTILISATEUR@aphidzen.irbv.umontreal.ca:/chemin/vers/dossier_serveur/fichier /chemin/vers/dossier_local/
```

#### Téléverser des fichiers vers le serveur

Pour téléverser des fichiers ou dossiers depuis votre ordinateur local vers le serveur, utilisez 
cette commande:  
```bash
rsync -avz /chemin/vers/dossier_local/fichier UTILISATEUR@aphidzen.irbv.umontreal.ca:/chemin/vers/dossier_serveur/
```

---

## 5. Exercices pratiques sur le serveur

### Exercice 1 : Explorer les différentes partitions

1. **Accédez à votre espace personnel** dans `/home`:  
   ```bash
   cd /home/$USER
   ```

   Listez les fichiers présents avec:  
   ```bash
   ls
   ```

2. **Visitez la partition partagée `/data`**:  
   ```bash
   cd /data
   ls
   ```

3. **Accédez à la partition temporaire `/scratch`**:  
   ```bash
   cd /scratch
   ls
   ```

### Exercice 2 : Vérifier l’espace disque disponible

Utilisez la commande suivante pour afficher l’utilisation de l’espace disque:  
```bash
df -h
```

### Exercice 3 : Surveiller les processus avec `htop` et `btop`

1. **Lancer `htop`** pour observer l’utilisation du CPU, de la RAM, et des processus:  
   ```bash
   htop
   ```
   (Appuyez sur `F10` pour quitter.)

2. **Lancer `btop`** si disponible:  
   ```bash
   btop
   ```

   (Appuyez sur `q` pour quitter.)

### Exercice 4 : Créer et éditer un fichier

1. **Allez dans votre répertoire personnel**:  
   ```bash
   cd /home/$USER
   ```

2. **Créez un fichier texte** nommé `testfile.txt`:  
   ```bash
   touch testfile.txt
   ```

3. **Éditez ce fichier** avec `nano`:  
   ```bash
   nano testfile.txt
   ```

   Ajoutez du texte, puis sauvegardez et quittez (`Ctrl+X`, `Y`, `Entrée`).

4. **Vérifiez que le fichier a bien été créé**:  
   ```bash
   ls
   ```

### Exercice 5 : Observer l’utilisation de la mémoire

Utilisez `free -h` pour voir la mémoire utilisée et disponible ainsi que la mémoire **swap**:  
```bash
free -h
```

### Exercice 6 : Téléverser et télécharger des fichiers entre votre ordinateur et le serveur


1. **Téléchargez un fichier** depuis le serveur vers votre ordinateur local.  
   Pendant que vous êtes connecté au serveur, créer un fichier:  
   ```bash
   cd ~
   touch fichier_test
   exit
   ```
   Maitenant, à partir de votre ordinateur personnel, téléchargez ce fichier:  
   ```bash
   CLUSTER_USERNAME=Votre_nom_utilisateur_sur_le_serveur
   rsync -avz $CLUSTER_USERNAME@aphidzen.irbv.umontreal.ca:/home/$CLUSTER_USERNAME/fichier_test .
   ```

2. **Téléversez un fichier** vers le serveur. Essayez de le faire par vous-même. Demandez de l'aide 
au professeur au besoin.
 

---

## 6. Conclusion

Grâce à ces exercices, vous avez appris à explorer les partitions du serveur, surveiller les 
processus en cours, et manipuler des fichiers. Cela vous aidera à mieux utiliser le serveur de 
calcul pour vos analyses dans le cadre du cours **BIO6245 Analyses phylogénétiques**.

Si vous rencontrez des difficultés, n'hésitez pas à consulter la documentation ou à demander de 
l'aide à votre professeur.

