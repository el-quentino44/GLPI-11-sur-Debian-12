# Installation de GLPI 11.0.8 sur Debian 12 Bookworm

## Introduction
***GLPI*** est une solution de gestion de parc informatique, de helpdesk (ticketing), et propose bien d’autres fonctionnalités. Indispensable en entreprise, nous allons voir comment procéder à son installation complète de base.

## Prérequis
L'installation s'appuie sur la stack **LAMP**, garante d'une infrastructure fiable, performante et 100 % open source :
- Un système d'exploitation basé sur Linux (**Debian 12**)
- Un serveur web **Apache2** en mode Event avec **PHP-FPM**
- Une base de données **MariaDB**
- Un langage de traitement **PHP**

Bien sûr, il faut prévoir une connexion à Internet pour le téléchargement de certains paquets.

## Préparation de l'environnement
### 1. Mise à jour du système (si ce n'est pas déjà fait)
Ouvrez votre terminal et lancez ces commandes pour mettre à jour le système :
- Ouvrir le fichier ***sources.list***  
```
nano /etc/apt/sources.list
```
- Mettez **#** devant les lignes contenant bookworm et ajouter les lignes suivantes en dessous :
```
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
```

- Enregistrez les modifications et mettez à jour le système :
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
sudo apt-get install php8.2-fpm php-xml php-common php-json php-mysql php-mbstring php-curl php-gd php-intl php-zip php-bz2 php-imap php-apcu php-ldap php-bcmath php-redis
```

### 3. Configuration de la timezone 

La ***timezone*** est à configurer en fonction du fuseau horaire (**ici, c'est Europe/Paris**),
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

Ensuite, connectez-vous à mysql : 
```
sudo mysql -u root -p
```
Puis créez la base de données et l'utilisateur :
```
CREATE DATABASE nom_de_la_bd;
CREATE USER 'glpi_admin'@'localhost' IDENTIFIED BY '@SuperP4ssword';
GRANT ALL PRIVILEGES ON nom_de_la_bd.* TO 'glpi_admin'@'localhost';
GRANT SELECT ON `mysql`.`time_zone_name` TO 'glpi_admin'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Installation de GLPI
### 1. Télécharger et extraire les sources 

```
wget https://github.com/glpi-project/glpi/releases/download/11.0.8/glpi-11.0.8.tgz

sudo tar -xzvf glpi-11.0.8.tgz -C /var/www/
```
- Attribuez les droits à ***www-data*** qui est l'utilisateur système par défaut du serveur web Apache2:
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

- Créez le fichier ***downstream.php*** :
```
sudo nano /var/www/glpi/inc/downstream.php
```

- Collez-y le contenu ci-après. Ce fichier va servir de pont de redirection pour charger les fichiers de configuration de GLPI.
```
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
    require_once GLPI_CONFIG_DIR . '/local_define.php';
}
```

- Créez ensuite ***local_define.php*** :
```
sudo nano /etc/glpi/local_define.php
```
  
- Y collez le contenu ci-après. Ce fichier va servir à redéfinir les dossiers d'enregistrement, de logs, de configuration, de sécurité...
```
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_DOC_DIR', GLPI_VAR_DIR);
define('GLPI_CACHE_DIR', GLPI_VAR_DIR . '/_cache');
define('GLPI_CRON_DIR', GLPI_VAR_DIR . '/_cron');
define('GLPI_GRAPH_DIR', GLPI_VAR_DIR . '/_graphs');
define('GLPI_LOCAL_I18N_DIR', GLPI_VAR_DIR . '/_locales');
define('GLPI_LOCK_DIR', GLPI_VAR_DIR . '/_lock');
define('GLPI_PICTURE_DIR', GLPI_VAR_DIR . '/_pictures');
define('GLPI_PLUGIN_DOC_DIR', GLPI_VAR_DIR . '/_plugins');
define('GLPI_RSS_DIR', GLPI_VAR_DIR . '/_rss');
define('GLPI_SESSION_DIR', GLPI_VAR_DIR . '/_sessions');
define('GLPI_TMP_DIR', GLPI_VAR_DIR . '/_tmp');
define('GLPI_UPLOAD_DIR', GLPI_VAR_DIR . '/_uploads');
define('GLPI_INVENTORY_DIR', GLPI_VAR_DIR . '/_inventories');
define('GLPI_THEMES_DIR', GLPI_VAR_DIR . '/_themes');
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
- Collez-y le contenu suivant et ajustez-le au choix
```
# Configuration HTTP

<VirtualHost *:80>

    ServerName votre-nom-de.domaine
    DocumentRoot /var/www/glpi/public

    <Directory /var/www/glpi/public>
        Require all granted
        AllowOverride None
        RewriteEngine On
        Options FollowSymlinks
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php/php8.2-fpm.sock|fcgi://localhost/"
    </FilesMatch>

     ErrorLog ${APACHE_LOG_DIR}/glpi-error.log
     CustomLog ${APACHE_LOG_DIR}/glpi-access.log combined
</VirtualHost>
```

- Activez les modules : 
```
sudo a2enmod rewrite
sudo a2ensite votre-domaine.conf
sudo systemctl restart apache2
```

**N.B** : S'il existe une autorité de certification SSL, on peut générer une clé privée + une demande (CSR) pour obtenir le certificat signé. Alors on pourra paramétrer le site pour utiliser du **HTTPS** au lieu **HTTP** par défaut comme suit : 

```
# Redirection HTTP → HTTPS

<VirtualHost *:80>
    ServerName votre-nom-de.domaine
    Redirect permanent / https://votre-nom-de.domaine/
</VirtualHost>

# Configuration HTTPS

<VirtualHost *:443>
    ServerName votre-nom-de.domaine
    DocumentRoot /var/www/glpi/public

    # Certificats SSL
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/certificat-signe.crt
    SSLCertificateKeyFile /etc/ssl/private/cle-privee.key

    <Directory /var/www/glpi/public>
        Require all granted
        AllowOverride None
        RewriteEngine On
        Options FollowSymlinks
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php/php8.2-fpm.sock|fcgi://localhost/"
    </FilesMatch>

    ErrorLog ${APACHE_LOG_DIR}/glpi-error.log
    CustomLog ${APACHE_LOG_DIR}/glpi-access.log combined
</VirtualHost>
```

### 5. Utilisation de PHP-FPM

- Activez ***PHP-FPM***, une version optimisée de PHP qui permet d'exécuter du code PHP en arrière plan, séparément du serveur web
```
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php8.2-fpm
sudo systemctl reload apache2
```

### 6. Configuration du fichier php.ini

- Ouvrez le fichier ***php.ini*** de **PHP-FPM**
```
sudo nano /etc/php/8.2/fpm/php.ini
```
  
- Modifiez-y les paramètres suivants :

    - ``` session.cookie_httponly = on ``` pour indiquer au navigateur que le cookie de session ne doit être accessible que par **HTTP/HTTPS**,
 
    - ``` upload_max_filesize = 20M ``` pour définir la taille des pièces jointes pour les utilisateurs à **20Mo**

    - ``` session.cookie_secure = on ``` : à faire lorsqu'une première connexion au GLPI en **HTTPS** a déjà été faite et pour définir le protocole **HTTPS** comme l'unique moyen d'accès,

    - ``` session.cookie_samesite = Lax ```: pour bloquer le piratage venant de sites web malveillants

    - ``` max_execution_time = 60 ``` : pour indiquer que le temps d'exécution maximal d'un script PHP est **60s**

    - ``` memory_limit = 256M ``` : pour indiquer que la quantité maximale de mémoire qu'un script PHP peut utiliser est **256Mo**
      
    - ``` date.timezone = Europe/Paris ``` : pour régler le fuseau horaire de  PHP sur celui du système

    - ``` max_input_vars = 5000 ``` : pour définir le nombre maximal de variables d'entrée d'un script à 5000

    - ``` post_max_size = 20M ``` : pour définir la taille maximale des données qu'un utilisateur peut envoyer en une seule fois

### 7. Gestion des permissions 

Par défaut, l'utilisateur ***root*** est le propriétaire de tous les dossiers et fichiers de GLPI, ce qui laisse l'utilisateur web ***www-data*** sans droit d'écriture. On va donc changer de propriétaire et attribuer aux dossiers et aux fichiers systèmes des droits spécifiques. L'utilisateur ***root*** ne doit être propriétaire que du code source de GLPI.
```
chown root:root /var/www/glpi/ -R
chown www-data:www-data /etc/glpi -R
chown www-data:www-data /var/lib/glpi -R
chown www-data:www-data /var/log/glpi -R
chown www-data:www-data /var/www/glpi/marketplace -Rf
find /var/www/glpi/ -type f -exec chmod 0644 {} \;
find /var/www/glpi/ -type d -exec chmod 0755 {} \;
find /etc/glpi -type f -exec chmod 0644 {} \;
find /etc/glpi -type d -exec chmod 0755 {} \;
find /var/lib/glpi -type f -exec chmod 0644 {} \;
find /var/lib/glpi -type d -exec chmod 0755 {} \;
find /var/log/glpi -type f -exec chmod 0644 {} \;
find /var/log/glpi -type d -exec chmod 0755 {} \;
```

### 8. Vérification 

- Redémarrez les services et vérifier si les statuts sont sur ***active: (running)*** :
```
sudo systemctl restart apache2 mariadb php8.2-fpm
sudo systemctl status apache2 mariadb php8.2-fpm
```

## Accès à GLPI

Ouvrez un navigateur et rendez-vous sur 127.0.1.1 si installé en local, ou sur votre nom de domaine configuré.
Choisissez **Installer**, puis configurez l’accès à la base :

- Serveur SQL : **localhost**
- Utilisateur SQL : **glpi_admin**
- Mot de passe : **@SuperP4ssword**
- Identifiants de connexion : **glpi/glpi**
  

N'oubliez pas à la fin d'attribuer des mots de passe à vos utilisateurs par défaut ou les supprimer. Assurez-vous également que les dates sont correctes, car elles sont essentielles pour un bon traitement des tickets.

