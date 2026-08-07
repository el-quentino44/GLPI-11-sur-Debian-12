# Installation de GLPI 11.0.8 sur Debian 12 BookWorm

## Introduction
***GLPI*** est une solution de gestion de parc informatique, de helpdesk (ticketing), et propose bien d’autres fonctionnalités. Indispensable en entreprise, nous allons voir comment procéder à son installation complète de base.

## Prérequis
L'installation s'appuie sur la stack **LAMP**, garante d'une infrastructure fiable, performante et 100 % open source :
- Un système d'exploitation basé sur Linux (**Debian 12**)
- Un serveur web **Apache2**
- Une base de données **MariaDB**
- Un langage de traitement **PHP**

## Préparation de l'environnement
### 1. Mise à jour du système (si c'est pas déjà fait)
Ouvrez votre terminal et lancez ces commandes pour mettre à jour le système:
- Ouvrir le fichier ***sources.list***  
```
nano /etc/apt/sources.list
```
- Mettre **#** devant le BOOKWARM et mettre en dessous
```
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
```

- Enregistrer les modifications et mettre à jour le système
```
sudo apt update && sudo apt upgrade -y
```

### 2. Installation des dépendances

- Après avoir installé Debian (VM ou serveur physique), commencez par installer les paquets requis : 
```
sudo apt install apache2 php mariadb-server
```
- Ensuite ajoutez les extensions PHP nécessaires : 
```
sudo apt-get install php8.2-fpm php-xml php-common php-json php-mysql php-mbstring php-curl php-gd php-intl php-zip php-bz2 php-imap php-apcu php-ldap
```

### 3. Configuration de la timezone 

La ***timezone*** est à configurer en fonction du fuseau horaire (**ici, c'est paris**),
```
timedatectl set-timezone "Europe/Paris"

mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
```

### 4. Création de la base de données
Exécutez : 
```
sudo mysql_secure_installation
```

Répondez comme suit :

- Switch to unix_socket → N
- Change the root password → Y (choisissez un mot de passe fort)
- Remove anonymous users → Y
- Disallow root login remotely → Y
- Remove test database → Y
- Reload privilege tables → Y

Ensuite, connectez-vous : 
```
sudo mysql -u root -p
```
Puis créez la base et l'utilisateur :
```
CREATE DATABASE nom_de_la_bd;

GRANT ALL PRIVILEGES ON nom_de_la_bd.* TO glpi_admin@localhost IDENTIFIED BY "@SuperP4ssword";

FLUSH PRIVILEGES;

EXIT;
```

## Installation de GLPI
### 1. Télécharger et extraire les sources 

```
wget https://github.com/glpi-project/glpi/releases/download/11.0.8/glpi-11.0.8.tgz

sudo tar -xzvf glpi-11.0.8.tgz -C /var/www/
```
- Puis donnez les droits :
```
sudo chown www-data /var/www/glpi -R
```
### 2. Déplacer les fichiers de configuration et logs

```
sudo mkdir /etc/glpi
sudo chown www-data /etc/glpi/
sudo mv /var/www/glpi/config /etc/glpi

sudo mkdir /var/lib/glpi
sudo chown www-data /var/lib/glpi/
sudo mv /var/www/glpi/files /var/lib/glpi

sudo mkdir /var/log/glpi
sudo chown www-data /var/log/glpi
```
### 3. Configurer GLPI

- Créer le fichier ***downstream.php*** et y coller le contenu ci-après. Ce fichier va servir de pont de redirection pour charger les fichiers de configuration de GLPI.
```
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
    require_once GLPI_CONFIG_DIR . '/local_define.php';
}
```

- Créer ensuite ***local_define.php*** et y coller le contenu ci-après. Ce fichier va servir à redéfinir les dossiers d'enregistrement, de logs, de configuration, de sécurité...
```
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi/files');
define('GLPI_LOG_DIR', '/var/log/glpi');
```

### 4.

  
