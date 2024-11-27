# BIO6245 Analyses phylogénétiques

Ce dépôt Github contient les tutoriels des travaux pratiques (T.P.) du cours BIO6245 Analyses 
phylogénétiques offert par le professeur 
[Étienne Léveillé-Bourret](https://irbv.umontreal.ca/le-personnel/etienne-leveille/) du département de 
sciences biologiques de l'Université de Montréal.



## Programmes à installer pour le cours

Il faut installer les programmes ci-dessous pour le cours:  

- [AliView](https://ormbunkar.se/aliview/) pour visualiser et réaligner des alignements de données 
génétiques.
- [MEGA](https://www.megasoftware.net/) pour visualiser et analyser des petits alignements de données 
génétiques.  
- [FigTree](https://github.com/rambaut/figtree/releases) pour visualiser les arbres phylogénétiques.  
- Un bon éditeur de texte. J'aime particulièrement [Notepad++](https://notepad-plus-plus.org/) sur 
Windows, 
[Sublime](https://www.sublimetext.com/) sur Mac et 
[gedit](https://gedit-technology.github.io/apps/gedit/) sur Linux.  
- Le 
[VPN de l'Université de Montréal](https://wiki.umontreal.ca/pages/viewpage.action?pageId=127184571) 
pour vous connecter au serveur de calcul à partir de la maison.  

## Connexion et utilisation du serveur de calcul du cours
Le serveur de calcul utilisé pour les T.P. est joignable à cette adresse: 
`aphidzen.irbv.umontreal.ca`. Pour se connecter, il faut utiliser l'outil ssh dans une ligne de 
commande Unix (ligne de commande native sur Macintosh et Linux). Les versions récentes de Windows 
offrent aussi l'option d'atteindre une ligne de commande Unix via le 
[service WSL](https://learn.microsoft.com/windows/wsl/tutorials/linux). Une fois en ligne de 
commande Unix, tapez ce code (**remplacez "UTILISATEUR" par votre propre nom d'utilisateur**):  
```bash
ssh UTILISATEUR@aphidzen.irbv.umontreal.ca

```

Le serveur de calcul est organisé en trois partitions, qui ont chacunes des usages différents:
- **`/home`** est une partition de **3Tb** qui doit servir à installer des programmes ou de petits 
fichiers. Chaque étudiant a son propre espace sur cette partition (`/home/$USER`), qui n'est pas 
partagé avec les autres étudiants.
- **`/data`** est une partition de **5Tb** qui doit servir à l'entreposage de données. Cette 
partition est partagée entre tous les utilisateurs du serveur (professeurs et étudiants). 
- **`/scratch`** est une partition de **9.2Tb** qui doit servir uniquement à l'entreposage 
temporaire de données et au calcul. Les données qui sont entreposées dans `/scratch` seront 
supprimées périodiquement. Les utilisateurs doivent donc transférer les résultats essentiels de 
leurs analyses vers `/data` ou vers `/home` pour l'entreposage à plus long terme.

## Informations importantes lorsqu'on utilise une ligne de commande

Pour n'importe quelle tâche à faire sur une ligne de commande (commandes sur le serveur, sur votre 
ordinateur, sur un programme en mode ligne de commande), il est **essentiel** de conserver une 
copie de toutes les commandes que vous avez exécutées dans un fichier texte. Cela permet de 
documenter les étapes de votre travail pour que le professeur ou d'autres étudiants puissent venir 
faire des corrections ou ajustements si nécessaires. Ça vous permet aussi de réutiliser certaines 
commandes lorsque nécessaire, et d'ainsi sauver un temps considérable.

Pour ce faire, chaque fois qu'il y a du code à exécuter en ligne de commande, plutôt que de 
l'écrire directement dans la ligne de commande, copiez-la plutôt dans un document texte. Ajouter 
aussi des commentaires (précédés du caractère dièse #) pour documenter correctement toutes les 
étapes. Une fois que vous avez une copie en format texte des commandes à exécuter, copiez-les 
dans la ligne de commande (copier-coller). Vous pouvez même ajouter des commentaires sur le 
résultat.

Voici un tutoriel expliquant une approche pour créer un **journal de bord** qui vous permettra 
de bien documenter votre travail bioinformatique dans ce cours: 
[General/04--Journal_de_bord.md](General/04--Journal_de_bord.md).


## Organisation du Github du cours

Les travaux sont organisés par thème dans des dossiers:
  - HybSeq: assemblage et analyse phylogénétique des données HybSeq
  - RADseq: assemblage et analyse phylogénétique des données RADseq
  - Reseq: assemblage et analyse phylogénétique des données de reséquençage de génome complet
  - Sanger: assemblage et analyse phylogénétique de données issues de séquençage Sanger

À l'intérieur de chaque dossier, un tutoriel vous guidera à travers les étapes d'analyse sur le 
serveur.

## T.P. 1: Utiliser une ligne de commande et un serveur de calcul

Pour ce T.P., faites les tutoriels suivants, dans cet en ordre:  
  - [General/01--Ligne_de_commande_bash.md](General/01--Ligne_de_commande_bash.md)  
  - [General/02--Serveur_de_calcul.md](General/02--Serveur_de_calcul.md)  
  - [General/03--Ordonnanceur_SLURM.md](General/03--Ordonnanceur_SLURM.md)  
  - [General/04--Journal_de_bord.md](General/04--Journal_de_bord.md)  

## T.P. 2: Télécharger, assembler et aligner des séquences

Pour ce T.P., faites les tutoriels suivants, dans cet en ordre:  

Avec les données Sanger:  
  - [Sanger/01--Telecharger_donnees.md](Sanger/01--Telecharger_donnees.md)  
  - [Sanger/02--Aligner_sequences.md](Sanger/02--Aligner_sequences.md)  

Avec les données HybSeq:  
  - [HybSeq/01--Telecharger_donnees.md](HybSeq/01--Telecharger_donnees.md)  
  - [HybSeq/02--ControleQualite.md](HybSeq/02--ControleQualite.md)  
  - [HybSeq/03--Assembler_avec_HybPiper.md](HybSeq/03--Assembler_avec_HybPiper.md)  
  - [HybSeq/04--Alignement.md](HybSeq/04--Alignement.md)  

## T.P. 3: Analyse de parcimonie

Pour ce T.P., faites les tutoriels suivants, dans cet en ordre:  

Avec les données Sanger:  
  - [Sanger/03--Parcimonie.md](Sanger/03--Parcimonie.md)  

Avec les données HybSeq:  
  - [HybSeq/05--Filtrage_alignement.md](HybSeq/05--Filtrage_alignement.md)  
  - [HybSeq/06--Parcimonie.md](HybSeq/06--Parcimonie.md)  
  
## T.P. 4: Analyse de parcimonie

Pour ce T.P., faites les tutoriels suivants, dans cet en ordre:  

Avec les données Sanger:  
  - [Sanger/04--Congruence_support.md](Sanger/04--Congruence_support.md)  

## T.P. 5: Distance et modèles de substitution

Pour ce T.P., faites les tutoriels suivants, dans cet ordre:

Avec les données Sanger:  
  - [Sanger/05--Selection_modele_distance.md](Sanger/05--Selection_modele_distance.md)

Avec les données HybSeq:  
  - [HybSeq/07--Selection_modele.md](HybSeq/07--Selection_modele.md)
  - [HybSeq/08--Distance.md](HybSeq/08--Distance.md)
  
## T.P. 6: Maximum de vraisemblance

Pour ce T.P., faites les tutoriels suivants, dans cet ordre:

Avec les données Sanger:  
  - [Sanger/06--Vraisemblance.md](Sanger/06--Vraisemblance.md)

Avec les données HybSeq:  
  - [HybSeq/09--Vraisemblance.md](HybSeq/09--Vraisemblance.md)

## T.P. 7: Analyse phylogénétique Bayesienne

Pour ce T.P., faites les tutoriels suivants, dans cet ordre:

Avec les données Sanger:  
  - [Sanger/07--Analyse_Bayesienne.md](Sanger/07--Analyse_Bayesienne.md)

Avec les données HybSeq:  
  - [HybSeq/10--Analyse_Bayesienne.md](HybSeq/10--Analyse_Bayesienne.md)

## T.P. 8: Estimation de l'arbre d'espèce

Pour ce T.P., faites le tutoriel suivant:

Avec les données Sanger:  
  - [Sanger/08--Arbre_despeces.md](Sanger/08--Arbre_despeces.md)

Avec les données HybSeq:    
  - [HybSeq/11--Arbre_despeces.md](HybSeq/11--Arbre_despeces.md)