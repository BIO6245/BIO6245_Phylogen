# BIO6245 Analyses phylogénétiques

Ce dépôt Github contient les tutoriels des travaux pratiques (T.P.) du cours BIO6245 Analyses phylogénétiques 
offert par le professeur [Étienne Léveillé-Bourret](https://irbv.umontreal.ca/le-personnel/etienne-leveille/) 
du département de sciences biologiques de l'Université de Montréal.

## Connexion et utilisation du serveur de calcul du cours
Le serveur de calcul utilisé pour les T.P. est joignable à cette adresse: `aphidzen.irbv.umontreal.ca`. Pour se connecter, il faut 
utiliser l'outil ssh dans une ligne de commande Unix (ligne de commande native sur Macintosh et Linux). 
Les versions récentes de Windows offrent aussi l'option d'atteindre une ligne de commande Unix via le 
[service WSL](https://learn.microsoft.com/windows/wsl/tutorials/linux). Une fois en ligne de commande 
Unix, tapez ce code (remplacez "UTILISATEUR" par votre propre nom d'utilisateur):  
```bash
ssh UTILISATEUR@aphidzen.irbv.umontreal.ca

```

Le serveur de calcul est organisé en trois partitions, qui ont chacunes des usages différents:
- **`/home`** est une partition de **3Tb** qui doit servir à installer des programmes ou de petits 
fichiers. Chaque étudiant a son propre espace sur cette partition (`/home/$USER`), qui n'est pas partagé 
avec les autres étudiants.
- **`/data`** est une partition de **5Tb** qui doit servir à l'entreposage de données. Cette partition 
est partagée entre tous les utilisateurs du serveur (professeurs et étudiants). 
- **`/scratch`** est une partition de **9.2Tb** qui doit servir uniquement à l'entreposage temporaire 
de données et au calcul. Les données qui sont entreposées dans `/scratch` seront supprimées 
périodiquement. Les utilisateurs doivent donc transférer les résultats essentiels de leurs analyses 
vers `/data` ou vers `/home` pour l'entreposage à plus long terme.

## Organisation du Github du cours

Les travaux sont organisés par thème dans des dossiers:
- HybSeq: assemblage et analyse phylogénétique des données HybSeq
- RADseq: assemblage et analyse phylogénétique des données RADseq
- Reseq: assemblage et analyse phylogénétique des données de reséquençage de génome complet
- Sanger: assemblage et analyse phylogénétique de données issues de séquençage Sanger

À l'intérieur de chaque dossier, un tutoriel vous guidera à travers les étapes d'analyse sur le serveur.

## T.P. 1: Télécharger, assembler et aligner des séquences

Pour ce T.P., faites les tutoriels suivants, dans cet en ordre:
1. [HybSeq/01--Telecharger_donnees.md](HybSeq/01--Telecharger_donnees.md)
2. HybSeq/02--ControleQualite.md
3. HybSeq/03--Assembler_avec_HybPiper.md
4. HybSeq/04--Alignement.md

