# Projet Kubernetes - Déploiement des application de IC Group
 
## Objectif

Ce projet a pour objectifs de  conteneuriser, configurer et déployer les applications du IC Group dans un cluster Kubernetes tout en assurant la persistance des données des différentes ressources.  
- Odoo 13.0 : ERP open source  
- pgAdmin 4 : Interface Web d'administration pour PostgreSQL  
- Application web développer avec Flask permettant l'accès aux deux services ci-dessus.  
  
Source :  [Github repository](https://github.com/OlivierKouokam/mini-projet-5esgi)   

---
 
## Prérequis
 
- Cluster Kube
- Docker 
- Docker Hub
- Github 

 
---

# Mise en place

- 3 conteneur Kubernetes au sein du namespace icgroup : 
PostgreSQL
Odoo port 33077  
PgAdmin port 33078
  
## Copy du GIT
 
```bash
git clone https://github.com/qahtan7/kubeprojet-esgi.git
cd ic-webapp
```

# Mise en place de l'application avec Docker 

## Dockerfile

```bash
# Image de base
FROM python:3.6-alpine

# Définit le répertoire de travail
WORKDIR /opt

# Copie les fichiers de l’application dans l’image
COPY . .

# Installe Flask version 1.1.2
RUN pip install flask==1.1.2

# Expose le port 8080
EXPOSE 8080

# Définit les variables d’environnement
ENV ODOO_URL="https://www.odoo.com"
ENV PGADMIN_URL="https://www.pgadmin.org"

# Lance l’application
ENTRYPOINT ["python", "app.py"]
```

## Test 

Build 
```bash
docker build -t ic-webapp:1.0
```
![docker build](./images/)

Check Image
```bash
docker image ls
```
![docker build](./images/)

Launch App
```bash
docker run -d --name test-ic-webapp -p 8080:8080 -e ODOO_URL="https://www.odoo.com" -e PGADMIN_URL="https://www.pgmain.org" ic-webapp:1.0
```
Check
```bash
docker ps -a
```
![docker ps](./images/)

Test Web Interface

```bash
http://192.177.10.140:8080
```
![web interface (docker)](./images/)

ODOO
![odoo interface (docker)](./images/)

Creation d'un tag et push sur notre compte DokcerHub

```bash
docker login
docker tag ic-webapp:1.0 aslimani94470/ic-webapp:1.0
docker push aslimani94470/ic-webapp:1.0
```
![docker push](./images/)

Check DockerHub 
![dockerhub image](./images/)

Docker stop & remove 
```bash
docker stop <container id>
docker rm <container id>
```
![docker rm](./images/)

# Manifest Kube

Voici les fichiers manifests utilisés dans cette partie : 
![Files list](./images/)

## Deploiement WebAPP

Namespace et Deploiement WebAPP

```bash
kubectl create ns icgroup 
kubectl apply -f ic-webapp-deployment.yaml -n icgroup  
kubectl apply -f ic-webapp-service.yaml -n icgroup  
```
## Deploiement de la Base de données

```bash
kubectl apply -f postgresql.yaml -n icgroup  
```
Creation de l'utilisateur odoo dans la base de donnée 

```bash
kubectl exec -it <pod name> -n icgroup -- bash
psql -U postgres
CREATE USER odoo WITH PASSWORD odoo;  
ALTER USER odoo CREATEDB;
```
![New user](./images/)
![New permission](./images/)


## Deploiement de Odoo
```bash
kubectl apply -f odoo.yaml -n icgroup  
```
## Deploiement de pgAdmin
```bash
kubectl apply -f pgadmin.yaml -n icgroup  
```
## Check ALL

```bash
kubectl get all -n icgroup  
```
![Toutes les ressources](./images/) 
Manifest 

```bash
kubectl get cm -n icgroup  
```
![configmap](./images/) 
ConfigMap  

```bash
kubectl get cm -n icgroup  
```
![pvc](./images/) 
Volume

## Check GUI 

Check Odoo 

```bash
http://192.177.10.140:33077 
```
Odoo Web interface OK

Création de la base de donnée ODOO
![Odoo db creation](./images/) 

![Odoo interface](./images/) 
PGAdmin Web interface OK 

```bash
http://192.177.10.140:33078 
```
![pgAdmin login](./images/) 
PGAdmin Web interface OK

BDD Connection
![pgAdmin connected](./images/)

Connection avec l'utilisateur ODOO
![pgAdmin connection to odoo](./images/)  
![pgAdmin connected to odoo](./images/)  
OK

Odoo Web interface
```bash
http://192.177.10.140:30001  
```
Access Web interface port 30001 :
![Interface web (30001)](./images/)  

Login Odoo
![Odoo login](./images/)  


Join PgAdmin
![pgAdmin login](./images/)  

After Connect
![pgAdmin connected to odoo](./images/)  


