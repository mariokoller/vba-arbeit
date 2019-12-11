# Arbeit für das Fach V-BA
Diese Arbeit wurde erstellt von:
- Hemse Al Deomi
- Harbin Dehari
- Mario Koller

## Inhaltsverzeichnis
- [Einleitung](##Einleitung)
- [Vagrantfile](##Vagrantfile)
- [Dockerdateien](##Dockerdateien)
- [YML Datei](##YML-Datei)
- [Installation Apache & MySQL](##Installation-Apache-&-MySQL)
- [PHP Ausgabe](##PHP-Ausgabe)
---
## Einleitung
Mit unserer Arbeit wollen wir aufzeigen, wie die Automatisierung mittels Vagrant und Docker funktioniert. In dieser Arbeit, wird eine Linux Maschine automatisiert erstellt und auf dieser wird der Webservice inkluse Datenbank installiert, das Ziel ist die Anzeige der DB-Inhalte.

---
## Vagrantfile
### **Linux Installation**
> [&uarr; *Zum Inhaltsverzeichnis*](##Inhaltsverzeichnis)

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Definition der Vagrant Box
Vagrant.configure("2") do |config|
# Definition des Box Namens
config.vm.define "dockermachine" do |dockermachine|
  # Definition der zur Verfügung gestellten Box
  dockermachine.vm.box = "ubuntu/xenial64"
  # Definiton der Hypervisors
  dockermachine.vm.provider "virtualbox" do |vb|
  # Definition des RAM's auf 2GB
  vb.memory = "2048"
  # Definition der virtuellen CPU's
  vb.cpus = "2"
  end
  # Gibt den Ordner Share auf der Docker VM frei und synchronisiert diese
  config.vm.synced_folder "share", "/share"
  # Definiert die statische IP Adresse
  dockermachine.vm.network "private_network", ip: "192.168.1.10"
  # Richtet das NAT auf den Localhost von Port 80 zu Port 80
  dockermachine.vm.network "forwarded_port", guest: 80, host: 80, host_ip: "127.0.0.1"
  # Führt nach der VM Installation das unten definierte Shellscript aus
  dockermachine.vm.provision "shell", path: "installer/installer.sh"
  end
  config.vm.box = "ubuntu/xenial64"
end
```
---
## Dockerdateien
> [&uarr; *Zum Inhaltsverzeichnis*](##Inhaltsverzeichnis)
```
# Aktualisiert alle neuen Repositories
sudo apt-get update -y
# Installiert alle benötigten Tools für die Docker mitsamt Docker und Docker Compose Installationen
sudo apt-get install  curl  apt-transport-https ca-certificates software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update -y
sudo apt-get install docker-ce -y
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# Macht den Ordner für alle ausführbar
sudo chmod +x /usr/local/bin/docker-compose
# Wechselt ins Verzeichnis /share
cd /share
# Führt das .YML File aus, welches sich auf dem Ordner /share befindet
sudo docker-compose up
```
---
## YML Datei
Hier geben Wir die Konfiguration für die MySQL-Datenbank mit.
> [&uarr; *Zum Inhaltsverzeichnis*](##Inhaltsverzeichnis)
```
# Definition des Web Containers
web:
  # Definert von vo Docker das Image bzw. COntainer starten soll (Dockerfile)
  build: .
  # Gibt den Port 80 auf 80 frei
  ports:
    - "80:80"
  #Macht eine Verlinkung (für die Kommunikation) auf den COntainer db
  links:
    - db
  # Definiert einen Shared folder von ./ zu /var/www/html/
  volumes:
    - ./:/var/www/html/
db:
  # Holt den MYSQL Container mit der Version 5.7
  image: "mysql:5.7"
  # Aktiviert die Neustart Option
  restart: always
  volumes:
  # Definiert einen Shared folder von ./ zu /var/www/html/
    - ./_mysql:/var/lib/mysql
  # Definiert die Appikationspezifischen Variablen
  environment:
    # Setzt das Root Kennwort auf root
    MYSQL_ROOT_PASSWORD: root
    # Definiert den User admin
    MYSQL_USER: admin
    # Definiert das Passwort test für den User admin
    MYSQL_PASSWORD: test
    # Definiert die Datenbank database
    MYSQL_DATABASE: database
  ports:
    # Gibt den Port 80 auf 80 frei
    - "3306:3306"
```
---
## Installation Apache & MySQL
> [&uarr; *Zum Inhaltsverzeichnis*](##Inhaltsverzeichnis)
```
# Definiert den PHP Container mit Apache und installiert alle benötigten PHP-Module
FROM php:5.6-apache
RUN a2enmod rewrite
RUN apt-get update && apt-get install -y libpng-dev libjpeg-dev libpq-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
    && docker-php-ext-install gd mbstring pdo pdo_mysql pdo_pgsql
# Definiert das Workdirectory des Webservers
WORKDIR /var/www/html
```
---
## PHP Ausgabe
> [&uarr; *Zum Inhaltsverzeichnis*](##Inhaltsverzeichnis)

Anzeige der Webseite, ob die Verbindung erfolgreich war oder fehlgeschlagen ist.
```
<?php
echo "<html><h1>DB Verbindung</h1>";
try {
    $db = new PDO('mysql:host=db;dbname=database;charset=utf8mb4', 'admin', 'test');
    echo "Datenbank Verbindung war erfolgreich!";
} catch(PDOException $e) {
    echo "Datenbank Verbindung fehlgeschlagen!: " . $e->getMessage();
}
```