FROM debian:9.5

# Ce fichier contient toutes les insctuctions à executer sur le conteneur "server_apache_php_dev" lancé par le service "apache-php" dans le docker-composer.yml

MAINTAINER Thomas ROSSIER <fr.thomas.rossier@gmail.com>

LABEL version="1.0"
LABEL description="LAMP - dev environment"

RUN apt-get -y update && apt-get install -y \
apache2 \
php7.0 \
libapache2-mod-php7.0 \
php7.0-mysql \
php7.0-curl \
php7.0-json \
php7.0-mcrypt \
php7.0-xml \
php7.0-mbstring \
php7.0-zip \
mysql-client 

# Installation des utilitaires
RUN apt-get install -y nano wget unzip curl git

RUN git config --global user.email "${EMAIL}" && git config --global user.name "${NAME}"

RUN cd /usr/ && wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.2.0.1227-linux.zip
RUN cd /usr/ && unzip /usr/sonar-scanner-cli-3.2.0.1227-linux.zip && rm /usr/sonar-scanner-cli-3.2.0.1227-linux.zip
RUN export PATH=${PATH}:/usr/sonar-scanner-3.2.0.1227-linux/bin/

# On veut une machine de dev qui affiche toutes les erreurs PHP
RUN sed -i -e 's/^error_reporting\s*=.*/error_reporting = E_ALL/' /etc/php/7.0/apache2/php.ini 
RUN sed -i -e 's/^display_errors\s*=.*/display_errors = On/' /etc/php/7.0/apache2/php.ini

# Installation de composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

# Set timezone
RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime
RUN "date"

# Suppression du vhost par défaut
RUN rm /etc/apache2/sites-enabled/000-default.conf

# Sans cette ligne on obtient l'erreur "AH00558: apache2: Could not reliably determine the server's fully qualified domain name"
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

RUN chown -R www-data:www-data /var/www
RUN find /var/www -type d -exec chmod 775 {} +
RUN find /var/www -type f -exec chmod 664 {} +

# Activation des modules SSL et REWRITE
#RUN a2enmod ssl
RUN a2enmod rewrite
RUN a2enmod headers
# On expose le port 80 et 443
#EXPOSE 80
#EXPOSE 443

# Commandes à exécuter au démarrage de l'instance de l'image
# Ici on démarrera Apache

CMD apachectl -D FOREGROUND

# start Apache2 on image start
#CMD ["/usr/sbin/apache2ctl","-DFOREGROUND"]
