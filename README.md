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




