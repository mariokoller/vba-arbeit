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
Vagrant.configure("2") do |config|
config.vm.define "dockermachine" do |dockermachine|
  dockermachine.vm.box = "ubuntu/xenial64"
  dockermachine.vm.provider "virtualbox" do |vb|
  vb.memory = "2048"
  vb.cpus = "2"
  end
  config.vm.synced_folder "share", "/share"
  dockermachine.vm.network "private_network", ip: "192.168.1.10"
  dockermachine.vm.network "forwarded_port", guest: 80, host: 80, host_ip: "127.0.0.1"
  dockermachine.vm.provision "shell", path: "installer/installer.sh"
  end
  config.vm.box = "ubuntu/xenial64"
end
```
---
## Dockerdateien
> [&uarr; *Zum Inhaltsverzeichnis*](##Inhaltsverzeichnis)
```
sudo apt-get update -y
sudo apt-get install  curl  apt-transport-https ca-certificates software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update -y
sudo apt-get install docker-ce -y
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
cd /share
sudo docker-compose up
```
---
## YML Datei
Hier geben Wir die Konfiguration für die MySQL-Datenbank mit.
> [&uarr; *Zum Inhaltsverzeichnis*](##Inhaltsverzeichnis)
```
web:
  build: .
  ports:
    - "80:80"
  links:
    - db
  volumes:
    - ./:/var/www/html/
db:
  image: "mysql:5.7"
  restart: always
  volumes:
    - ./_mysql:/var/lib/mysql
  environment:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_USER: admin
    MYSQL_PASSWORD: test
    MYSQL_DATABASE: database
  ports:
    - "3306:3306"
```
---
## Installation Apache & MySQL
> [&uarr; *Zum Inhaltsverzeichnis*](##Inhaltsverzeichnis)
```
FROM php:5.6-apache
RUN a2enmod rewrite
RUN apt-get update && apt-get install -y libpng-dev libjpeg-dev libpq-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
    && docker-php-ext-install gd mbstring pdo pdo_mysql pdo_pgsql
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