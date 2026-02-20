<div align="center">
  
# Journal de bord
</div>

## Ce ci est mon journal de bord, il a pour but de faciliter le suivie de mon avancement dans le projet.
### 03/02/2026
Je prend connaissance de notre projet qui consite à faire une station météo, les informations la station serons lisible via un site web et une appplication android.
Mon rôle dans le projet est de mettre en place le server avec la base de donnée et le site web.<br>
Afin de partir sur une bonne base nous décidons d'un sénario où notre station météo sera utilisé. Nous choisons une station météo qui sera utilisé par une exploitation agricole. Il reste à voir dans quelle contexte une station qui donne les infos en instantané est utile.

Crétation du github afin de suivre le projet et de pouvoir partager mon travail avec mes camarades.
### 05/02/2026
Je me créer un one note perso afin d'avoir un endroit brouillon pour travailler

Pour mon serveur je choisis de faire le serveur sur Linux, car beaucoup d'entreprise utlise ça et dans une société qui dépend de plus en plus de windows je trouve ça intéressant de savoir être indépendant de leur service. J'ai le souvenir d'avoir déjà fait un serveur web sur linux l'année dernnière mais je me souvien plus très bien comment on avait fait. Je regarde donc une vidéo tuto pour faire un serveur LAMP (Linux, Apache, MySql et Php) c'est exactement ce dont j'ai besoin.

Lien de la vidéo : [https://www.youtube.com/watch?v=RpAffaESwyk&t=7s](url)

Vue que pour le moment je suis en phase de test je décide de faire tout ça sur une machine virtuel via VirtualBox, après l'installation de ubuntu server je met en place mon serveur LAMP :

Installation de l'environement LAMP
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

Crétation de la base de donnée ainsi que de la table
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

Création des fichier pour visualiser la table meteo
```bash
sudo nano /var/www/html/index.php
sudo nano /var/www/html/meteo.php
```
Code des fichiers
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
