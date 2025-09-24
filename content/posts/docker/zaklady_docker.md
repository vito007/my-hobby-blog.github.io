---
title: "Docker - Kompletný sprievodca pre začiatočníkov"
description: "Komplexný návod na Docker pre začiatočníkov s praktickými príkladmi. Naučte sa základy kontajnerizácie, networking, volumes a Docker Compose."
date: 2025-01-15
draft: false
tags: ["docker", "kontajnery", "devops"]
categories: ["Návody","Technológie"]
author: "Váš Author"
slug: "docker-zaklady-sprievodca"
toc: true
---

> **Prečo Docker?** Docker revolutionalizoval spôsob, ako vyvíjame a deployujeme aplikácie. Umožňuje zabaliť aplikáciu so všetkými závislostmi do prenosného kontajnera.

## 1. Úvod do Dockeru

### Čo je Docker?
Docker je platforma na kontajnerizáciu aplikácií. Umožňuje zabaliť aplikáciu a všetky jej závislosti do prenosného kontajnera, ktorý beží rovnako na každom systéme.

### Základné pojmy:
- **Image (obraz)** - šablóna na vytvorenie kontajnera
- **Container (kontajner)** - bežiaca inštancia obrazu  
- **Dockerfile** - súbor s inštrukciami na vytvorenie obrazu
- **Registry** - úložisko obrazov (napr. Docker Hub)

---

## 2. Správa Images (obrazov)

### Teoretický úvod
Image je read-only šablóna, z ktorej sa vytvárajú kontajnery. Skladá sa z vrstiev (layers), čo umožňuje efektívne zdieľanie a caching.

### Základné príkazy:
```bash
# Stiahnutie obrazu
docker pull ubuntu:20.04

# Zoznam lokálnych obrazov
docker images

# Vytvorenie obrazu z Dockerfile
docker build -t moja-app:1.0 .

# Odstránenie obrazu
docker rmi ubuntu:20.04

# Zobrazenie histórie obrazu
docker history ubuntu:20.04
```

### Praktický príklad - vytvorenie vlastného obrazu:
```dockerfile
# Dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3 python3-pip
COPY app.py /app/
WORKDIR /app
CMD ["python3", "app.py"]
```

```bash
# Vytvorenie obrazu
docker build -t python-app:1.0 .
```

---

## 3. Správa Containers (kontajnerov)

> **Pozor:** Kontajnery sú dočasné. Po ich odstránení sa stratia všetky zmeny, ktoré neboli uložené do volumes.

### Teoretický úvod
Kontajner je bežiaca inštancia obrazu. Je to izolované prostredie s vlastným filesystémom, procesmi a sieťovým rozhraním.

### Základné príkazy:
```bash
# Spustenie kontajnera
docker run -d --name web-server nginx

# Interaktívne spustenie
docker run -it ubuntu:20.04 /bin/bash

# Zoznam bežiacich kontajnerov
docker ps

# Zoznam všetkých kontajnerov
docker ps -a

# Zastavenie kontajnera
docker stop web-server

# Spustenie zastaveného kontajnera
docker start web-server

# Odstránenie kontajnera
docker rm web-server

# Pripojenie do bežiaceho kontajnera
docker exec -it web-server /bin/bash
```

### Praktický príklad:
```bash
# Spustenie web servera s portom a názvom
docker run -d --name moj-nginx -p 8080:80 nginx

# Kontrola logov
docker logs moj-nginx

# Pripojenie do kontajnera
docker exec -it moj-nginx /bin/bash
```

---

## 4. Sieťovanie (Networking)

### Teoretický úvod
Docker poskytuje rôzne typy sietí pre komunikáciu medzi kontajnermi a so vonkajším svetom:

#### Typy sietí:
- **Bridge** - Predvolená sieť pre kontajnery na jednom hoste. Kontajnery môžu komunikovať medzi sebou a s internetom.
- **Host** - Kontajner používa priamo sieť hosta. Najrýchlejšia možnosť, ale menej izolovaná.
- **None** - Kontajner nemá prístup k sieti. Používa sa pre špecifické prípady.
- **Custom** - Vlastné definované siete s pokročilými nastaveniami.

### Základné príkazy:
```bash
# Zoznam sietí
docker network ls

# Vytvorenie vlastnej siete
docker network create moja-siet

# Spustenie kontajnera v konkrétnej sieti
docker run -d --name app1 --network moja-siet nginx

# Pripojenie kontajnera do siete
docker network connect moja-siet app1

# Odpojenie kontajnera zo siete
docker network disconnect moja-siet app1

# Informácie o sieti
docker network inspect moja-siet
```

### Praktický príklad - komunikácia medzi kontajnermi:
```bash
# Vytvorenie vlastnej siete
docker network create app-network

# Spustenie databázy
docker run -d --name mysql-db \
  --network app-network \
  -e MYSQL_ROOT_PASSWORD=heslo123 \
  mysql:8.0

# Spustenie web aplikácie
docker run -d --name web-app \
  --network app-network \
  -p 3000:3000 \
  node:16

# Kontajnery sa môžu komunikovať cez názvy
# web-app môže pristupovať k databáze cez "mysql-db:3306"
```

---

## 5. Volumes a Storage

### Teoretický úvod
Volumes umožňujú trvalé ukladanie dát mimo kontajnera. Existujú tri typy:

| Typ | Popis | Použitie |
|-----|-------|----------|
| **Named volumes** | Spravované Dockerom | Produkčné databázy |
| **Bind mounts** | Pripojenie priečinka z hosta | Vývoj aplikácií |
| **tmpfs mounts** | Dočasné úložisko v pamäti | Citlivé dáta |

### Základné príkazy:
```bash
# Zoznam volumes
docker volume ls

# Vytvorenie volume
docker volume create moje-data

# Spustenie kontajnera s volume
docker run -d --name db -v moje-data:/var/lib/mysql mysql:8.0

# Bind mount - pripojenie priečinka
docker run -d --name web -v /host/path:/container/path nginx

# Informácie o volume
docker volume inspect moje-data

# Odstránenie volume
docker volume rm moje-data
```

### Praktický príklad - trvalé uloženie databázy:
```bash
# Vytvorenie volume pre databázu
docker volume create mysql-data

# Spustenie MySQL s trvalým úložiskom
docker run -d --name mysql-server \
  -e MYSQL_ROOT_PASSWORD=heslo123 \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0

# Aj po odstránení kontajnera ostanú dáta zachované
docker rm -f mysql-server
docker run -d --name mysql-server2 \
  -e MYSQL_ROOT_PASSWORD=heslo123 \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0
```

---

## 6. Environment Variables a Konfigurácia

### Teoretický úvod
Environment variables umožňujú konfigurovať aplikácie bez zmeny kódu. Docker podporuje rôzne spôsoby ich zadávania.

### Základné príkazy:
```bash
# Nastavenie environment variable
docker run -e MYSQL_ROOT_PASSWORD=heslo123 mysql:8.0

# Načítanie z .env súboru
docker run --env-file .env mysql:8.0

# Zobrazenie environment variables v kontajneri
docker exec container_name env
```

### Praktický príklad s .env súborom:
```bash
# .env súbor
MYSQL_ROOT_PASSWORD=tajne_heslo
MYSQL_DATABASE=moja_databaza
MYSQL_USER=user
MYSQL_PASSWORD=user_heslo
```

```bash
# Spustenie s .env súborom
docker run -d --name mysql-configured \
  --env-file .env \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

---

## 7. Docker Compose

> **Tip:** Docker Compose je ideálny pre lokálny vývoj a staging prostredia. Pre produkciu zvážte Kubernetes alebo Docker Swarm.

### Teoretický úvod
Docker Compose umožňuje definovať a spravovať viac-kontajnerové aplikácie pomocou YAML súboru. Ideálne pre lokálny vývoj a testing.

### Základná syntax docker-compose.yml:
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    volumes:
      - .:/app
    depends_on:
      - database

  database:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: heslo123
      MYSQL_DATABASE: myapp
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"

volumes:
  mysql_data:
```

### Základné príkazy:
```bash
# Spustenie všetkých služieb
docker-compose up -d

# Zastavenie všetkých služieb
docker-compose down

# Zobrazenie stavu služieb
docker-compose ps

# Zobrazenie logov
docker-compose logs

# Rebuild a reštart
docker-compose up --build

# Spustenie príkazu v službe
docker-compose exec web /bin/bash
```

### Komplexný praktický príklad - Web aplikácia s databázou:

<details>
<summary>Zobrazenie úplného docker-compose.yml</summary>

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Web aplikácia
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://backend:5000
    volumes:
      - ./frontend:/app
      - /app/node_modules
    depends_on:
      - backend
    restart: unless-stopped

  # Backend API
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=mysql://user:heslo@database:3306/myapp
      - NODE_ENV=development
      - REDIS_URL=redis://cache:6379
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - database
      - cache
    restart: unless-stopped

  # MySQL databáza
  database:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root_heslo
      MYSQL_DATABASE: myapp
      MYSQL_USER: user
      MYSQL_PASSWORD: heslo
    volumes:
      - mysql_data:/var/lib/mysql
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    restart: unless-stopped

  # Redis cache
  cache:
    image: redis:alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    restart: unless-stopped

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    restart: unless-stopped

volumes:
  mysql_data:
  redis_data:

networks:
  default:
    name: myapp-network
```

</details>

### Spustenie a správa:
```bash
# Spustenie celej aplikácie
docker-compose up -d

# Sledovanie logov konkrétnej služby
docker-compose logs -f backend

# Reštart konkrétnej služby
docker-compose restart backend

# Scaling služby
docker-compose up --scale backend=3

# Vyčistenie všetkého
docker-compose down -v --rmi all
```

---

## 8. Bezpečnosť a Best Practices

### Optimalizácia Dockerfile:
```dockerfile
# Dobré praktiky
FROM node:16-alpine

# Vytvorenie non-root usera
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Nastavenie working directory
WORKDIR /app

# Kopírovanie package.json najprv (lepší caching)
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Kopírovanie zdrojového kódu
COPY --chown=nextjs:nodejs . .

# Nastavenie usera
USER nextjs

# Definovanie portu
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["npm", "start"]
```

### Bezpečnostné tipy:

> **⚠️ DÔLEŽITÉ:** Nikdy neukladajte citlivé údaje priamo do Dockerfile alebo docker-compose.yml! Používajte Docker secrets alebo external configuration.

**Dobré praktiky:**
- ✅ Používajte specific image tags namiesto `latest`
- ✅ Spúšťajte kontajnery ako non-root user
- ✅ Skenujte images na vulnerabilities
- ✅ Používajte multi-stage builds
- ❌ Neinkludujte .git, node_modules v build context

---

## 9. Monitoring a Debugging

### Užitočné príkazy na údržbu:
```bash
# Vyčistenie všetkých nepoužívaných objektov
docker system prune -a

# Vyčistenie volumes
docker volume prune

# Zobrazenie využitia miesta
docker system df

# Sledovanie využitia zdrojov
docker stats

# Export/Import kontajnera
docker export container_name > backup.tar
docker import backup.tar new_image:tag
```

### Monitoring a debugging:
```bash
# Sledovanie logov v reálnom čase
docker logs -f --tail 100 container_name

# Zobrazenie procesov v kontajneri
docker top container_name

# Inšpekcia kontajnera (detailné informácie)
docker inspect container_name

# Kopírovanie súborov do/z kontajnera
docker cp file.txt container_name:/path/
docker cp container_name:/path/file.txt ./

# Sledovanie udalostí
docker events

# Zobrazenie port mappings
docker port container_name
```

### Docker Compose monitoring:
```bash
# Zobrazenie využitia zdrojov všetkých služieb
docker-compose top

# Sledovanie logov všetkých služieb
docker-compose logs -f --tail=100

# Validácia compose súboru
docker-compose config

# Zobrazenie závislostí medzi službami
docker-compose ps --services
```

---

## 10. Pokročilé témy

### Multi-stage builds
```dockerfile
# Build stage
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
USER node
CMD ["npm", "start"]
```

### Docker secrets (pre produkciu):
```bash
# Vytvorenie secret
echo "moje_tajne_heslo" | docker secret create mysql_password -
```

```yaml
# Použitie v docker-compose.yml
version: '3.8'
services:
  database:
    image: mysql:8.0
    secrets:
      - mysql_password
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_password

secrets:
  mysql_password:
    external: true
```

---

## Záver a ďalšie kroky

> **🎉 Gratulujeme!** Teraz poznáte základy Dockeru. Docker je mocný nástroj, ktorý zjednodušuje deployment a správu aplikácií.

### Kľúčové koncepty na zapamätanie:

1. **Images** sú šablóny, **containers** sú bežiace inštancie
2. **Volumes** zabezpečujú trvalé uloženie dát
3. **Networks** umožňujú komunikáciu medzi kontajnermi
4. **Docker Compose** zjednodušuje správu viac-kontajnerových aplikácií
5. **Environment variables** umožňujú flexibilnú konfiguráciu

### Ďalšie kroky:

#### Pre pokročilých:
- Kubernetes orchestrácia
- Docker Swarm clustering  
- CI/CD integrácia
- Container security scanning

#### Užitočné zdroje:
- [Docker oficiálna dokumentácia](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)

> **💡 Tip pre začiatočníkov:** Začnite s jednoduchými príkladmi a postupne pridávajte komplexnejšie funkcie. Docker vám ušetrí veľa času pri vývoji a deploymente aplikácií!
*Návod vytvoreny pomocou [AI](https://claude.ai/)*
