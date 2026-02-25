<div align="center">
  
# Journal de bord
</div>

## Ce ci est mon journal de bord, il a pour but de faciliter le suivi de mon avancement dans le projet.
### 03/02/2026
Je prends connaissance de notre projet qui consiste à faire une station météo. Les informations de la station seront lisibles via un site web et une application Android.
Mon rôle dans le projet est de mettre en place le serveur avec la base de données et le site web.<br>
Afin de partir sur une bonne base nous décidons d'un scénario où notre station météo sera utilisée. Nous choisissons une station météo pour une exploitation agricole. Il reste à voir dans quel contexte une station qui donne les informations en instantané est utile.

Création du GitHub afin de suivre le projet et de pouvoir partager mon travail avec mes camarades.
### 05/02/2026
Je me crée un OneNote personnel afin d'avoir un endroit brouillon pour travailler

Pour mon serveur, je choisis de faire le serveur sur Linux, car beaucoup d'entreprises utilisent ça et dans une société qui dépend de plus en plus de Windows, je trouve ça intéressant de savoir être indépendant de leur service. J'ai le souvenir d'avoir déjà fait un serveur web sur Linux l'année dernière, mais je ne me souviens plus très bien comment on avait fait. Je regarde donc une vidéo tuto pour faire un serveur LAMP (Linux, Apache, MySQL et PHP). C'est exactement ce dont j'ai besoin.

Lien de la vidéo : [https://www.youtube.com/watch?v=RpAffaESwyk&t=7s](url)

Vu que pour le moment je suis en phase de test, je décide de commencer sur une machine virtuelle via VirtualBox. Après l'installation de Ubuntu Server, je mets en place mon serveur LAMP :

Installation de l'environnement LAMP
```bash
sudo apt install apache2 -y
sudo apt install mysql-server -y
sudo apt install php -y # language coté serveur
sudo apt install libapache2-mod-php -y
sudo apt install php-mysql -y
```

Connexion à MySQL
```sql
sudo mysql -u root -> connexion à mysql
```

Création de la base de données ainsi que de la table
```sql
CREATE DATABASE stationmeteo;
USE stationmeteo;
CREATE TABLE meteo (
    id INT AUTO_INCREMENT PRIMARY KEY,
    temperature FLOAT NOT NULL,
    humidite FLOAT NOT NULL,
    vitesse_vent FLOAT NOT NULL,
    date DATETIME NOT NULL
);
```

Création d'un utilisateur pour la base de données
```sql
CREATE USER 'meteo'@'localhost' IDENTIFIED BY 'meteo123';
GRANT ALL PRIVILEGES ON stationmeteo.* TO 'meteo'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Création des fichiers pour visualiser la table météo
```bash
sudo nano /var/www/html/index.php
sudo nano /var/www/html/meteo.php
```
Code des fichiers

index.php
```php
<?php include 'meteo.php'; ?>
<!DOCTYPE html>
<html>
    <head>
        <title>Station Météo</title>
    </head>
    <body>
        <h3>Données météo : </h3>
        <table border="1">
            <tr>
                <th>Date</th>
                <th>Température</th>
                <th>Humidité</th>
                <th>Vitesse du vent</th>
                <th>Point de rosé</th>
            </tr>
<?php foreach($meteoData as $row): ?>
    <tr>
        <td><?= $row['date'] ?></td>
        <td><?= $row['temperature'] ?></td>
        <td><?= $row['humidite'] ?></td>
        <td><?= $row['vitesse_vent'] ?></td>
        <td><?= $row['point_rose'] ?></td>
    </tr>
<?php endforeach; ?>
        </table>
    </body>
</html>
```

meteo.php
```php
<?php
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

// Information de connexion à MySQL
$host = "localhost";
$user = "meteo_app";	// root pour la vm de test
$password = "meteo_app1234"; // vide car pas de mot de passe
$dbname = "stationmeteo";

// Connexion à MySQL
try {
	// Connexion PDO
	$conn = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $user, $password);
	$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

	// Récupérer toutes les données météo
	$stmt = $conn->query("SELECT * FROM meteo ORDER BY date DESC");
	$meteoData = $stmt->fetchAll(PDO::FETCH_ASSOC);


} catch (PDOException $e) {
	echo "Erreur : " . $e->getMessage();
} 
?>
```

<div align="center">
	
![Image avant la réquete](https://github.com/LucienRpro/Projet_station_meteo_bts/blob/main/weather_station_view%20before_insert.png)
</div>

Insertion de données dans la table
```sql
USE stationmeteo;

INSERT INTO meteo (temperature, humidite, vitesse_vent, date, point_rose)
VALUES (22.5, 55.2, 12.3, NOW(), 12.1);
```
<div align="center">
	
![Image après la réquete](https://github.com/LucienRpro/Projet_station_meteo_bts/blob/main/weather_station_view%20after_insert.png)
</div>

### 17/02/2026
Après avoir expérimenté et compris à quoi ressemblerait mon projet, je décide de faire la modélisation de ma base de données, pour cela je fais un diagramme MLD. Cela permettra de montrer rapidement à une personne extérieure au projet ce que contient la base de données. Et ça prépare le terrain pour programmer la base de données en SQL ensuite.

Diagramme UML :

<div align="center">
	
![Diagramme uml](https://github.com/LucienRpro/Projet_station_meteo_bts/blob/main/weather_station_db_uml.png)
</div>

MLD (Modèle logique de données) :

<div align="center">
	
![MLD](https://github.com/LucienRpro/Projet_station_meteo_bts/blob/main/weather_station_db_mld.png)
</div>

### 18/02/2026
Ajout du MPD qui se traduit directement en langage SQL prêt à être utilisé dans mon cas

MPD (Modèle Physique de données) :

```sql
CREATE TABLE IF NOT EXISTS `weather_data` (
    `id` INTEGER NOT NULL AUTO_INCREMENT,
    `temperature` FLOAT NOT NULL,
    `humidity` FLOAT NOT NULL,
    `wind_speed` FLOAT NOT NULL,
    `dew_point` FLOAT NOT NULL,
    `recorded_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY(`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `users` (
    `id` INTEGER NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(50) NOT NULL UNIQUE,
    `password_hash` VARCHAR(255) NOT NULL,
    `role` ENUM('admin', 'user') NOT NULL DEFAULT 'user',
    `email` VARCHAR(255) NOT NULL,
    `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY(`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 19/02/2026
Installation du serveur officiel sur un ordinateur que j'ai en plus chez moi. Je trouve cette solution avantageuse, car je ne suis pas dépendant d'une machine qui serait dans l'établissement de mon BTS. L'autre avantage est que, si besoin, je peux passer mon ordinateur portable à un camarade pour qu'il avance sur une partie liée au serveur.

### 20/02/2026
J'ai eu plusieurs problèmes sur mon PC portable avec l'installation d'Ubuntu Server que j'ai essayé de résoudre sans succès

### 21/02/2026
J'ai réussi à installer Ubuntu serveur, pour cela j'ai dû :

Créer le fichier de configuration du wifi /etc/netplan/01-wifi.yaml
```yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlo1:
      dhcp4: true
      access-points:
        "MON WIFI":
          password: "MDP"
```
Modifier le fichier /etc/default/grub en mettant cette ligne pour soulager le CPU 

```bash 
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pcie_aspm=off"
```
Puis, afin de mettre le fichier à jour
```bash
sudo update-grub
sudo reboot
```
### 24/02/2026
Aujourd'hui j'ai fait l'installation de tout le nécessaire pour le serveur LAMP, exactement comme je l'avais fait pour la machine virtuelle précédemment
J'ai aussi installé OpenSSH pour me connecter à distance sur mon serveur en utilisant VS Code pour pouvoir modifier les fichiers dans var/html afin de coder plus facilement le site web du serveur.
```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
```
Puis j'ai crée la base de donnée weather_station
```sql
CREATE DATABASE weather_station;
```
A qui j'ai associer un utilsateur mysql qui sera dédier à la connexion pour mon site
```sql
CREATE USER 'weather_station_user'@'localhost' IDENTIFIED BY 'MYqE),45]jg2';
GRANT ALL PRIVILEGES ON weather_station.* TO 'weather_station_user'@'localhost';
FLUSH PRIVILEGES;
```
Et ensuite j'ai sécuriser mysql avec
```bash
sudo mysql_secure_installation
```
Voici la configuration :

- VALIDATE PASSWORD : activé (niveau MEDIUM) pour imposer des mots de passe robustes aux comptes MySQL (réduction du risque de force brute) ; choix MEDIUM pour être suffisamment strict sans bloquer l’utilisation.
- Changement du mot de passe root : non, car l’accès admin se fait via authentification socket (sudo mysql) ; évite de gérer un mot de passe root MySQL et limite l’attaque par force brute.
- Suppression des utilisateurs anonymes : oui, pour supprimer des comptes inutiles pouvant permettre un accès non authentifié.
- Interdiction du login root à distance : oui, pour empêcher toute connexion admin depuis le réseau et réduire la surface d’attaque.
- Suppression de la base test : oui, car inutile en production/projet et parfois utilisée pour des tests d’exploitation.
- Rechargement des privilèges : oui, pour appliquer immédiatement les changements de sécurité.

### 25/02/2026
Création des tables weather_data et users dans la base de données, voici le script sql :
```sql
CREATE TABLE IF NOT EXISTS `weather_data` (
    `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
    `temperature` DECIMAL(4,1) NOT NULL,
    `humidity` DECIMAL(4,1) NOT NULL,
    `wind_speed` DECIMAL(4,1) NOT NULL,
    `dew_point` DECIMAL(4,1) NOT NULL,
    `recorded_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY(`id`),
    INDEX idx_recorded_at ('recorded_at')
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `users` (
    `id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(50) NOT NULL UNIQUE,
    `password_hash` VARCHAR(255) NOT NULL,
    `role` ENUM('admin', 'user') NOT NULL DEFAULT 'user',
    `email` VARCHAR(255) NOT NULL,
    `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY(`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```
Les données sont en décimal 4,1 pour couvrir une plage large allant de -999,1 à 999,9. Pour recorded_at et created_at la valeur par défaut est la valeur lors de la création de la donnée. Et pour faciliter les affichages en fonction des dates, j'ai créé un index.Les données sont en décimal 4,1 pour couvrir une plage large allant de -999,1 à 999,9. pour recorded_at et created_at la valeur par défaut est la valeur lors de la création de la donnée. Et pour faciliter les affichage en fonction des dates j'ai créer un index.
