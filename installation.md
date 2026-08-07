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
- Mettez **#** devant le BOOKWARM et mettre en dessous
```
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
```

- Enregistrez les modifications et mettre à jour le système
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
- Puis donnez les droits à ***www-data*** qui est l'utilisateur système par défaut du serveur web Apache2:
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

- Créez le fichier ***downstream.php*** et y coller le contenu ci-après. Ce fichier va servir de pont de redirection pour charger les fichiers de configuration de GLPI.
```
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
    require_once GLPI_CONFIG_DIR . '/local_define.php';
}
```

- Créez ensuite ***local_define.php*** et y coller le contenu ci-après. Ce fichier va servir à redéfinir les dossiers d'enregistrement, de logs, de configuration, de sécurité...
```
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi/files');
define('GLPI_LOG_DIR', '/var/log/glpi');
```

### 4. Configuration d'Apache2

- Désactivez le site par défaut :
```
sudo a2dissite 000-default.conf
```
- Créez un ***VirtualHost*** qui va abriter la configuration de notre serveur GLPI
```
sudo nano /etc/apache2/sites-available/votre-domaine.conf
```
- Collez y le contenu suivant et ajuster au choix
```
<VirtualHost *:80>
    ServerName votre-nom-de.domaine
    DocumentRoot /var/www/glpi/public

    <Directory /var/www/glpi/public>
        Require all granted
        AllowOverride All
        RewriteEngine On
        Options FollowSymlinks
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>

    <FilesMatch \.phpgt;
        SetHandler "proxy:unix:/run/php/php8.2-fpm.sock|fcgi://localhost/"
    </FilesMatch>
</VirtualHost>
```

- Activez les modules : 
```
sudo a2enmod rewrite
sudo a2ensite votre-domaine.conf
sudo systemctl restart apache2
```

### 5. Utiliser PHP-FPM

- Activez ***PHP-FPM***, une version optimisée de PHP qui permet d'éxecuter du code PHP en arrière plan, séparément du serveur web
```
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php8.2-fpm
sudo systemctl reload apache2
```

- Editez le fichier ***php.ini*** et modifier le paramètre suivant : 
```
session.cookie.httponly = on
```

- Puis redémarrez ces deux services :
```
sudo systemctl restart php8.2-fpm.service
sudo systemctl restart apache2
```

## Accès à GLPI

Ouvrez un navigateur et rendez-vous sur 127.0.1.1 si installé en local, ou sur votre domaine.
Choisissez **Installer**, puis configurez l’accès à la base :

- Serveur SQL : **localhost**
- Utilisateur SQL : **glpi_admin**
- Mot de passe : **@SuperP4ssword**
