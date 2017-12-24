# TP-Docker-Sofiane => Nginx PHP MySQL

Docker exécutant Nginx, PHP-FPM et MySQL.

## Aperçu

1. [Installer les prérequis](#installer-prerequis)

    Avant d'installer le projet, assurez-vous que les conditions préalables suivantes ont été remplies.	

2. [Cloner le projet](#cloner-le-projet)

    Vous devez télécharger le code depuis son dépôt sur GitHub.

3. [Mise en place de l'environnement](#mise-en-place) [Facultatif]

    Vous pouvez placer votre application dans le dossier /web/public/.

4. [Exécuter l'application](#executer-application)

    À ce stade, vous aurez tous les éléments du projet en place.

5. [Utiliser les commandes Docker](#utiliser-commandes-docker)

    Lors de l'éxecution, vous pouvez utiliser les commandes du docker pour effectuer des opérations récurrentes.

___

## Installer les prérequis

Toutes les conditions requises doivent être disponibles pour votre distribution. Les plus importants sont :

* [Git](https://git-scm.com/downloads)
* [Docker](https://docs.docker.com/engine/installation/)
* [Docker Compose](https://docs.docker.com/compose/install/)

Vérifiez si `docker-compose` est déjà installé en entrant la commande suivante : 

```sh
which docker-compose
```

Vérifier la comptatibilité de Docker Compose :

 - [La référence de la version 2 du fichier Compose](https://docs.docker.com/compose/compose-file/)

---
 
### Images à utiliser

* [Nginx](https://hub.docker.com/_/nginx/)
* [MySQL (Créer à partir de centos)](https://hub.docker.com/_/centos/)
* [PHP-FPM](https://hub.docker.com/_/php/)

Vous devez faire attention lors de l'installation de serveurs Web tiers tels que MySQL ou Nginx

Ce projet utilise les ports suivants :

| Server     | Port |
|------------|------|
| MySQL      | 8989 |
| PHP-fpm    | 9000 |
| Nginx      | 8080 |

---

## Cloner le projet

Pour installer [Git](https://git-scm.com/book/fr/v2/Démarrage-rapide-Installation-de-Git), téléchargez-le et installez-le en suivant les instructions : 

```sh
git clone https://github.com/Sbenziane/TP-Docker-Sofiane.git
```

Allez dans le répertoire du projet : 

```sh
cd ./TP-Docker-Sofiane/
```

### Arbre du projet 

```sh
.
├── README.md
├── data
│   └── db
│	├── dumps
│       └── mysql
├── docker-compose.yml
├── logs
│   ├── nginx-access.log
│   └── nginx-error.log
├── nginx
│   └── default.conf
├── php
│   └── Dockerfile
├── mysql
│   ├── docker-entrypoint.sh
│   └── Dockerfile
└── web
    └── public
        └── index.php
```

---

## Configurer Nginx

1. Configure Nginx

    Éditer le fichier default.conf dans le dossier nginx :

```sh
server {

    # Set the port to listen on and the server name
    listen 80;

    # Set the document root of the project
    root /usr/share/nginx/html;

    # Set the directory index files
    index index.php index.html index.html;

    # Specify the default character set
    charset utf-8;

    # Setup the default location configuration
    location / {
        try_files $uri $uri/ /index.php;
    }

    # Specify the details of favicon.ico
    location = /favicon.ico { access_log off; log_not_found off; }

    # Specify the details of robots.txt
    location = /robots.txt  { access_log off; log_not_found off; }

    # Specify the logging configuration
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    sendfile off;

    client_max_body_size 100m;

    # Specify what happens when PHP files are requested
    location ~ .php$ {
        fastcgi_split_path_info ^(.+.php)(/.+)$;
        fastcgi_pass phpfpm:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param APPLICATION_ENV development;
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
    }

    # Specify what happens what .ht files are requested
    location ~ /.ht {
        deny all;
    }
}
```

---

## Exécuter l'application

1. Copiez votre application dans le répertoire /web/public.

2. Démarrez l'environnement :

    ```sh
    docker-compose up -d
    ```

    **Veuillez patienter, cela peut prendre plusieurs minutes...**

    ```sh
    docker-compose logs -f # Suit la sortie du journal
    ```

3. Ouvrez le dans votre navigateur :

    * [http://localhost:8080](http://localhost:8080/) si vous disposez de windows 10 pro
	* [http://192.168.99.100:8080](http://192.168.99.100:8080/) si vous utilisez windows 10 familial 

4. Arrêter et effacer les services

    ```sh
    docker-compose down -v
    ```

---

## Utiliser les commandes Docker

### Vérification des extensions PHP installées.

```sh
docker-compose exec phpfpm php -m
```

### Gestion de la base de données

** Par défaut sur l'environnement, l'utilisateur est 'root' et le mot de passe 'admin'. **


#### Accès au shell MySQL

```sh
docker-compose exec mysql bash
```

et 

```sh
mysql -u "$MYSQL_ROOT_USER" -p "$MYSQL_ROOT_PASSWORD"
```

#### Sauvegarde

```sh
mkdir -p data/db/dumps
```

```sh
docker exec $(docker-compose ps -q mysql) mysqldump --all-databases -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/db.sql"
```

ou

```sh
docker exec $(docker-compose ps -q mysql) mysqldump test -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/test.sql"
```

#### Restaurer la base de données

```sh
docker exec -i $(docker-compose ps -q mysql) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/db.sql"
```

---
