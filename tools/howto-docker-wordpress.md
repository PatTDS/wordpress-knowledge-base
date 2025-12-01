---
title: Docker WordPress Development Setup
category: tools
type: howto
updated: 2025-12-01
version: 1.0.0
tags:
  - docker
  - docker-compose
  - development-environment
  - containerization
sources:
  - https://www.docker.com/blog/how-to-dockerize-wordpress/
  - https://collabnix.com/best-practices-for-dockerizing-wordpress-plugins-and-themes/
  - https://runcloud.io/blog/install-wordpress-on-docker
  - https://wpengine.com/resources/containers-clusters-wordpress/
---

# Docker WordPress Development Setup

Docker provides consistent, isolated WordPress development environments that eliminate "works on my machine" issues.

## Prerequisites

- Docker Desktop installed
- Docker Compose installed
- Basic command line knowledge

## Basic Setup

### 1. Create Project Structure

```bash
mkdir wordpress-project
cd wordpress-project
```

### 2. Create docker-compose.yml

```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:6.7-php8.2
    container_name: wp_site
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress_password
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DEBUG: 1
    volumes:
      - wordpress_data:/var/www/html
      - ./wp-content/themes:/var/www/html/wp-content/themes
      - ./wp-content/plugins:/var/www/html/wp-content/plugins
    depends_on:
      - db
    networks:
      - wp_network

  db:
    image: mysql:8.0
    container_name: wp_db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress_password
      MYSQL_ROOT_PASSWORD: root_password
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wp_network

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: wp_pma
    restart: unless-stopped
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: root_password
    depends_on:
      - db
    networks:
      - wp_network

volumes:
  wordpress_data:
  db_data:

networks:
  wp_network:
    driver: bridge
```

### 3. Start Environment

```bash
docker-compose up -d
```

Access WordPress at `http://localhost:8080`
Access phpMyAdmin at `http://localhost:8081`

## Advanced Configuration

### With WP-CLI

```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:6.7-php8.2
    container_name: wp_site
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress_password
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html
      - ./wp-content/themes:/var/www/html/wp-content/themes
      - ./wp-content/plugins:/var/www/html/wp-content/plugins
      - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
    depends_on:
      - db
    networks:
      - wp_network

  db:
    image: mysql:8.0
    container_name: wp_db
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress_password
      MYSQL_ROOT_PASSWORD: root_password
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wp_network

  wpcli:
    image: wordpress:cli
    container_name: wp_cli
    volumes:
      - wordpress_data:/var/www/html
      - ./wp-content/themes:/var/www/html/wp-content/themes
      - ./wp-content/plugins:/var/www/html/wp-content/plugins
    depends_on:
      - wordpress
      - db
    networks:
      - wp_network
    user: "33:33"
    entrypoint: wp
    command: "--info"

volumes:
  wordpress_data:
  db_data:

networks:
  wp_network:
    driver: bridge
```

### Custom PHP Configuration

Create `uploads.ini`:

```ini
file_uploads = On
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300
memory_limit = 256M
max_input_vars = 3000
```

## Running WP-CLI Commands

```bash
# Via docker-compose
docker-compose run --rm wpcli plugin list

# Direct execution
docker exec -it wp_site wp plugin list --allow-root

# Create alias
alias wp="docker-compose run --rm wpcli"
wp plugin list
```

## Production Configuration

```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:6.7-php8.2-fpm
    container_name: wp_site
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: ${DB_USER}
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
      WORDPRESS_DB_NAME: ${DB_NAME}
    volumes:
      - wordpress_data:/var/www/html
      - ./wp-content:/var/www/html/wp-content
    depends_on:
      - db
    networks:
      - wp_network
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: mysql:8.0
    container_name: wp_db
    restart: always
    environment:
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wp_network

  nginx:
    image: nginx:alpine
    container_name: wp_nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - wordpress_data:/var/www/html:ro
    depends_on:
      - wordpress
    networks:
      - wp_network

  redis:
    image: redis:alpine
    container_name: wp_redis
    restart: always
    networks:
      - wp_network

volumes:
  wordpress_data:
  db_data:

networks:
  wp_network:
    driver: bridge
```

### Environment Variables

Create `.env`:

```bash
# Database
DB_NAME=wordpress
DB_USER=wordpress
DB_PASSWORD=secure_password_here
DB_ROOT_PASSWORD=very_secure_root_password

# WordPress
WORDPRESS_TABLE_PREFIX=wp_
```

## With Traefik (HTTPS)

```yaml
version: '3.8'

services:
  traefik:
    image: traefik:v2.10
    container_name: traefik
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:read_only
      - ./traefik/traefik.yml:/traefik.yml:ro
      - ./traefik/acme.json:/acme.json
    networks:
      - wp_network

  wordpress:
    image: wordpress:6.7-php8.2
    container_name: wp_site
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.wordpress.rule=Host(`example.com`)"
      - "traefik.http.routers.wordpress.entrypoints=websecure"
      - "traefik.http.routers.wordpress.tls.certresolver=letsencrypt"
    networks:
      - wp_network

  db:
    image: mysql:8.0
    container_name: wp_db
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wp_network

volumes:
  wordpress_data:
  db_data:

networks:
  wp_network:
    driver: bridge
```

## Common Operations

### Start/Stop

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# Stop and remove volumes (CAUTION: deletes data)
docker-compose down -v

# Restart specific service
docker-compose restart wordpress
```

### Logs

```bash
# View all logs
docker-compose logs

# Follow WordPress logs
docker-compose logs -f wordpress

# View last 100 lines
docker-compose logs --tail=100 wordpress
```

### Database Backup/Restore

```bash
# Backup
docker exec wp_db mysqldump -u wordpress -pwordpress_password wordpress > backup.sql

# Restore
docker exec -i wp_db mysql -u wordpress -pwordpress_password wordpress < backup.sql
```

### File Sync Issues

```bash
# Manual sync to container
docker cp ./wp-content/themes/my-theme/. wp_site:/var/www/html/wp-content/themes/my-theme/

# Fix permissions
docker exec wp_site chown -R www-data:www-data /var/www/html/wp-content
```

### Shell Access

```bash
# WordPress container
docker exec -it wp_site bash

# MySQL container
docker exec -it wp_db mysql -u root -p
```

## Security Best Practices

1. **Never commit .env files** - Add to .gitignore
2. **Use Docker secrets** for sensitive data in Swarm/production
3. **Mount docker.sock read-only** when using Traefik
4. **Update images regularly** - `docker-compose pull`
5. **Use specific version tags** - Not `latest` in production
6. **Limit container resources**:

```yaml
services:
  wordpress:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
```

## Monitoring

```bash
# Container stats
docker stats

# Check container health
docker ps

# Inspect container
docker inspect wp_site
```

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs wordpress

# Check if port is in use
netstat -tulpn | grep 8080

# Rebuild containers
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

### Database Connection Error

```bash
# Verify database container is running
docker-compose ps db

# Test database connection
docker exec wp_db mysql -u wordpress -pwordpress_password -e "SHOW DATABASES;"

# Check environment variables
docker exec wp_site env | grep WORDPRESS_DB
```

### Permission Issues

```bash
# Fix ownership
docker exec wp_site chown -R www-data:www-data /var/www/html

# Check current permissions
docker exec wp_site ls -la /var/www/html/wp-content
```

## Related Documents

- @ref-wp-cli-commands.md
- @howto-development-environment.md
- @howto-deployment-automation.md
