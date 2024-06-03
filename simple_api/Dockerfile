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
