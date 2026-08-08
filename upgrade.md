# Mise à niveau de GLPI 10 vers GLPI 11 sur Debian 12 Bookworm

## Introduction
***GLPI*** est la solution de référence pour la gestion de parc informatique et de helpdesk. Dans ce TP, nous allons voir comment procéder à la mise à niveau complète d’une instance GLPI 10 vers la version 11.

## Prérequis
- Un serveur GLPI 10
- Un accès à Internet

## Préparation de la mise à niveau

### 1. Sauvegarde des configurations
Générez les sauvegardes des base de données et fichiers (configuration, données et plugins)

```
# Base de données
mysqldump -u root -p --single-transaction glpi > /backup/glpi_pre_migration.sql

# Fichiers (configuration, données et plugins)
tar -czf /backup/glpi_files_pre_migration.tar.gz /var/www/glpi /etc/glpi /var/lib/glpi
```

### 2. Désactiver les plugins incompatibles 
Dans GLPI 10, allez dans ***Configuration*** > ***Plugins*** et désactivez tous ceux sans version pour la 11. Supprimez les répertoires des incompatibles. Cette étape n'est pas optionnelle : c'est la cause n°1 d'échec à la commande de mise à jour.

### 3. Mettre à jour les fichiers de GLPI
- Téléchargez toujours la dernière version stable de la ligne 11.x, pas forcément la 11.0.0.
```
cd /tmp
wget https://github.com/glpi-project/glpi/releases/download/11.0.0/glpi-11.0.0.tgz
tar -xzf glpi-11.0.0.tgz
```
- Remplacez les fichiers en préservant configuration, données et plugins
```
rsync -av --delete /tmp/glpi/ /var/www/glpi/ --exclude plugins/ --exclude marketplace/
chown -R www-data:www-data /var/www/glpi
