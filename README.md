# Moodle-on-docker

Instance Docker Compose simple de Moodle créée dans pour le projet ANR Tea-Time.

Cette pile de service docker compose a pour objectif de fournir une instance de Moodle simple pour des tests et preuves de concepts. Elle repose sur 2 services : une base de de données SQL et un service Apache-PHP exposant une instance de Moodle.

# Versions des services

- Moodle : 4.0.3
- Php : 8.2 (debian 11)
- MariaDB : 11

# Prérequis

- Docker >= 20
- Docker Compose >= 2

# Installation

## Nettoyage d'installation pre-existante

1. Supprimez tous les volumes et services pré-existant :

```
docker compose down -v
```

## Initialisation de la base de données et de la structure Moodle

1. Assurez-vous que l'image du worker est à jour :

```
docker compose build
```

2. Lancez le service de base de données uniquement :

```
docker compose up db -d
```

3. Lancez le service worker de moodle en exécution unique pour une installation non interactive :

```
docker compose run --rm -u www-data -ti worker php admin/cli/install.php --chmod=2750 --lang=fr --wwwroot=http://localhost --dataroot=/var/www/moodledata --dbtype=mariadb --dbhost=db --dbname=moodle --dbuser=moodleusr --dbpass=$(cat ./secrets/db-user-pwd.pwd) --prefix=mdl_ --fullname="My Moodle" --shortname="mymoodle" --adminuser=admin --adminpass=adminpass --adminemail=admin@mymoodle.org --supportemail=support@mymoodle.org --agree-license --non-interactive
```

Paramètres d'installation de la structure moodle : 

- _chmod_ : droit du dossier des données (**ne pas modifier**)
- _lang_ : langue par défaut
- _wwwroot_ : url d'accès de base au moodle (_ne pas modifier pour une installation locale de test_)
- _dataroot_ : chemin d'accès dans le conteneur vers les données (**ne pas modifier**)
- _dbtype_ : type de base de données utilisée (**ne pas modifier**)
- _dbhost, dbname, dbuser, dbpass_ : informations d'accès à la db  (**modifier au besoin en accord avec les informations fournes au service**)
- _prefix_ : prefixe des noms de tables utilisées dans la bd
- _fullname, shortname_ : noms long et court de l'instance de Moodle
- _adminuser, adminpass, adminemail_ : information du compte administrateur à créer initialement
- _supportemail_ : adresse mel du courriel de support
- _agree-license_ : acceptation automatique de la license de Moodle (obligatoire en non interactif)


4. Lancez le reste des services (le worker moodle)

```
docker compose up -d
```

# Accès à Moodle

Une fois la pile de service lancée, vous pouvez accéder à l'interface web de Moodle depuis un navigateur de la machine hôte à l'adresse http://localhost (port par défaut 80).

Les identifiants du compte administrateur sont ceux renseignés à l'installation de Moodle au moment de l'initialisation de la base de données et de la structure Moodle (par défaut : admin/adminpass).

# Personnalisation de la configuration

## Configuration PHP

Vous pouvez modifier le fichier de configuration PHP en éditant le fichier `worker/worker_php.ini` puis en redémarrant le service worker `docker compose restart worker`.

## Configuration Moodle

le fichier _config.php_ de Moodle, présent dans le volume _web-content_ peut être modifié simple en le copiant depuis le conteneur sur l'hôte puis en faisant la copie inverse :

1. Copie Conteneur -> hôte : `docker compose cp worker:/var/www/html/config.php <chemin-local>/`
2. Edition du fichier config.php dans \<chemin-local>/
3. Copie -> hôte : `docker compose cp <chemin-local>/config.php worker:/var/www/html/config.php`

