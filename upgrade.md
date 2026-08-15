# Plan de migration de GLPI 10 vers GLPI 11 sur Debian 12 Bookworm

## Introduction
***GLPI*** est la solution de référence pour la gestion de parc informatique et de helpdesk. Dans ce TP, nous allons voir comment procéder à la mise à niveau complète d’une instance GLPI 10 vers la version 11.

## Prérequis
- Un serveur GLPI 10
- Un accès à Internet

## Préparation de la mise à niveau

### 1. Activation du mode maintenance
Pour empêcher l'utilisation de GLPI pendant la procédure comme par exemple l'émission de tickets..., il est fortement conseillé de mettre le serveur en mode maintenance :
```
php /var/www/glpi/bin/console maintenance:enable
```
À la fin de la mise à niveau, il faudra taper la même commande mais en remplaçant le ***enable*** par ***disable***

### 2. Sauvegarde des configurations
Générez les sauvegardes des base de données et fichiers (configuration, données et plugins)

```
# Base de données
mysqldump -u root -p --single-transaction glpi > /backup/glpi_pre_migration.sql

# Fichiers (configuration, données et plugins)
tar -czf /backup/glpi_files_pre_migration.tar.gz /var/www/glpi /etc/glpi /var/lib/glpi
```

### 3. Désactiver les plugins incompatibles 
Dans GLPI 10, allez dans ***Configuration*** > ***Plugins*** et désactivez tous ceux sans version pour la 11. Supprimez les répertoires des incompatibles. Cette étape n'est pas optionnelle : c'est la cause n°1 d'échec à la commande de mise à jour.

## Mise à niveau de GLPI

### 1. Mettre à jour les fichiers de GLPI
- Téléchargez toujours la dernière version stable de la ligne 11.x, pas forcément la 11.0.0 :
```
cd /tmp
wget https://github.com/glpi-project/glpi/releases/download/11.0.0/glpi-11.0.x.tgz
tar -xzf glpi-11.0.x.tgz
```
- Remplacez les fichiers en préservant les configuration, données et plugins :
```
rsync -av --delete /tmp/glpi/ /var/www/glpi/ --exclude plugins/ --exclude marketplace/ --exclude inc/downstream.php
chown -R www-data:www-data /var/www/glpi
```

### 4. Lancer la migration de la base 
```
php /var/www/glpi/bin/console db:update --no-interaction
```
Cette commande applique toutes les migrations de schéma. Suivez toute la sortie car toute erreur doit être résolue avant de continuer et jamais ignorée.

### 5. Vider le cache et les sessions
Vider le cache de GLPI permet d'***éviter les erreurs d'affichage et de bugs, et de forcer la prise en compte du nouveau code*** : 
```
php /var/www/glpi/bin/console cache:clear
rm -rf /var/lib/glpi/_sessions/*
```

### 6. Vérification post-migration
- La connexion fonctionne et la version sous Configuration > Général affiche 11.x.
- Tickets, actifs et utilisateurs sont présents avec les totaux attendus.
- L'ouverture d'un nouveau ticket fonctionne de bout en bout.
- Journaux sans erreurs récurrentes après quelques minutes d'usage.
- Réactivez et mettez à jour les plugins compatibles un par un, en testant entre chaque.
