# Télécharger des données publiques Sanger

## Trouver et télécharger des données sur Genbank

La base de donnéesGenbank(http://www.ncbi.nlm.nih.gov/) est le plus important dépôt de séquences assemblées, 
la majorité provenant du séquençage Sanger, qui est une méthode plus coûteuse et lente, mais donnant les 
résultats les plus précis encore à ce jour. Vous allez faire une recherche pour des séquences de nucléotide 
dans la base de données GENBANK. Notez qu’il y a plusieurs bases de données et sortes de recherches listées 
dans le menu déroulant à gauche. Vous pouvez faire des recherches par groupe taxonomique, par gène d’intérêt, 
ou par numéro d’accession de GenBank.  

Vous allez monter une matrice de séquences du gène plastidique matK pour quelques espèces du genre de 
fougères *Dryopteris*, ainsi qu'un hors-groupe du genre *Polystichum*.  

1. Dans le menu déroulant à côté de "Search", sélectionnez "Nucleotide".  
2. Tapez dans la boite de recherche "Dryopteris Polystichum matK".  
3. Cochez une seule séquence pour chacune des espèces suivantes:  
  - *Dryopteris campyloptera*;  
  - *Dryopteris celsa*;  
  - *Dryopteris expansa*;  
  - *Dryopteris goldieana*;  
  - *Dryopteris intermedia*;  
  - *Dryopteris ludoviciana*;  
  - une séquence de n'importe quelle espèce de *Polystichum*.
4. Cliquez sur "Send to" en haut à droite, puis sélectionnez "Complete Record" -> "File" -> Format 
"FASTA", puis "Create File".  

## Examiner les séquences et ajuster les noms

Dans votre dossier de Téléchargements, vous devriez avoir un fichier nommé "sequence.fasta" qui contient 
l'ensemble des séquences que vous venez de télécharger de Genbank. Ouvrez ce fichier à l'aide d'un éditeur 
de texte, comme par exemple Notepad, Notepad++, TextWrangler, SublimeText, etc. **N'ouvrez pas le fichier **
**dans Word**, car cela risque de créer des problèmes de formattage à la sauvegarde.

Examinez le format du fichier. Vous pouvez télécharger les mêmes séquences en format Genbank et l'ouvrir de 
la même façon, pour voir les différences entre ces différents formats.

Pour faciliter l'utilisation du fichier par d'autres programmes, renommez chaque ligne en remplacant les 
espaces par des "_", et en ne conservant que le nom de l'espèce suivi du numéro Genbank et en éliminant. 
Les points et tous les autres caractères spéciaux. Par exemple, la séquence nommée à l'origine 
">JQ941638.1 Dryopteris campyloptera voucher EBS 22 (WIS) maturase K-like (matK) gene, partial sequence" 
devrait être renommée ">Dryopteris_campyloptera_JQ941638".
