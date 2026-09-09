# Format des données Illumina en ligne de commande

Les séquenceurs Illumina produisent des fichiers en format **FASTQ**
 (`.fastq`), généralement compressés avez gzip (`.fastq.gz`). Ce tutoriel
 montre comment vous connecter au serveur de calcul, naviguer jusqu'aux données, et examiner leur structure avec des outils standards.

---

## Connexion au serveur aphidzen

```bash
ssh <votre_identifiant>@aphidzen.umontreal.ca
```

Une fois connecté, naviguez vers le répertoire contenant les données :

```bash
## naviguer vers le dossier des données HybSeq de Maurane Bourgouin
cd /data/sequenceData/hybseq/hybseq20250821_Maurane

## faire la liste de tous les fichiers, avez leur taille (s pour size)
## dans un format lisible par les humains (h pour human)
ls -sh
```

- **Question**: quelle est l'extension des fichiers? Quelle est leur taille?
 Y a-t-il des fichiers `R1` et `R2`? À quoi correspondent-ils?

---

## Structure du format FASTQ

Un fichier FASTQ code chaque lecture sur **4 lignes** :

```
@identifiant_de_lecture
SÉQUENCE_NUCLÉOTIDIQUE
+
SCORES_DE_QUALITÉ
```

La ligne 4 encode le score de qualité **Phred** de chaque base sous forme d'un
 caractère ASCII. Plus le caractère est élevé dans la table ASCII (ex. `I` >
 `5`), meilleure est la qualité.

Pour afficher les 2 premières lectures (8 premières lignes), il faut tout
 d'abord décompresser le fichier et l'afficher à l'écran avec `zcat`, puis
 passer ce résultat (pipe `|`) à la commande `head -n 8` pour n'afficher que
 les huit premières lignes:  
```bash
zcat Bidens_radiata_ELB311_S9_L003_R1_001.fastq.gz | head -n 8 
```

Si vous tentez d'afficher tout le fichier avec uniquement la commande `zcat`,
 l'écran déroulera très rapidement pendant très longtemps, car le fichier
 contient de très nombreuses lignes. Si, on contraire, vous affichez
 directement les 8 premières lignes sans décompresser en premier (en exécutant
 uniquement `head -n 8`), vous verrez que c'est illisible car c'est en binaire.
 
- **Question**: identifiez les 4 lignes de la première lecture. Quelle est la
  longueur de ces deux séquences en paires de bases?

- **Question**: où ce trouve le ou les code-barres uniques à cet échantillon,
  utilisés pour démultiplexer cet échantillon parmi tous les autres qui ont
  été séquencés dans la même ligne de séquençage?

- **Question**: est-ce possible de savoir qu'il s'agit de lectures Illumina
  pairées, en examinant uniquement ces 8 lignes?


## Compter le nombre de lectures

Chaque lecture occupe exactement 4 lignes. Le nombre de lectures est donc le
 nombre de lignes divisé par 4. La commande `wc -l` affiche le nombre de lignes
 d'un fichier en format texte (non binaire / non compressé):  
```bash
zcat Bidens_radiata_ELB311_S9_L003_R1_001.fastq.gz | wc -l
```

- **Question** : combien de lectures contient ce fichier? Avez-vous un nombre
 égal de lectures dans les fichiers R1 et R2?
