# Mémo commandes — Labs S3, Kubernetes et DuckDB

## 1. Terminal / Linux

### Où suis-je ?
pwd

### Lister les fichiers
ls
ls -la
ls -lh

### Se déplacer
cd dossier/
cd ..
cd ~

### Créer un dossier
mkdir -p dossier/

### Créer un fichier
touch fichier.txt

### Afficher un fichier
cat fichier.txt

### Afficher les premières lignes
head fichier.txt

### Afficher les dernières lignes
tail fichier.txt

### Rechercher dans un fichier
grep "mot" fichier.txt

### Afficher un fichier avec les numéros de ligne
nl -ba fichier.txt

### Supprimer un fichier
rm fichier.txt

### Supprimer un fichier sans erreur s'il n'existe pas
rm -f fichier.txt

### Variables d'environnement
echo "$VARIABLE"

export VARIABLE="valeur"

### Voir les variables AWS sans afficher les secrets
env | grep '^AWS_' | grep -v 'SECRET\|TOKEN'

### Historique des commandes
history

### Quitter un programme
Ctrl+D
# ou, selon le programme :
.quit
exit


## 2. Git

### Vérifier le dépôt
git status

### Voir la branche actuelle
git branch

### Voir les commits récents
git log --oneline

### Voir les modifications
git diff

### Ajouter un fichier
git add fichier.txt

### Ajouter plusieurs fichiers
git add fichier1 fichier2

### Ajouter tous les changements
git add .

### Vérifier ce qui sera commité
git status

### Créer un commit
git commit -m "message du commit"

### Envoyer vers GitHub
git push

### Récupérer les changements
git pull

### Voir les dépôts distants
git remote -v

### Annuler une modification locale non commitée
git restore fichier.txt

### Retirer un fichier du staging
git restore --staged fichier.txt


## 3. Kubernetes

### Vérifier que kubectl fonctionne
kubectl version --client

### Voir le contexte Kubernetes
kubectl config current-context

### Voir les namespaces
kubectl get namespaces

### Voir les ressources du namespace courant
kubectl get all

### Voir les pods
kubectl get pods

### Voir les pods avec plus de détails
kubectl get pods -o wide

### Voir les jobs
kubectl get jobs

### Voir les services
kubectl get services

### Voir les ConfigMaps
kubectl get configmaps

### Voir les Secrets
kubectl get secrets

### Décrire une ressource
kubectl describe pod NOM_DU_POD
kubectl describe job NOM_DU_JOB

### Voir les logs d'un pod
kubectl logs NOM_DU_POD

### Suivre les logs en direct
kubectl logs -f NOM_DU_POD

### Voir les événements
kubectl get events --sort-by=.lastTimestamp

### Appliquer un fichier YAML
kubectl apply -f fichier.yaml

### Supprimer une ressource définie dans un YAML
kubectl delete -f fichier.yaml

### Supprimer un pod
kubectl delete pod NOM_DU_POD

### Supprimer un job
kubectl delete job NOM_DU_JOB

### Voir le YAML d'une ressource
kubectl get pod NOM_DU_POD -o yaml


## 4. S3 avec AWS CLI

### Vérifier l'identité AWS
aws sts get-caller-identity --profile default

### Voir la configuration AWS
aws configure list --profile default

### Lister les buckets
aws s3 ls --profile default

### Lister le contenu d'un bucket
aws s3 --profile default ls "s3://$LAB_BUCKET_NAME/"

### Lister récursivement
aws s3 --profile default ls "s3://$LAB_BUCKET_NAME/" --recursive

### Lister un dossier S3
aws s3 --profile default ls "s3://$LAB_BUCKET_NAME/bronze/"

### Envoyer un fichier
aws s3 --profile default cp fichier.csv \
  "s3://$LAB_BUCKET_NAME/bronze/fichier.csv"

### Télécharger un fichier
aws s3 --profile default cp \
  "s3://$LAB_BUCKET_NAME/bronze/fichier.csv" fichier.csv

### Supprimer un objet
aws s3 --profile default rm \
  "s3://$LAB_BUCKET_NAME/bronze/fichier.csv"

### Supprimer tout un dossier
aws s3 --profile default rm \
  "s3://$LAB_BUCKET_NAME/large/" --recursive


## 5. DuckDB

### Lancer DuckDB
duckdb

### Lancer une base DuckDB
duckdb analytics.duckdb

### Lancer DuckDB avec un fichier SQL d'initialisation
duckdb analytics.duckdb -init init.sql

### Exécuter une requête directement
duckdb -c "SELECT 1;"

### Quitter DuckDB
.quit

### Afficher les tables
.tables

### Décrire une table
DESCRIBE users;

### Compter les lignes
SELECT count(*) FROM users;

### Lire un CSV
SELECT *
FROM read_csv('fichier.csv')
LIMIT 10;

### Lire un CSV avec une URL S3
SELECT *
FROM read_csv(
    getvariable('bucket') || '/bronze/orders.csv',
    strict_mode = false
);

### Lire un Parquet
SELECT *
FROM read_parquet(
    getvariable('bucket') || '/analytics/orders.parquet'
);

### Créer une table depuis un CSV
CREATE OR REPLACE TABLE orders AS
FROM read_csv(
    getvariable('bucket') || '/bronze/orders.csv'
);

### Exporter une table en Parquet
COPY orders
TO (getvariable('bucket') || '/analytics/orders.parquet')
(FORMAT parquet);

### Activer le chronomètre
.timer on

### Voir les métadonnées Parquet
SELECT *
FROM parquet_metadata(
    getvariable('bucket') || '/analytics/orders.parquet'
);

### Voir les fichiers correspondant à un chemin
SELECT file
FROM glob(
    getvariable('bucket') || '/analytics/**'
);


## 6. uv / Python

### Vérifier uv
uv --version

### Installer les dépendances
uv sync

### Ajouter une dépendance
uv add duckdb

### Lancer un script/une commande du projet
uv run orders-report

### Lancer avec un argument
uv run orders-report -p cookie

### Lancer un module Python
uv run python fichier.py

### Vérifier Python
python --version


## 7. Commandes utiles pour les labs

### Voir le namespace Kubernetes
echo "$KUBERNETES_NAMESPACE"

### Définir le bucket du lab
export LAB_BUCKET_NAME="$KUBERNETES_NAMESPACE"

### Définir l'endpoint S3
export S3_ENDPOINT_URL="$(
  aws configure get endpoint_url --profile default
)"

### Vérifier les fichiers bronze
aws s3 --profile default ls \
  "s3://$LAB_BUCKET_NAME/bronze/"

### Vérifier les fichiers du lab
aws s3 --profile default ls \
  "s3://$LAB_BUCKET_NAME/" --recursive

### Vérifier Git avant de terminer
git status

### Voir le dernier commit
git log -1 --oneline


## 8. À NE JAMAIS COMMITER

Ne jamais mettre dans Git :

- mot de passe
- Access Key
- Secret Key
- Session Token
- credentials AWS
- fichiers `.env` contenant des secrets
- fichiers contenant des tokens
- données sensibles

Vérifier toujours `git status` avant `git add .`.


## 9. Routine de fin de lab

# 1. Vérifier les fichiers modifiés
git status

# 2. Vérifier les différences
git diff

# 3. Ajouter uniquement les fichiers nécessaires
git add fichier1 fichier2

# 4. Vérifier le staging
git status

# 5. Commit
git commit -m "message"

# 6. Push
git push

# 7. Vérifier
git status
git log -1 --oneline
