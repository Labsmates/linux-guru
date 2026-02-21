# 🐳 Docker - Guide Complet des Commandes

Maîtriser Docker de A à Z : images, containers, volumes, networks, et Docker Compose.

---

## 📦 Images Docker

### docker pull - Download Image

```bash
# Pull dernière version (latest)
docker pull nginx

# Version spécifique
docker pull nginx:1.25

# Depuis registry custom
docker pull myregistry.com:5000/nginx

# Toutes les tags d'une image
docker pull -a nginx
```

### docker images - List Images

```bash
# Lister toutes les images
docker images

# Lister avec plus de détails
docker images --no-trunc

# Filtrer par nom
docker images nginx

# Afficher seulement IDs
docker images -q

# Afficher taille totale
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

**Comprendre docker images :**
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    a6bd71f48f68   2 weeks ago    187MB
mysql         8.0       3218b38490ce   3 weeks ago    516MB
```
- **REPOSITORY** : Nom de l'image
- **TAG** : Version (latest = dernière)
- **IMAGE ID** : Identifiant unique
- **CREATED** : Date de création
- **SIZE** : Taille de l'image

### docker build - Build Image

```bash
# Build depuis Dockerfile (dossier courant)
docker build -t myapp:1.0 .

# Spécifier Dockerfile
docker build -t myapp:1.0 -f Dockerfile.prod .

# Build avec build args
docker build -t myapp:1.0 --build-arg VERSION=1.0 .

# No cache (rebuild from scratch)
docker build --no-cache -t myapp:1.0 .

# Cibler stage spécifique (multi-stage build)
docker build --target production -t myapp:1.0 .

# Platform spécifique
docker build --platform linux/amd64 -t myapp:1.0 .
```

**Exemple Dockerfile :**
```dockerfile
# Dockerfile
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy app files
COPY . .

# Expose port
EXPOSE 3000

# User non-root (sécurité)
USER node

# Start app
CMD ["node", "server.js"]
```

### docker tag - Tag Image

```bash
# Tagger image
docker tag nginx:latest myregistry.com/nginx:v1

# Tagger avec nouveau nom
docker tag myapp:1.0 myapp:latest

# Tagger pour Docker Hub
docker tag myapp:1.0 username/myapp:1.0
```

### docker push - Upload Image

```bash
# Login registry
docker login

# Push image
docker push username/myapp:1.0

# Push vers registry custom
docker push myregistry.com/myapp:1.0
```

### docker rmi - Remove Image

```bash
# Supprimer image
docker rmi nginx

# Supprimer par ID
docker rmi a6bd71f48f68

# Force delete (même si container utilise l'image)
docker rmi -f nginx

# Supprimer toutes les images dangling (non taguées)
docker image prune

# Supprimer toutes les images inutilisées
docker image prune -a

# Supprimer images multiples
docker rmi $(docker images -q)
```

---

## 🚢 Containers Docker

### docker run - Create and Start Container

```bash
# Run basique
docker run nginx

# Détaché (background)
docker run -d nginx

# Avec nom
docker run -d --name webserver nginx

# Port mapping (host:container)
docker run -d -p 8080:80 nginx
# Accès: http://localhost:8080

# Publish all exposed ports
docker run -d -P nginx

# Volume mount
docker run -d -v /host/path:/container/path nginx

# Named volume
docker run -d -v mydata:/data nginx

# Read-only volume
docker run -d -v /host/path:/container/path:ro nginx

# Environment variables
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Environment file
docker run -d --env-file .env mysql

# Interactive terminal
docker run -it ubuntu /bin/bash

# Auto-remove after stop
docker run --rm -it ubuntu /bin/bash

# Restart policy
docker run -d --restart=always nginx
docker run -d --restart=unless-stopped nginx
docker run -d --restart=on-failure:3 nginx

# Resource limits
docker run -d --memory="512m" --cpus="1.5" nginx

# Network
docker run -d --network=mynetwork nginx

# Hostname
docker run -d --hostname=webserver nginx

# DNS
docker run -d --dns=8.8.8.8 nginx

# User
docker run -d --user 1000:1000 nginx

# Working directory
docker run -d -w /app nginx
```

**Options communes expliquées :**
```
-d              : Detached (arrière-plan)
-it             : Interactive + TTY (terminal)
--rm            : Auto-remove après arrêt
--name          : Nom du container
-p host:cont    : Port mapping
-v host:cont    : Volume mount
-e VAR=value    : Variable d'environnement
--network       : Réseau custom
--restart       : Politique de redémarrage
--memory        : Limite RAM
--cpus          : Limite CPU
```

### docker ps - List Containers

```bash
# Containers en cours
docker ps

# Tous les containers (y compris stoppés)
docker ps -a

# Seulement IDs
docker ps -q

# Derniers 5 containers créés
docker ps -n 5

# Format custom
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Filtrer par status
docker ps -f "status=running"
docker ps -f "status=exited"

# Filtrer par nom
docker ps -f "name=web"

# Taille des containers
docker ps -s
```

### docker exec - Execute Command in Container

```bash
# Exécuter commande
docker exec webserver ls /var/www/html

# Shell interactif
docker exec -it webserver /bin/bash

# Exécuter en tant qu'utilisateur
docker exec -u root -it webserver /bin/bash

# Exécuter dans working directory spécifique
docker exec -w /app -it webserver ls

# Variables d'environnement
docker exec -e VAR=value webserver printenv VAR
```

**Exemples pratiques :**
```bash
# Accéder shell MySQL
docker exec -it mysql mysql -uroot -p

# Voir logs Nginx
docker exec webserver cat /var/log/nginx/access.log

# Copier fichiers
docker exec webserver cat /etc/nginx/nginx.conf > nginx.conf

# Installer package (temporaire)
docker exec -it webserver apt-get update && apt-get install -y vim
```

### docker logs - View Logs

```bash
# Logs complets
docker logs webserver

# Follow (temps réel)
docker logs -f webserver

# Dernières 100 lignes
docker logs --tail 100 webserver

# Depuis timestamp
docker logs --since 2024-02-21T10:00:00 webserver

# Jusqu'à timestamp
docker logs --until 2024-02-21T12:00:00 webserver

# Timestamps
docker logs -t webserver

# Dernières 10 minutes
docker logs --since 10m webserver
```

### docker start / stop / restart

```bash
# Démarrer container
docker start webserver

# Arrêter container
docker stop webserver

# Arrêt forcé (SIGKILL)
docker kill webserver

# Redémarrer
docker restart webserver

# Démarrer tous les containers stoppés
docker start $(docker ps -a -q -f status=exited)

# Arrêter tous les containers
docker stop $(docker ps -q)
```

### docker rm - Remove Container

```bash
# Supprimer container (doit être arrêté)
docker rm webserver

# Force remove (même si running)
docker rm -f webserver

# Supprimer multiple containers
docker rm web1 web2 web3

# Supprimer tous containers stoppés
docker container prune

# Supprimer tous containers (force)
docker rm -f $(docker ps -aq)
```

### docker inspect - Detailed Info

```bash
# Inspect container
docker inspect webserver

# Format JSON spécifique
docker inspect --format='{{.NetworkSettings.IPAddress}}' webserver

# Récupérer IP
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' webserver

# Récupérer ports
docker inspect -f '{{range $p, $conf := .NetworkSettings.Ports}}{{$p}} -> {{(index $conf 0).HostPort}}{{end}}' webserver

# Volumes montés
docker inspect -f '{{json .Mounts}}' webserver | jq

# Env variables
docker inspect -f '{{.Config.Env}}' webserver
```

### docker cp - Copy Files

```bash
# Copier depuis container vers host
docker cp webserver:/etc/nginx/nginx.conf ./nginx.conf

# Copier depuis host vers container
docker cp ./index.html webserver:/usr/share/nginx/html/

# Copier dossier
docker cp webserver:/var/log/ ./logs/
```

### docker stats - Resource Usage

```bash
# Stats de tous les containers
docker stats

# Container spécifique
docker stats webserver

# No stream (snapshot)
docker stats --no-stream

# Format custom
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

---

## 💾 Volumes Docker

### docker volume create

```bash
# Créer volume
docker volume create mydata

# Avec driver spécifique
docker volume create --driver local mydata

# Avec options
docker volume create --opt type=nfs --opt device=:/path mydata
```

### docker volume ls

```bash
# Lister volumes
docker volume ls

# Filtrer dangling volumes (non utilisés)
docker volume ls -f dangling=true

# Format
docker volume ls --format "{{.Name}}: {{.Driver}}"
```

### docker volume inspect

```bash
# Inspect volume
docker volume inspect mydata

# Mountpoint (emplacement sur host)
docker volume inspect -f '{{.Mountpoint}}' mydata
```

### docker volume rm

```bash
# Supprimer volume
docker volume rm mydata

# Supprimer tous volumes non utilisés
docker volume prune

# Force prune
docker volume prune -f
```

**Utilisation volumes :**
```bash
# Named volume
docker run -d -v mydata:/var/lib/mysql mysql

# Bind mount
docker run -d -v /host/data:/container/data nginx

# Read-only
docker run -d -v mydata:/data:ro nginx

# tmpfs (RAM, non persistent)
docker run -d --tmpfs /tmp nginx
```

---

## 🌐 Networks Docker

### docker network create

```bash
# Créer network bridge (défaut)
docker network create mynetwork

# Network avec subnet custom
docker network create --subnet=172.18.0.0/16 mynetwork

# Avec gateway
docker network create --subnet=172.18.0.0/16 --gateway=172.18.0.1 mynetwork

# Network overlay (Swarm)
docker network create --driver overlay myoverlay
```

### docker network ls

```bash
# Lister networks
docker network ls

# Filtrer par driver
docker network ls --filter driver=bridge
```

### docker network inspect

```bash
# Inspect network
docker network inspect mynetwork

# Voir containers connectés
docker network inspect -f '{{range .Containers}}{{.Name}} {{end}}' mynetwork
```

### docker network connect / disconnect

```bash
# Connecter container à network
docker network connect mynetwork webserver

# Avec IP statique
docker network connect --ip 172.18.0.100 mynetwork webserver

# Déconnecter
docker network disconnect mynetwork webserver
```

### docker network rm

```bash
# Supprimer network
docker network rm mynetwork

# Supprimer tous networks non utilisés
docker network prune
```

**Networks types :**
```
bridge      : Réseau isolé sur host (défaut)
host        : Utilise réseau de l'host directement
none        : Pas de réseau
overlay     : Multi-host (Swarm)
macvlan     : Assigne MAC address au container
```

**Exemples pratiques :**
```bash
# App multi-tier
docker network create app-network
docker run -d --name db --network app-network mysql
docker run -d --name web --network app-network nginx
# web peut accéder à db via hostname "db"

# Isolation complète
docker run -d --network none nginx
```

---

## 🎼 Docker Compose

### docker-compose.yml Structure

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    environment:
      - NGINX_HOST=example.com
    networks:
      - frontend
    restart: unless-stopped

  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_PASSWORD=${DB_PASSWORD}
    networks:
      - frontend
      - backend
    volumes:
      - app-data:/data

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: myapp
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - backend
    restart: always

volumes:
  app-data:
  db-data:

networks:
  frontend:
  backend:
```

### docker-compose up

```bash
# Démarrer services
docker-compose up

# Détaché (background)
docker-compose up -d

# Rebuild images
docker-compose up --build

# Scale service
docker-compose up --scale app=3

# Spécifier fichier compose
docker-compose -f docker-compose.prod.yml up -d
```

### docker-compose down

```bash
# Arrêter et supprimer containers
docker-compose down

# Supprimer aussi volumes
docker-compose down -v

# Supprimer aussi images
docker-compose down --rmi all
```

### docker-compose ps

```bash
# Lister services
docker-compose ps

# Tous les containers (même stoppés)
docker-compose ps -a
```

### docker-compose logs

```bash
# Logs de tous les services
docker-compose logs

# Service spécifique
docker-compose logs web

# Follow
docker-compose logs -f

# Dernières 100 lignes
docker-compose logs --tail=100
```

### docker-compose exec

```bash
# Shell dans service
docker-compose exec web /bin/bash

# Commande dans service
docker-compose exec db mysql -uroot -p
```

### docker-compose build

```bash
# Build tous les services
docker-compose build

# Build service spécifique
docker-compose build app

# No cache
docker-compose build --no-cache
```

### Autres commandes docker-compose

```bash
# Démarrer services
docker-compose start

# Arrêter services
docker-compose stop

# Redémarrer services
docker-compose restart

# Pause services
docker-compose pause

# Unpause
docker-compose unpause

# Voir configuration
docker-compose config

# Valider syntaxe
docker-compose config -q

# Pull images
docker-compose pull

# Push images
docker-compose push
```

---

## 🔧 Dockerfile Best Practices

### Multi-stage Build

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

### Optimiser Layers

```dockerfile
# ❌ Mauvais (chaque RUN = nouvelle layer)
RUN apt-get update
RUN apt-get install -y nginx
RUN apt-get install -y curl

# ✅ Bon (une seule layer)
RUN apt-get update && \
    apt-get install -y \
      nginx \
      curl && \
    rm -rf /var/lib/apt/lists/*
```

### .dockerignore

```
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
.env
*.md
Dockerfile
docker-compose.yml
```

### ARG vs ENV

```dockerfile
# ARG : Build-time seulement
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}

# ENV : Build-time + Runtime
ENV NODE_ENV=production
ENV PORT=3000
```

### Security

```dockerfile
# Utiliser user non-root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Scanner vulnérabilités
# docker scan myimage:latest

# Ne pas stocker secrets dans image
# Utiliser --secret au build
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret > /tmp/config
```

---

## 🧹 Nettoyage et Maintenance

### docker system prune

```bash
# Nettoyer tout (containers, networks, images dangling)
docker system prune

# Inclure volumes
docker system prune --volumes

# Inclure toutes images non utilisées
docker system prune -a

# Force (pas de confirmation)
docker system prune -f
```

### docker system df

```bash
# Afficher espace disque utilisé
docker system df

# Détaillé
docker system df -v
```

**Commandes de nettoyage spécifiques :**
```bash
# Supprimer containers stoppés
docker container prune

# Supprimer images dangling
docker image prune

# Supprimer toutes images non utilisées
docker image prune -a

# Supprimer volumes non utilisés
docker volume prune

# Supprimer networks non utilisés
docker network prune
```

---

## 🐛 Troubleshooting

### Débugger container qui crash

```bash
# Voir logs
docker logs container_name

# Inspecter
docker inspect container_name

# Essayer de démarrer en interactif
docker run -it --entrypoint /bin/bash image_name

# Override CMD
docker run -it image_name /bin/sh

# Voir pourquoi container s'est arrêté
docker inspect --format='{{.State.ExitCode}}' container_name
docker inspect --format='{{.State.Error}}' container_name
```

### Problèmes réseau

```bash
# Voir IP container
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container

# Tester connectivité
docker exec container ping google.com
docker exec container curl http://api:3000

# DNS resolution
docker exec container nslookup database
docker exec container cat /etc/resolv.conf

# Ports
docker port container_name
```

### Performance issues

```bash
# Stats temps réel
docker stats

# Voir processus dans container
docker top container_name

# Events Docker
docker events

# Filtrer events
docker events --filter container=webserver
```

---

## 📊 Exemples Complets

### Stack WordPress

```yaml
# docker-compose.yml
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: secret
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootsecret
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: secret
    volumes:
      - db-data:/var/lib/mysql

volumes:
  wordpress-data:
  db-data:
```

### Stack LEMP (Linux, Nginx, MySQL, PHP)

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./html:/var/www/html
    depends_on:
      - php

  php:
    image: php:8.2-fpm-alpine
    volumes:
      - ./html:/var/www/html
    environment:
      - MYSQL_HOST=mysql
      - MYSQL_DATABASE=app
      - MYSQL_USER=user
      - MYSQL_PASSWORD=pass

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: app
      MYSQL_USER: user
      MYSQL_PASSWORD: pass
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

---

## 🔑 Astuces et Commandes Utiles

```bash
# Entrer dans container avec shell root (même si USER défini)
docker exec -u 0 -it container /bin/bash

# Copier image d'un host à un autre (sans registry)
docker save myimage:latest | gzip > myimage.tar.gz
# Sur l'autre host:
docker load < myimage.tar.gz

# Voir historique d'une image
docker history nginx

# Export container comme image
docker export container_name > container.tar
docker import container.tar myimage:latest

# Commit container vers nouvelle image
docker commit container_name newimage:tag

# Resource cleanup automatique
# Ajouter à crontab:
0 3 * * * docker system prune -af --volumes

# Lister images par taille
docker images --format "{{.Size}}\t{{.Repository}}:{{.Tag}}" | sort -hr

# Trouver containers utilisant plus de RAM
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}" | sort -k2 -hr

# Variables d'environnement depuis fichier
docker run --env-file .env myimage

# Healthcheck
docker run -d \
  --health-cmd='curl -f http://localhost/ || exit 1' \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  nginx

# Restart seulement si unhealthy
docker run -d --restart=on-failure:5 nginx
```

---

**🎓 Prochaine étape : [OpenShift & Kubernetes](./openshift-kubernetes.md) →**
