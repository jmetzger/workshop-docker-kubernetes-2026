# Exercise network 

## Prerequisites 

  * Wordpress Stack läuft

## Exercise - Teil 1

```
# Wir wollen das Netzwerk sehen
docker network ls
docker network inspect bridge # oder kuerzer: docker inspect bridge
docker network inspect wp_default
```

```
cd
cd wp
```

```
nano docker-compose.yml
```

```
services:
    database:
        image: mysql:5.7
        volumes:
            - database_data:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: wordpress:latest
        depends_on:
            - database
        ports:
            - 8080:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: database:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - wordpress_plugins:/var/www/html/wp-content/plugins
            - wordpress_themes:/var/www/html/wp-content/themes
            - wordpress_uploads:/var/www/html/wp-content/uploads
## Diese 3 Zeilen 
    busybox:
        image: busybox
        command: sleep infinity  

volumes:
    database_data:
    wordpress_plugins:
    wordpress_themes:
    wordpress_uploads:
```

```
docker compose up -d
docker compose exec -it busybox sh
```

```
# in der shell
# +++ -> ip addresse notieren 
ping -c4 wordpress
ping -c4 database
```


## Exercise Teil 2: aus anderem Projekt 

```
cd
mkdir -p appmaster
cd appmaster
```

```
nano docker-compose.yml
```

```
## Diese 3 Zeilen
services:
    busyboxapp:
        image: busybox
        command: sleep infinity
```

```
docker compose up -d
```

```
docker compose exec busyboxapp sh
```

```
ping -c 4 <ip-eines-services-aus-dem-wp-project>
# z.B.
# ping -c 4 172.18.0.3
```
