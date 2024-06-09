
# Student List Application

## Introduction
Ce projet consiste à dockeriser une application simple qui liste les étudiants avec un serveur web (PHP) et une API (Flask). L'objectif est de démontrer comment Docker peut améliorer l'agilité, la scalabilité et l'automatisation des déploiements.

## Infrastructure
### Contexte Actuel
L'application actuelle fonctionne sur un seul serveur sans scalabilité ni haute disponibilité. Les déploiements de nouvelles versions sont souvent problématiques.

### Objectifs avec Docker
- Rendre l'infrastructure scalable et facilement déployable.
- Maximiser l'automatisation du déploiement.
- Utiliser CentOS 7.6 comme système d'exploitation.
- Maintenir la sécurité (ne pas désactiver les mécanismes de sécurité sans justification).

## Application
L'application "student_list" est composée de deux modules :
1. **API REST** (Flask) : Authentification de base requise, renvoie la liste des étudiants en format JSON.
2. **Application Web** (HTML + PHP) : Permet aux utilisateurs d'obtenir une liste des étudiants.

## Dockerisation de l'API
### Dockerfile
Le Dockerfile suivant est utilisé pour créer l'image Docker de l'API :
```Dockerfile
# image de base
FROM python:3.8-buster

# Maintainer information
LABEL maintainer="florian <mtopiflorian@gmail.com>"

# Copy des fichiers et dossiers à la racine
ADD . /

# Installations des dépendances
RUN apt-get update -y && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
RUN pip3 install -r /requirements.txt

# Déclarer un point de montage ssspour le dossier data
VOLUME /data

# Exposé le port
EXPOSE 5000

# CMD
CMD [ "python3", "./student_age.py" ]
```

### Étapes pour Construire et Tester l'Image de l'API
1. **Construire l'image** : Cette étape crée une image Docker à partir du Dockerfile.
   ```bash
   docker build -t student_api ./simple_api
   ```
2. **Lancer le conteneur** : Cette commande démarre un conteneur basé sur l'image construite, tout en montant le fichier `student_age.json` comme volume.
   ```bash
   docker run -d -v ./simple_api/student_age.json:/data/student_age.json student_list```
   ```
  
3. **Tester l'API** : Utilisez `curl` pour envoyer une requête GET à l'API et vérifier qu'elle répond correctement.
   ```bash
   curl -u toto:python -X GET http://<host IP>:5000/pozos/api/v1.0/get_student_ages
   ```
   ![resultat.png](images%2Fresultat.png)


## Déploiement avec Docker Compose
Le fichier `docker-compose.yml` permet de déployer les deux services de l'application de manière orchestrée :
### docker-compose.yml
```yaml
version: '3.8'

services:
  website:
    container_name: website
    image: php:apache
    ports:
      - "80:80"
    volumes:
      - /website:/var/www/html
    environment:
      USERNAME: toto
      PASSWORD: python
    networks:
      - student_network

  api:
    container_name: API
    build:
      context: ./simple_api
      dockerfile: Dockerfile
    volumes:
      - ./simple_api/student_age.json:/data/student_age.json
    depends_on:
      - website
    networks:
      - student_network

networks:
  student_network:

```

### Étapes pour Déployer l'Application avec Docker Compose
1. **Créer le fichier `docker-compose.yml`** : Définir les services API et Web, leurs volumes et ports.
   [docker-compose.yml](docker-compose.yml)

2. **Lancer l'application** : Utilisez Docker Compose pour démarrer les services définis.
   ```bash
   docker-compose up -d
   ```
  
3. **Vérifier le déploiement** : verifier que les containers ont bien démarré.
   ![test.png](images%2Ftest.png)

## Docker Registry
### Déploiement d'un Registre Privé Docker
1. **Déployer un registre privé** : Lancez les conteneurs de registre Docker pour stocker vos images en privé.
   ```bash
    docker-compose -f docker-compose-registry.yml up -d
   ```
   J'ai utilisé registry:2.8.2 image pour le registre, et joxit/docker-registry-ui:static pour son interface graphique et j'ai ajouté un fichier de config pour personaliser ma configuration.
   ```bash
    version: '3.8'
    services:
        registry-srv:
        image: registry:2.8.2
        restart: always
        ports:
        - 5010:5000
        volumes:
          - ./registry/storage:/var/lib/registry
          - ./config.yml:/etc/docker/registry/config.yml:ro
          - ./registry/htpasswd:/etc/docker/registry/htpasswd
          container_name: registry-srv
          environment:
          - REGISTRY_STORAGE_DELETE_ENABLED=true
          networks:
          - registry
        
        registry-ui:
        image: joxit/docker-registry-ui:1.5-static
        restart: always
        ports:
        - 5001:80
        environment:
          - REGISTRY_URL=http://registry-srv:5000
          - DELETE_IMAGES=true
          - SINGLE_REGISTRY=true
          - REGISTRY_TITLE=demo
          - SHOW_CONTENT_DIGEST=true
          - SHOW_CATALOG_NB_TAGS=true
          - CATALOG_MIN_BRANCHES=1
          - CATALOG_MAX_BRANCHES=1
          - TAGLIST_PAGE_SIZE=10
          - REGISTRY_SECURED=true
          - CATALOG_ELEMENTS_LIMIT=10
          container_name: registry-ui
          networks:
          - registry
        
        networks:
        registry:
        driver: bridge

   ``` 

2. **Pousser l'image dans le registre** : Taguez l'image et poussez-la dans le registre privé.
    Pousser une image sur le registre et tester l'interface utilisateur
   Vous devez le renommer avant:

  ```bash
        - docker login localhost:5010
        - docker image tag student_list-api localhost:5010/student_list-api
        - images docker
        - docker image push localhost:5010/student_list-api 
 
  ```
vous devriez avoir un resultat similaire à celui-ci
![Capture.png](images%2FCapture.png)
