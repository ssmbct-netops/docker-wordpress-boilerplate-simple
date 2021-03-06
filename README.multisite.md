# Multi-Site installation

This boilerplate will support you installing multiple wordpress sites in one stack, why still
 
 - splitting them entirely in own FPM process and isolating them in process and filesystem
 - still not overcomplicating anything since we have Traefik to deal with the routing right away
 - reusing the same DB daemon - using own databases though

 ## Getting started.

 create a new override configuration for your second site 

`vim config/docker-compose-site2.yml`

```yaml
version: "3.3"

services:
  wordpress-site2:
    depends_on:
      - db
    image: eugenmayer/wordpress
    volumes:
      - ./data/site2/wordpress_files:/var/www/html
      - ./data/site2/backups:/backups
    restart: always
    labels:
      traefik.http.routers.wordpress_site2.rule: "Host(`${WORDPRESS_SITE2_DOMAIN}`)"
      traefik.http.routers.wordpress_site2.tls: "true"
      traefik.http.routers.wordpress_site2.tls.certresolver: "default"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ${DB_PW}
      WORDPRESS_DB_NAME: site2
 ```

Now connect to your database to create a new database

```bash
docker-compose exec db bash
# use your mysql root password
mysql -p
# use your DB password here ( not root db password)
grant all on site2.* to wordpress identified by '<>'
flush privileges
```

Now add the domain to your `.env` file and also add our override as i explained in `README.md`

`vim .env`

```
COMPOSE_FILE=docker-compose.yml:docker-compose-traefik-http.yaml:config/docker-compose-site2.yml
WORDPRESS_DOMAIN=firssite.com
WORDPRESS_SITE_2_DOMAIN=secondsite.com
```

You might want to change `docker-compose-traefik-http.yml` to `docker-compose-traefik-dns.yml` if you used `DNS-01`

## Adding a third side

If you plan adding a third site even - simple enough.

Repeat the steps above creating `config/docker-compose-site3.yml`

 - just call the service `wordpress-site3`,
 - the volume folder `site3` 
 - change the label path from `traefik.http.routers.wordpress_site2..` to `traefik.http.routers.wordpress_site3..`
 - change the `WORDPRESS_DB_NAME: site2` to `WORDPRESS_DB_NAME: site3`
 - create the database `site3` with the credentials
 - and the env file `WORDPRESS_SITE_3_DOMAIN` ..

you get it. Then put this in your `.env` file

```
COMPOSE_FILE=docker-compose.yml:docker-compose-traefik-http.yaml:config/docker-compose-site2.yml:config/docker-compose-site3.yml
WORDPRESS_DOMAIN=firssite.com
WORDPRESS_SITE_2_DOMAIN=secondsite.com
WORDPRESS_SITE_3_DOMAIN=thirdsite.com
```
