# Tenir un journal de bord en utilisant Markdown

## Introduction

Tenir un journal de bord est essentiel pour documenter les commandes exécutées, les résultats 
obtenus, et les observations importantes pendant vos travaux sur le serveur. Markdown est un 
langage de balisage léger qui vous permet de formater du texte facilement en utilisant des fichiers texte. Ce tutoriel vous guidera à travers les bases du Markdown et vous montrera comment l'utiliser pour tenir un journal de bord efficace.

## Qu'est-ce que Markdown?

Markdown est un langage de balisage utilisé pour formater du texte brut en documents structurés. 
Il est largement utilisé pour la documentation et est pris en charge par de nombreux éditeurs de 
texte et plateformes de gestion de versions.

### Syntaxe de Base du Markdown

- **Titres** : Utilisez des hashtags pour créer des titres.

  ```markdown
  # Titre de niveau 1
  ## Titre de niveau 2
  ### Titre de niveau 3
  ```

- **Listes** : Utilisez des tirets ou des astérisques pour les listes non ordonnées et des nombres 
pour les listes ordonnées.

  ```markdown
  - Élément de liste
  - Un autre élément

  1. Premier élément
  2. Deuxième élément
  ```

- **Texte en Gras et Italique** :

  ```markdown
  **Texte en gras**
  *Texte en italique*
  ```

- **Liens et Images** :

  ```markdown
  [Texte du lien](http://lien.com)
  ![Texte alternatif de l'image](http://url-de-l-image.com)
  ```

- **Code** : Encapsulez le code avec des accents graves pour le code en ligne ou triple accents 
graves pour des blocs de code.

  ````markdown
  
  ```bash
  ## Bloc de code bash
  Commandes bash...
  ```
  
  ```R
  ## Bloc de code R
  Commandes R...
  ```
  
  ```python
  ## Bloc de code python
  Commandes python...
  ```
  
  ````

## Créer un Journal de Bord en Markdown

### Étape 1: Créer un Fichier Markdown

1. Connectez-vous à votre serveur via SSH.
2. Naviguez vers le répertoire où vous souhaitez créer votre journal de bord.
3. Utilisez un éditeur de texte en ligne de commande, comme `nano` ou `vim`, pour créer un fichier 
Markdown.

   ```bash
   nano journal_de_bord.md
   ```

### Étape 2: Documenter les Commandes et Résultats

Commencez par ajouter des titres pour chaque jour ou session de travail et incluez les commandes 
que vous avez exécutées ainsi que les résultats obtenus.

#### Exemple de Journal de Bord

````markdown
# Journal de Bord

## 2024-09-14

### Commandes Exécutées

- **Connexion au Serveur**
  ```bash
  ssh utilisateur@aphidzen.irbv.umontreal.ca
  ```

- **Vérification des Ressources**
  ```bash
  htop
  ```

- **Soumission d'un Job SLURM**
  ```bash
  sbatch mon_script.slurm
  ```

### Observations

- Le job SLURM a démarré sans erreurs.
- La mémoire allouée semble suffisante pour les tâches en cours.

### Problèmes Rencontrés

- Le fichier de sortie ne contient pas les résultats attendus. Vérifier les erreurs dans le fichier `mon_script.slurm.err`.

## 2024-09-15

### Commandes Exécutées

- **Création d'un Job Array**
  ```bash
  sbatch --array=1-5 mon_array_script.slurm
  ```

### Observations

- Les tâches de l'array se répartissent correctement.
- Les fichiers de sortie sont générés comme prévu.

### Problèmes Rencontrés

- Certaines tâches ont échoué en raison d'une erreur de chemin de fichier.

````

### Étape 3: Ajouter des Notes et Réflexions

Ajoutez des notes sur les ajustements apportés, les observations importantes et les réflexions 
personnelles sur les résultats.

### Étape 4: Sauvegarder et Mettre à Jour

1. Sauvegardez vos modifications dans l'éditeur de texte.
2. Continuez à mettre à jour le fichier au fur et à mesure de vos travaux.
3. **Important** : Assurez-vous de sauvegarder les fichiers de votre journal de bord sur votre 
propre ordinateur, pas sur le serveur. Cela garantit que vous ne perdez pas vos données en cas de 
problème avec le serveur.

## Recommandations d'éditeurs de Texte

Pour une meilleure expérience de rédaction et de gestion de vos fichiers Markdown, vous pouvez 
utiliser des éditeurs de texte puissants tels que **[Notepad++](https://notepad-plus-plus.org/)**.

## Exercices

1. **Créer un journal de bord** : Créez un fichier Markdown pour un jour de travail. Incluez au 
moins trois commandes exécutées, leurs résultats, et vos observations.

2. **Ajouter des notes** : Ajoutez des sections pour les problèmes rencontrés et les ajustements 
apportés.

3. **Documenter un job SLURM** : Soumettez un job SLURM et documentez le processus en utilisant 
Markdown. Assurez-vous d'inclure les commandes, les résultats, et toute erreur éventuelle.

4. **Utiliser des images et liens** : Si vous avez des graphiques ou des résultats pertinents sous 
forme d'images, ajoutez-les à votre fichier Markdown. Incluez également des liens vers des ressources ou des documentations utiles.

Ce tutoriel vous aide à créer un journal de bord organisé et informatif en utilisant Markdown. 
Documenter vos travaux de manière structurée vous permettra de mieux suivre vos progrès et de 
résoudre plus efficacement les problèmes.

---