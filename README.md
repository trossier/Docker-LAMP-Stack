# LAMP Stack

Une stack Docker pour votre application PHP

Cette stack contient:
- server Apache 2
- serveur MySQL 5.7
- PHP 7.0
- SonarQube 7.1
- GIT
- COMPOSER 
- Gestion des logs apache et mysql

Structure:
- doc
    - readme.md
- docker-compose.yml
- engine
    - apache-php
        - Dockerfile
    - config
        - apache2
            - vhost
                - virtualhost.conf
        - mysql
            - my.cnf
    - logs
        - apache
            - access.log
            - error.log
            - other_vhosts_access.log
        - mysql
            - error.log
    - mysql
        - data
    - sonarqube
        - data
- www

Pré-requis:
- Docker
- Docker-compose 
- Créer un vhost dans engine/config/apache2/vhost
- Créer votre application dans le dossier www (si vous souhaitez changer le chemin du volume il faut modifier le docker-compose.yml)
- Créer le fichier sonar-project.properties à la racine du projet PHP - contenu du fichier section SonarQube (ci-dessous)
- optionnel :: Configurer le fichier my.cnf
- optionnel :: Congigurer le php.ini (en cours de développement)

Compatibilité:
Cette installation a été testée sur un Mac OS X El Capitan v10.11.2

Attention performance Mac OS X et Windows :
Docker ne fonctionne que sur le système d’exploitation Linux, il ne fonctionne pas sur macOS ni sur Windows nativement. 
Pour pouvoir s’en servir sur macOS, l’application Docker for Mac va installer pour vous un Linux minimaliste dans une machine virtuelle grâce à l’hyperviseur natif de macOS, disponible depuis OS X Yosemite (10.10), et HyperKit. 
Le principe est le même sur Windows avec Hyper-V.
Pour transférer les données entre votre host et la machine virtuelle, c’est osxfs qui en a la charge. 
Malheureusement, osxfs souffre encore de gros problèmes de performances et les développeurs en sont bien conscients et ont commencé à mettre en place des solutions.
La mise en place d’un cache est désormais disponible et elle présente une solution intéressante le temps d’une mise à jour d’osxfs qui permettra d’avoir des performances similaires à celle sur Linux. En effet, les performances d’une application conteneurisée sur Linux sont presque les mêmes que sur un host Linux.
Il existe trois options possibles pour mettre en place ce cache :
- delegated : les données dans le container sont prédominantes sur celle de l’host, il y a donc un délais entre ce qu’il se passe dans le container et ce qui est sauvegardé sur l’host. Permet d’augmenter les vitesses d’écriture.
- cached : les données sur l’host sont prédominantes sur celles dans le container, il y a donc un délais entre ce qu’il se passe sur l’host et ce qui est envoyé dans le container. Permet d’augmenter les vitesses de lecture.
- consistent : paramètre par défaut, il permet de garder une parfaite consistance de vos données entre l’host et les containers.
Dans votre docker-compose.yml, il suffit d’ajouter l’option :cached (par exemple) à la fin de la déclaration de votre volume.
Vous trouverez beacoup d'articles sur internet qui expliquent les problèmes de performances, la mise en cache réduit grandement cela ;-)

Temps de création de la stack: 30 minutes

Evolution:
Remplacer le couple Apache / PHP par Nginx / PHP-FPM ;-)

#Lancer la stack
- Modifier les variables d'environnements dans le fichier .env
- modifier votre host local avec 127.0.0.1  <nomdedomaine> (/etc/hosts pour mac OS X et Linux)
- docker-compose up -d // Pour lancer la création de toutes les images et les containers
- Go to http://<nomdedomaine>:83 pour acceder à l'application
- Go to http://<nomdedomaine>:8080 pour acceder à phpmyadmin
- Go to http://<nomdedomaine>:32768 pour acceder à SonarQube

#Boîte à outils
docker images // Liste toutes les images
docker ps // Liste tous les containers
docker rmi -f <image> // Force la suppression d'une image
docker rm -f <container> // Force la suppression d'un container
docker stop <container> // Stop un container
docker build ou docker-compose up --build // Reconstruire une image
docker-compose up // Lancer un ensemble de containers via un docker-compose.yml file
docker rm $(docker ps -a -q) // Supprimer tous les caontainers (idem image avec rmi docker rmi $(docker images -a -q))
docker stats $(docker inspect -f "" $(docker ps -q)) // Vérifier la consommation de CPU
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -f name=nginx -q) // Get nginx IP (changer nginx par le container ou service)
cat $DUMP | docker exec -i dockerapache_mysql_1 /usr/bin/mysql -uroot -psecret //Restaurer/Importer une base de donnée depuis un dump sql
docker logs --tail 100 -f dockerapache_httpd_1 // Afficher les logs Apache (nom du container) en temps réel ( Pour débugger )
docker rm -f $(docker ps -a -q) && docker rmi $(docker images -a -q) // A utiliser avec précaution

- Apache / PHP -
docker-compose exec apache-php bash //Acceder en bash remplacer php par le nom du service defini dans le docker-compose.yml ou juste docker mais remplacer le nom du service par le nom du container

- MySQL-
docker-compose exec mysql php /var/www/symfony/app/console cache:clear //Commandes Symfony
docker-compose exec db mysql -uroot -p"root" // Se connecter à MySQL
docker exec server_mysql_dev /usr/bin/mysqldump --databases -uroot -psecret $DATABASE > $DATABASE.sql //Créer un dump d'une base de données

- COMPOSER-
docker exec -w=/var/www/EgerieRM/ server_apache_php_dev composer install

- GIT-
docker exec -w=/var/www/EgerieRM/ server_apache_php_dev git branch
docker exec -w=/var/www/EgerieRM/ server_apache_php_dev git status
docker exec -w=/var/www/EgerieRM/ server_apache_php_dev git stash / pop / list / drop
etc .... (all git commands)

- SonarQube-
// Pour lancer le test du code de l'application (il faut avoir au préalable configuré le sonar-project.properties cf partie SonarQube)
docker exec -w=/var/www/ server_apache_php_dev /usr/sonar-scanner-3.2.0.1227-linux/bin/sonar-scanner

-- WARNING --
les comandes avec docker-compose fonctionne aussi avec juste docker mais on appelera plus le service mais le nom du container, cependant j'utilise docker plutôt que docker-compose car l'option -w n'existe pas encore avec la version de docker-compose que j'utilise.

#Volumes Docker
Créer un fichier en local (machine hôte) ou sur le container aura pour effet de le créer automatiquement de l'autre côté ;-)

Améliorer les performances sur MacOSx
: Dans preferences docker augmenter CPU et Memory
: cached // surtout le cache
: utiliser docker machine (non testé)

Ci-dessous des articles interessants:
https://medium.com/@TomKeur/how-get-better-disk-performance-in-docker-for-mac-2ba1244b5b70
http://markshust.com/2018/01/30/performance-tuning-docker-mac

Docker machine
docker-machine ip default
docker-machine ls
docker-machine rm my-docker-machine
utiliser kitematic pour l'ip et le port a utiliser (Pour Mac OS X)

#SonarQube

Default login / password => admin / admin

#Fichier sonar-project.properties
# SonarQube server
#sonar.host.url=sonarqube:32768 # non du service dans docker-compose.yml
sonar.host.url=http://sonarqube:9000

# must be unique in a given SonarQube instance
sonar.projectKey=nameprojectkey # créé dans l'application SonarQube lors de la création d'un projet 

# this is the name displayed in the SonarQube UI
sonar.projectName=projectname
sonar.projectVersion=1.0

# Comma-separated paths to directories with sources (required)
sonar.projectBaseDir=/var/www/<folderproject> # remplacer le folderproject par le nom de votre dossier à analyser, le chemin est celui sur le container et non en local

# Comma-separated paths to directories with sources (required)
sonar.sources=src

# Language (Only when it is a single language)
#sonar.language=php

# Encoding of the source code. Default is default system encoding
sonar.sourceEncoding=UTF-8

# If I want to exclude files or directories
#sonar.exclusions=**/Tests/**,**/*Extension.php
sonar.exclusions=src/Tests/**/*.* # on retire des fichiers ou dossiers de l'analyse (dossiers ou fichiers qui doivent être dans le dossier src car on a précédement configuré sonar.sources=src) 

