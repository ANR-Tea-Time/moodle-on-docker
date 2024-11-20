FROM php:8.2-apache

LABEL version="2.0"
LABEL maintainer="remi.venant@univ-lemans.fr"
LABEL description="Complete Moodle fpm image, with uninstalled moodle"

ARG MOODLE_VERSION=403

# On se place dans un dossier temporaire d'installation pour assurer un nettoyage correcte à la fin
WORKDIR /tmp

# Configuration de pecl derrière un proxy si requis
RUN test -z "${http_proxy}" || pear config-set http_proxy ${http_proxy}

# Installation Package système requis pour les extensions php
RUN apt-get update \
  && apt-get install -y \
  cron \
  git \
  libicu-dev \
  libzip-dev \
  unzip \
  libfreetype6-dev \
  libjpeg62-turbo-dev \
  libpng-dev \
  libcurl3-dev \
  libxml2-dev \
  libtidy-dev \
  # Reconfiguration de php gd
  && docker-php-ext-configure gd --with-freetype --with-jpeg \
  # Installation des extenstions gérées par pecl
  && pecl install apcu \
  # Installation des extensions gérées par php-ext-install
  && docker-php-ext-install -j$(nproc) \
  intl \
  opcache \
  zip \
  gd \
  curl \
  mysqli pdo_mysql \
  soap \
  exif \
  tidy \
  # Activation d'extensions php
  && docker-php-ext-enable \
  intl \
  opcache \
  apcu \
  zip \
  mysqli pdo_mysql \
  gd

#  Configuration git global et création structures de dossier
RUN mkdir -p /var/www/moodledata \
  && git config --global user.email "anonym@mail.com" \
  && git config --global user.name "Anne Onym"

# Récupération moodle
RUN git clone -b MOODLE_${MOODLE_VERSION}_STABLE https://github.com/moodle/moodle.git /var/www/html

# Changement des droit sur la structure /app
RUN  chown -R www-data:www-data /var/www \
  && chmod -R 0755 /var/www/html \
  && chmod 0777 /var/www/moodledata

# Installation du cron pour www-data
RUN echo "* * * * * /usr/local/bin/php  /var/www/html/admin/cli/cron.php" > current_cron \
  && crontab -u www-data current_cron \
  && rm current_cron

# Nettoyage,
RUN rm -rf /var/lib/apt/lists/* && rm -rf /tmp/*

# Basucle sur le dossier /app/
WORKDIR /var/www/html

# Déclaration des volumes de données
VOLUME /var/www/html /var/www/moodledata

# Command de départ: le superviseur
CMD ( cron -f & ) && apache2-foreground
