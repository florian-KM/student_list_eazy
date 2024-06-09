# SOLUTION

## le clone du projet est la premiere étape avant tout

1- Changer de répertoire et construire l'image du conteneur api :
- cd .student_list
- docker build ./simple_api/ -t student_list-api
- images docker

2- Création d'un réseau
- docker network create student_network
- docker network ls

3- lancez le conteneur de l'api avec ces paramètres

-- présentation du fichier Dockerfile
[Dockerfile](simple_api%2FDockerfile)

### - Buider l'image avec la commade
```bash 
4 - Pour faire le run lancez la commande 
docker run -d -v ./simple_api/student_age.json:/data/student_age.json -p 5000:5000 student_list```

5 - verifions que notre container fonctionne bien
```bash 
curl -u toto:python -X GET http://localhost:5000/pozos/api/v1.0/get_student_ages
```

### - resultat attendu
![resultat.png](images%2Fresultat.png)


# Deployment

#### pour créer notre docker compose nous pouvons directement le convertir grace au site composerize.com pour avoir notre infrastructure as code dans un fichier docker-compose.yml.

-- présentation du fichier Dockerfile
[docker-compose.yml](docker-compose.yml)

1- Lancer l'application (api + webapp) :
Comme nous avons déjà créé l'image de l'application, il suffit maintenant de lancer :

    - docker-compose up -d
    
    l'argument -d  est utilisé pour le mode détaché.

### - resultat attendu
![resultat_docker_compose.png](images%2Fresultat_docker_compose.png)


2- Création d'un regsitre privé
J'ai utilisé registry:2.8.2 image pour le registre, et joxit/docker-registry-ui:static pour son interface graphique et j'ai ajouté un fichier de config pour personaliser ma configuration.

     - docker-compose -f docker-compose-registry.yml up -d
     - ouvrez votre navigateur en utilisant le port 5010


3- Pousser une image sur le registre et tester l'interface utilisateur
Vous devez le renommer avant:

- docker login localhost:5010
- docker image tag student_list-api localhost:5010/student_list-api
- images docker
- docker image push localhost:5010/student_list-api

recharger le navigateur pour voir l'image poussée

### Resultat attendu

vous devriez avoir un resultat similaire à celui-ci
![Capture.png](images%2FCapture.png)








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
   ![Screenshot: Docker Compose Up](path_to_screenshot_docker_compose_up)

3. **Vérifier le déploiement** : Accédez à l'application web et cliquez sur le bouton "List Student" pour vérifier que la liste des étudiants apparaît.
   ![Screenshot: Application Web](path_to_screenshot_web_app)

## Docker Registry
### Déploiement d'un Registre Privé Docker
1. **Déployer un registre privé** : Lancez un conteneur de registre Docker pour stocker vos images en privé.
   ```bash
   docker run -d -p 5000:5000 --restart=always --name registry registry:2
   ```
   ![Screenshot: Docker Registry](path_to_screenshot_docker_registry)

2. **Pousser l'image dans le registre** : Taguez l'image et poussez-la dans le registre privé.
   ```bash
   docker tag student_api localhost:5000/student_api
   docker push localhost:5000/student_api
   ```
   ![Screenshot: Push Image](path_to_screenshot_push_image)

## Conclusion
Nous avons dockerisé avec succès l'application "student_list" et démontré comment Docker peut améliorer l'agilité, la scalabilité et l'automatisation des déploiements. L'application est maintenant plus facile à gérer et à déployer, répondant ainsi aux besoins de POZOS.

## Captures d'Écran
Ajoutez des captures d'écran pertinentes montrant les différentes étapes, y compris la construction de l'image, le test de l'API, le déploiement avec Docker Compose, et l'utilisation du registre privé Docker.
