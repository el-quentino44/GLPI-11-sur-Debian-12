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

### 2. Installation des dépendances

Après avoir installé Debian (VM ou serveur physique), commencez par installer les paquets requis : 
```
sudo apt install apache2 php mariadb-server
```
Ensuite ajoutez les extensions PHP nécessaires : 
```
sudo apt-get install php8.2-fpm php-xml php-common php-json php-mysql php-mbstring php-curl php-gd php-intl php-zip php-bz2 php-imap php-apcu php-ldap
```

### 3. Configuration de la timezone 

En fonction bien sûr de votre fuseau horaire (**ici, c'est paris**),
```
timedatectl set-timezone "Europe/Paris"

mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
```

### 4.Création de la base de données
Excécutez : 
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


  
