# This content is part of the native-dockerfiles project and released under the MIT License (see /LICENSE file)
# (c) 2015 Thomas Mayer
# Get the latest version at https://github.com/thomaszbz/native-dockerfiles-typo3

FROM debian:latest
MAINTAINER Thomas Mayer "thomas.mayer@2bis10.de"

LABEL Description="Provides basic test environment for TYPO3 CMS developers"

ENV DEBIAN_FRONTEND=noninteractive
ENV SHARE_USER=webadmin
ENV SHARE_UID=1000
ENV SHARE_GROUP=webadmin
ENV SHARE_GID=1000
RUN groupadd --gid $SHARE_GID $SHARE_GROUP
RUN useradd --gid $SHARE_GROUP --uid $SHARE_UID $SHARE_USER
RUN mkdir -p /var/webadmin
RUN chown $SHARE_USER:$SHARE_GROUP /var/webadmin
RUN mkdir -p /var/www/typo3/typo3conf/ext
RUN echo "<?php\n\$GLOBALS['TYPO3_CONF_VARS']['SYS']['systemLocale']='en_US.UTF-8';\n\$GLOBALS['TYPO3_CONF_VARS']['GFX']['im_version_5']='gm';" > /var/www/typo3/typo3conf/AdditionalConfiguration.php
RUN chown -R www-data:www-data /var/www/typo3
RUN cd /var/www/typo3;ln -s ../../webadmin/typo3_src typo3_src;ln -s typo3_src/index.php index.php;ln -s typo3_src/typo3 typo3;ln -s typo3_src/composer.json;ln -s typo3_src/composer.lock
RUN echo "mysql-server-5.5 mysql-server/root_password password {{password}}" | debconf-set-selections
RUN echo "mysql-server-5.5 mysql-server/root_password_again password {{password}}" | debconf-set-selections
RUN apt-get update
RUN apt-get upgrade -y --force-yes
RUN apt-get install -y --force-yes locales
ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LC_TYPE=en_US.UTF-8
RUN echo 'en_US.UTF-8' > /etc/default/locale
RUN echo 'LANGUAGE=en_US.UTF-8' >> /etc/environment
RUN echo 'LC_ALL=en_US.UTF-8' >> /etc/environment
RUN echo 'LANG=en_US.UTF-8' >> /etc/environment
RUN echo 'LC_TYPE=en_US.UTF-8' >> /etc/environment
RUN sed --in-place 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen
RUN locale-gen
RUN update-locale
RUN apt-get install -y --force-yes wget curl net-tools vim git man-db inotify-tools locate
RUN apt-get install -y --force-yes mysql-server mysql-client
RUN apt-get install -y --force-yes apache2 apache2-utils 
RUN apt-get install -y --force-yes php5 php5-cli libapache2-mod-php5 php5-mysqlnd php5-curl php5-gd php5-apcu
RUN apt-get install -y --force-yes graphicsmagick
RUN sed --in-place 's/\/var\/www\/html/\/var\/www\/typo3#NEXT#/g' /etc/apache2/sites-available/000-default.conf
RUN sed --in-place 's/#NEXT#/\n\t<Directory \/var\/www\/typo3>#NEXT#/g' /etc/apache2/sites-available/000-default.conf
RUN sed --in-place 's/#NEXT#/\n\t\t<IfModule mod_php5.c>#NEXT#/g' /etc/apache2/sites-available/000-default.conf
RUN sed --in-place 's/#NEXT#/\n\t\t\tphp_admin_value upload_max_filesize "10M"#NEXT#/g' /etc/apache2/sites-available/000-default.conf
RUN sed --in-place 's/#NEXT#/\n\t\t\tphp_admin_value post_max_size "15M"#NEXT#/g' /etc/apache2/sites-available/000-default.conf
RUN sed --in-place 's/#NEXT#/\n\t\t\tphp_admin_value max_execution_time "240"#NEXT#/g' /etc/apache2/sites-available/000-default.conf
RUN sed --in-place 's/#NEXT#/\n\t\t<\/IfModule>#NEXT#/g' /etc/apache2/sites-available/000-default.conf
RUN sed --in-place 's/#NEXT#/\n\t<\/Directory>/g' /etc/apache2/sites-available/000-default.conf
RUN sed --in-place 's/\/var\/www\/html/\/var\/www\/typo3/g' /etc/apache2/sites-available/default-ssl.conf
RUN mkdir -p /var/www/typo3; chgrp www-data /var/www/typo3; chmod 775 /var/www/typo3
RUN echo "apc.shm_size=128M" > /etc/php5/apache2/conf.d/20-apcu-custom.ini
RUN a2enmod rewrite
RUN a2enmod expires
RUN service apache2 restart
RUN gpasswd -a www-data webadmin
RUN gpasswd -a webadmin www-data
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
