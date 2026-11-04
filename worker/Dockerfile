FROM alpine/git:2.54.0 AS moodlecode

ARG MOODLE_VERSION=405

# RUN git config --global user.email "anonym@mail.com" \
#   && git config --global user.name "Anne Onym"

# Récupération moodle
RUN git clone -b MOODLE_${MOODLE_VERSION}_STABLE https://github.com/moodle/moodle.git /moodle

FROM php:8.3-apache

LABEL version="3.0"
LABEL maintainer="remi.venant@univ-lemans.fr"
LABEL description="Complete Moodle fpm image, with uninstalled moodle"

# On se place dans un dossier temporaire d'installation pour assurer un nettoyage correcte à la fin
WORKDIR /tmp

# Configuration de pecl derrière un proxy si requis
RUN test -z "${http_proxy}" || pear config-set http_proxy ${http_proxy}

# Installation Package système requis pour les extensions php et nettoyage
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
  gd \
  && rm -rf /var/lib/apt/lists/* && rm -rf /tmp/*

# Activation de module apache
RUN a2enmod rewrite

# Récupération moodle
COPY --from=moodlecode --chown=www-data:www-data /moodle /var/www/html

# Création structures de dossier data et Changement des droit sur la structure
RUN mkdir -p /var/www/moodledata \
  && chmod -R 0755 /var/www/html \
  && chmod 0777 /var/www/moodledata

# Installation du cron pour www-data
RUN echo "* * * * * su www-data -s /bin/sh -c '/usr/local/bin/php /var/www/html/admin/cli/cron.php' > /proc/1/fd/1 2>/proc/1/fd/2" > /etc/cron.d/moodle-crontab \
  && chmod 0644 /etc/cron.d/moodle-crontab \
  && crontab -u root /etc/cron.d/moodle-crontab

# Basucle sur le dossier /app/
WORKDIR /var/www/html

# Déclaration des volumes de données
VOLUME /var/www/html /var/www/moodledata

# Command par default : cron et apache
CMD ( cron -f & ) && apache2-foreground

# Si l'on veut séparer le cron et le worker lui-même:
# - 1 conteneur avec la commande apache2-foreground
# - 1 conteneur avec la command ["cron", "-f"]
