# php8_mysql

Docker image based on `php:8.5-fpm` with MySQL/MariaDB extensions installed:

- `mysqli`
- `pdo`
- `pdo_mysql`

The image is built for `linux/amd64` and `linux/arm64`.

## Usage

```bash
docker pull ghcr.io/wojtekadams/php8_mysql:latest
```

PHP-FPM listens on port `9000`, so the image needs a web server (e.g. nginx) in front of it.

### Docker Compose example

```yaml
services:
  php:
    image: ghcr.io/wojtekadams/php8_mysql:latest
    volumes:
      - ./src:/var/www/html

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./src:/var/www/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: app
```

Minimal `nginx.conf`:

```nginx
server {
    listen 80;
    root /var/www/html;
    index index.php;

    location ~ \.php$ {
        fastcgi_pass php:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

From PHP, connect to the database using the `db` host:

```php
$pdo = new PDO('mysql:host=db;dbname=app', 'root', 'secret');
```

## Tags

| Tag | When it is created |
|-----|--------------------|
| `latest`, `main` | every push to `main` |
| `sha-<commit>` | every push, points to a specific commit |
| `1.2.0`, `1.2` | pushing the tag `v1.2.0` |

To release a version:

```bash
git tag v1.0.0
git push origin v1.0.0
```

## Building locally

```bash
docker build -t php8_mysql .
```

## CI

The image is built and published to GitHub Container Registry by the workflow [.github/workflows/docker-publish.yml](.github/workflows/docker-publish.yml). Pull requests are only built, not published. Each image's description (`org.opencontainers.image.description`) lists the PHP version and the versions of the installed extensions, e.g. `PHP 8.5.0 (fpm) | mysqli 8.5.0 | pdo 8.5.0 | pdo_mysql 8.5.0 | mysqlnd 8.5.0`. After each publish, only the 2 newest images are kept in the registry; older ones are deleted.
