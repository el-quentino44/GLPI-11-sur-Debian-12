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
### 1. Mise à jour du système (Si c'est pas déjà fait)
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


  
