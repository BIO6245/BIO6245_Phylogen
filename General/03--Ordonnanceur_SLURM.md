# Introduction aux ordonnanceurs et à SLURM

Après avoir appris à vous connecter au serveur de calcul et à naviguer dans
 son environnement, ce tutoriel vous introduira aux concepts des ordonnanceurs
 de tâches et vous expliquera comment utiliser **SLURM** (Simple Linux Utility
 for Resource Management), l'ordonnanceur utilisé sur notre serveur.

## 1. Qu'est-ce qu'un ordonnanceur de tâches?

Un **ordonnanceur de tâches** est un logiciel qui gère et planifie l'exécution
 des tâches sur une grappe (cluster) de calcul ou un serveur
 multi-utilisateurs. Il alloue les ressources disponibles (comme les
 noeuds de calcul, processeurs, la mémoire et le stockage) aux différents
 utilisateurs et tâches en fonction des priorités et des politiques définies.
 
Un **noeud de calcul** est l'équivalent d'un ordinateur. Dans les grappes de
 calcul, plusieurs noeuds sont connectés ensembles pour pouvoir effectuer des
 calculs plus complexes en combinant les ressources de calcul de tous les
 noeuds. En général, lorsque des utilisateurs effectuent des calculs sur une 
 grappe, ils utilisent les ressources d'un seul noeud par tâche, bien que
 certaines tâches peuvent être divisées sur plusieurs noeuds.
 
Un **noeud d'accueil** est aussi l'équivalent d'un ordinateur, mais
 ils ne servent généralement pas aux calculs. C'est plutôt le noeud sur
 lequel les usagers se connectent, installent des programmes et font des
 opérations de base sur leurs données. C'est à partir de ce noeud que les
 usagers envoient des demandent à l'ordonnanceur de tâches.
 
Un **processeur** (Central Processing Unit ou **CPU** en anglais) est
 l'unité de base de traitement des données dans un ordinateur. En général,
 les calculs complexes en phylogénétique peuvent être subdivisés en plusieurs
 sous-tâches associées à des processeurs différents pour accélérer le calcul.
 C'est plus ou moins l'équivalent à faire un travail d'équipe, ou chaque
 coéquipier (équivalent d'un processeur) est responsable d'une partie du
 travail.
 
La **mémoire** est l'espace de stockage temporaire utilisé lors des calculs.
 Contrairement aux processeurs, allouer plus de mémoire à une tâche de calcul 
 n'accélère généralement pas cette tâche. Plus de mémoire ne fait que permettre
 des calculs plus complexes, ou des calculs sur des plus grands jeux de
 données.

Les principales fonctions d'un ordonnanceur sont:  
- **Gestion des ressources** : Attribuer les ressources nécessaires aux tâches.  
- **Planification des tâches** : Déterminer l'ordre d'exécution des tâches en
 fonction des priorités et des demandes.  
- **Suivi des tâches** : Surveiller l'état des tâches et s'assurer qu'elles
 s'exécutent correctement.  
- **Optimisation de l'utilisation des ressources** : Maximiser l'utilisation
 efficace des ressources disponibles pour améliorer la performance globale.  

---

## 2. Introduction à SLURM

**SLURM (Simple Linux Utility for Resource Management)** est un ordonnanceur de
 tâches populaire utilisé dans les environnements de calcul haute performance
 (HPC). SLURM est conçu pour gérer les ressources, planifier les tâches, et
 surveiller leur exécution sur des grappes de serveurs.

### Commandes de base SLURM

Voici quelques commandes SLURM courantes que vous utiliserez pour soumettre et gérer vos tâches sur le serveur :

- **`squeue`** : Affiche les tâches en attente, en cours d'exécution, ou
 terminées.
- **`sbatch`** : Soumet un script de tâche à SLURM pour exécution.
- **`srun`** : Exécute une tâche ou un programme sur les ressources allouées.
- **`scancel`** : Annule une tâche en cours d'exécution ou en attente.
- **`sinfo`** : Affiche des informations sur les nœuds et les partitions de la
 grappe.

### Exemple de soumission de tâche avec SLURM

Pour exécuter une tâche sur le serveur, vous devez créer un fichier de script
 SLURM (par exemple, `job_script.sh`) avec les directives nécessaires. Voici un
 exemple de script SLURM pour soumettre une tâche simple :

```bash
#!/bin/bash
#SBATCH --job-name=mon_job         # Nom du travail
#SBATCH --output=mon_job_%j.log    # Fichier de sortie (%j est remplacé par le JobID)
#SBATCH --error=mon_job_%j.err     # Fichier d'erreur (%j est remplacé par le JobID)
#SBATCH --ntasks=1                 # Nombre de tâches
#SBATCH --cpus-per-task=4          # Nombre de processeurs par tâche
#SBATCH --mem=8G                   # Mémoire par CPU (ex. 8 Go par CPU)
#SBATCH --time=01:00:00            # Temps maximal d'exécution (hh:mm:ss)

# Commencez le travail
echo "Début du travail à $(date)"
# Exécutez votre programme ou script ici
./mon_programme
echo "Fin du travail à $(date)"
```

Soumettez ce script en utilisant la commande `sbatch` :

```bash
sbatch job_script.sh
```

### Modifier la mémoire par CPU

Pour spécifier la mémoire allouée par processeur, utilisez l'option
 `--mem-per-cpu`. Par exemple, pour allouer 8 Go de mémoire par processeur,
 utilisez `--mem-per-cpu=8G`. Assurez-vous d'ajuster la mémoire totale 
 (`--mem`) en conséquence si nécessaire.

Exemple de script SLURM avec spéficication de la mémoire:  
```bash
#!/bin/bash
#SBATCH --job-name=mon_job         # Nom du travail
#SBATCH --output=mon_job_%j.log    # Fichier de sortie (%j est remplacé par le JobID)
#SBATCH --error=mon_job_%j.err     # Fichier d'erreur (%j est remplacé par le JobID)
#SBATCH --ntasks=1                 # Nombre de tâches
#SBATCH --cpus-per-task=4          # Nombre de processeurs par tâche
#SBATCH --mem-per-cpu=8G           # Mémoire par CPU (8 Go par CPU)
#SBATCH --time=01:00:00            # Temps maximal d'exécution (hh:mm:ss)

# Commencez le travail
echo "Début du travail à $(date)"
# Exécutez votre programme ou script ici
./mon_programme
echo "Fin du travail à $(date)"
```

---

## 3. Gestion des arrays de tâches avec SLURM

Un **array de tâches** dans SLURM permet de soumettre plusieurs tâches
 similaires avec un seul script de soumission, chaque tâche ayant un
 identifiant unique. Les arrays sont utiles pour répéter la même tâche de
 calcul sur plusieurs jeux de données. Par exemple, il est possible de
 soumettre un array de tâches où chaque membre du array est responsable
 d'estimer la phylogénie d'un gène différent dans un jeu de données où
 plusieurs centaines de gènes ont été séquencés.

Voici un exemple de script SLURM qui soumet un array de tâches:  
```bash
#!/bin/bash
#SBATCH --job-name=mon_array_job   # Nom du travail
#SBATCH --output=mon_array_job_%A_%a.log  # Fichier de sortie (%A est le JobID, %a est l'ID du Job Array)
#SBATCH --error=mon_array_job_%A_%a.err   # Fichier d'erreur (%A est le JobID, %a est l'ID du Job Array)
#SBATCH --array=1-10               # Définir les indices de l'array (de 1 à 10)
#SBATCH --ntasks=1                 # Nombre de tâches par job array
#SBATCH --cpus-per-task=1          # Nombre de processeurs par tâche
#SBATCH --mem-per-cpu=4G           # Mémoire par CPU
#SBATCH --time=00:30:00            # Temps maximal d'exécution (hh:mm:ss)

# Commencez le travail
echo "Début du travail à $(date) pour le job ID ${SLURM_ARRAY_JOB_ID}, tâche ${SLURM_ARRAY_TASK_ID}"
# Exécutez votre programme ou script ici en utilisant ${SLURM_ARRAY_TASK_ID} pour les indices
./mon_programme ${SLURM_ARRAY_TASK_ID}
echo "Fin du travail à $(date) pour le job ID ${SLURM_ARRAY_JOB_ID}, tâche ${SLURM_ARRAY_TASK_ID}"
```

- **`--array=1-10`** : Soumet un array de tâches avec les indices allant de
 1 à 10.
- **`${SLURM_ARRAY_TASK_ID}`** : Variable d'environnement contenant
 l'identifiant de la tâche dans l'array.


Supposons que vous ayez plusieurs fichiers d'entrée (par exemple, `file1.txt`,
 `file2.txt`, etc.) et que vous souhaitiez exécuter une tâche sur chaque
 fichier. Vous pouvez utiliser la variable `${SLURM_ARRAY_TASK_ID}` pour
 accéder à chaque fichier en fonction de l'index de la tâche dans l'array.

Exemple de script SLURM pour traiter différents fichiers:  
```bash
#!/bin/bash
#SBATCH --job-name=mon_file_array_job
#SBATCH --output=mon_file_array_job_%A_%a.log
#SBATCH --error=mon_file_array_job_%A_%a.err
#SBATCH --array=1-5
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=4G
#SBATCH --time=00:30:00

# Liste des fichiers à traiter
FILES=("file1.txt" "file2.txt" "file3.txt" "file4.txt" "file5.txt")

# Obtenez le fichier correspondant à l'index de l'array
FILE=${FILES[$SLURM_ARRAY_TASK_ID-1]}

echo "Début du travail à $(date) pour le fichier ${FILE}"
# Exécutez votre programme ou script en utilisant le fichier
./mon_programme ${FILE}
echo "Fin du travail à $(date) pour le fichier ${FILE}"
```

- **`FILES`** : Array de noms de fichiers.
- **`${FILES[$SLURM_ARRAY_TASK_ID-1]}`** : Accède au fichier correspondant
 à l'index de la tâche.

Utiliser la variable `${SLURM_ARRAY_TASK_ID}` pour accéder à des fichiers ou
 effectuer des opérations spécifiques sur des indices de tâches est une méthode
 efficace pour gérer des ensembles de données dans des scripts SLURM. Cette
 approche permet de traiter des fichiers de manière séquentielle en fonction de
 l'identifiant de la tâche dans un array SLURM. 

### Autre approche pour utiliser la variable SLURM_ARRAY_TASK_ID

Dans cette approche, vous utilisez des commandes Unix pour sélectionner des fichiers ou des ressources en fonction de l'identifiant de la tâche
 (`${SLURM_ARRAY_TASK_ID}`). Voici comment cela fonctionne:  

1. **Liste des Fichiers** : La commande `ls $READS_PATH/*trim_R1.fastq.gz`
 génère une liste de fichiers correspondant au motif spécifié (par exemple,
 tous les fichiers `.fastq.gz` dans le répertoire `$READS_PATH`).

2. **Sélection de Fichiers** : `head -n $SLURM_ARRAY_TASK_ID` extrait les
 premiers fichiers de la liste jusqu'à l'index de la tâche
 (`$SLURM_ARRAY_TASK_ID`). 

3. **Extraction du Fichier Spécifique** : `tail -1` prend le dernier fichier
 de cette sous-liste, ce qui correspond au fichier associé à l'index de la
 tâche.

Supposons que vous avez une série de fichiers de séquence dans un répertoire,
 et vous souhaitez traiter chaque fichier dans un job SLURM array. Vous pouvez
 utiliser cette approche pour sélectionner un fichier spécifique à chaque tâche
 de l'array.

Voici un exemple de script SLURM pour traiter différents fichiers en utilisant
 cette approche:  
```bash
#!/bin/bash
#SBATCH --job-name=process_files_array
#SBATCH --output=process_files_%A_%a.log
#SBATCH --error=process_files_%A_%a.err
#SBATCH --array=1-10  # Assumons que vous avez 10 fichiers à traiter
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=4G
#SBATCH --time=01:00:00

# Définir le chemin des fichiers
READS_PATH="/path/to/your/files"

# Obtenez le nom du fichier pour cette tâche
READ1=$(ls $READS_PATH/*trim_R1.fastq.gz | head -n $SLURM_ARRAY_TASK_ID | tail -1)

# Affichez des informations de débogage
echo "Traitement du fichier : ${READ1}"

# Exécutez votre programme ou script en utilisant le fichier
./your_program ${READ1}

echo "Fin du traitement pour le fichier : ${READ1}"
```

Explications:  

- **`ls $READS_PATH/*trim_R1.fastq.gz`**: Liste tous les fichiers
 correspondant au motif spécifié dans le répertoire `$READS_PATH`.
- **`head -n $SLURM_ARRAY_TASK_ID`**: Extrait les premiers fichiers jusqu'à
 l'index de la tâche.
- **`tail -1`** : Prend le dernier fichier de cette liste, correspondant à
 l'identifiant de la tâche.

Étapes pour tester ce script:  
1. **Préparez vos fichiers**: Assurez-vous que le répertoire spécifié par
 `$READS_PATH` contient les fichiers à traiter.
2. **Créez le script SLURM**: Sauvegardez le code ci-dessus dans un fichier
 (par exemple, `process_files_array.sh`).
3. **Soumettez le script**: Utilisez la commande
 `sbatch process_files_array.sh` pour soumettre le job array.
4. **Vérifiez les résultats**: Consultez les fichiers de sortie pour chaque
 tâche afin de vérifier que les fichiers ont été correctement traités.

Cette approche est pratique pour automatiser le traitement de nombreux fichiers
 ou échantillons en parallèle, et elle est particulièrement utile pour les
 tâches de calcul de données en bioinformatique et dans d'autres domaines
 scientifiques.

---

## 4. Exercices Pratiques

### Exercice 1: modifier le temps d'exécution et la mémoire

1. Modifiez le script SLURM `job_script.sh` pour changer le temps d'exécution à
 2 heures et allouer 4 Go de mémoire par processeur.
2. Soumettez le script avec `sbatch job_script.sh`.
3. Vérifiez l'état de la tâche avec `squeue`.

### Exercice 2: soumettre un array de tâches (job array)

1. Créez un script SLURM pour soumettre un array de tâches avec des indices de
 1 à 5.
2. Soumettez le script avec `sbatch job_array_script.sh`.
3. Vérifiez l'état des tâches de l'array avec `s

Utiliser la variable `${SLURM_ARRAY_TASK_ID}` pour accéder à des fichiers ou
 effectuer des opérations spécifiques sur des indices de tâches est une méthode
 efficace pour gérer des ensembles de données dans des scripts SLURM. Cette
 approche permet de traiter des fichiers de manière séquentielle en fonction de
 l'identifiant de la tâche dans un array SLURM. 

