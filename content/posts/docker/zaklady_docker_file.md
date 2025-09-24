---
title: "Docker Kontajnery a Dockerfile - Komplexný Návod"
date: 2025-09-24T09:00:00+02:00
draft: false
author: "Váš Autor"
categories: ["DevOps"]
tags: ["docker", "dockerfile", "kontajnery", "devops", "tutorial"]
description: "Komplexný návod na vytváranie Docker kontajnerov a písanie Dockerfile s praktickými príkladmi a best practices."
summary: "Naučte sa základy Docker kontajnerov, všetky dôležité Dockerfile príkazy a best practices pre efektívnu kontajnerizáciu aplikácií."
cover:
    image: "docker-guide-cover.jpg"
    alt: "Docker kontajnery a Dockerfile návod"
    caption: "Komplexný návod na Docker kontajnery"
ShowToc: true
TocOpen: true
---

## Čo je Docker?

Docker je platforma pre kontajnerizáciu aplikácií, ktorá umožňuje zabaliť aplikáciu a všetky jej závislosti do ľahkého, prenosného kontajnera.

## Vytvorenie Docker Kontajnera

### Základné kroky:

1. Vytvorenie Dockerfile
2. Build image pomocou `docker build`
3. Spustenie kontajnera pomocou `docker run`

```bash
# Build image
docker build -t moja-aplikacia .

# Spustenie kontajnera
docker run -p 8080:80 moja-aplikacia
```

## Dockerfile - Základné Príkazy

### FROM

Definuje základný image, z ktorého sa bude váš image vytvárať.

```dockerfile
# Oficiálny Node.js image
FROM node:18

# Ubuntu base image
FROM ubuntu:22.04

# Alpine Linux (malý, bezpečný)
FROM node:18-alpine
```

### WORKDIR

Nastavuje pracovný adresár v kontajneri.

```dockerfile
WORKDIR /app
# Všetky následné príkazy sa budú vykonávať v /app
```

### COPY a ADD

Kopírujú súbory z hostiteľského systému do kontajnera.

```dockerfile
# COPY (odporúčané)
COPY package.json ./
COPY . .

# ADD (môže rozbaľovať archívy a sťahovať z URL)
ADD https://example.com/file.tar.gz /tmp/
```

### RUN

Vykonáva príkazy počas build procesu.

```dockerfile
# Inštalácia balíkov
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Node.js závislosti
RUN npm install
```

### CMD

Definuje predvolený príkaz, ktorý sa spustí pri štarte kontajnera.

```dockerfile
# Exec form (odporúčané)
CMD ["npm", "start"]

# Shell form
CMD npm start
```

### ENTRYPOINT

Konfiguruje kontajner ako spustiteľný súbor.

```dockerfile
ENTRYPOINT ["python3", "app.py"]
# Parametre môžu byť pridané cez docker run
```

### EXPOSE

Informuje Docker o portoch, ktoré aplikácia používa.

```dockerfile
EXPOSE 8080
EXPOSE 3000 80
```

### ENV

Nastavuje environment premenné.

```dockerfile
ENV NODE_ENV=production
ENV PORT=8080
ENV DATABASE_URL=postgresql://user:pass@localhost/db
```

### ARG

Definuje build-time premenné.

```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}

ARG BUILD_DATE
LABEL build-date=$BUILD_DATE
```

### VOLUME

Vytvára mount pointy pre persistentné dáta.

```dockerfile
VOLUME ["/data", "/logs"]
```

### USER

Nastavuje používateľa pre spúšťanie príkazov.

```dockerfile
# Vytvorenie používateľa
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

USER nextjs
```

### LABEL

Pridáva metadata k image.

```dockerfile
LABEL version="1.0"
LABEL description="Moja Node.js aplikácia"
LABEL maintainer="meno@email.com"
```

## Praktické Príklady

### 1. Node.js Aplikácia

```dockerfile
# Použitie oficiálneho Node.js image
FROM node:18-alpine

# Nastavenie pracovného adresára
WORKDIR /app

# Kopírovanie package files
COPY package*.json ./

# Inštalácia závislostí
RUN npm ci --only=production

# Kopírovanie zdrojového kódu
COPY . .

# Vytvorenie non-root používateľa
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Zmena vlastníctva súborov
RUN chown -R nextjs:nodejs /app
USER nextjs

# Exponovanie portu
EXPOSE 3000

# Environment premenná
ENV NODE_ENV=production

# Spustenie aplikácie
CMD ["npm", "start"]
```

### 2. Python Flask Aplikácia

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Inštalácia system závislostí
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Kopírovanie requirements
COPY requirements.txt .

# Inštalácia Python závislostí
RUN pip install --no-cache-dir -r requirements.txt

# Kopírovanie aplikácie
COPY . .

# Vytvorenie používateľa
RUN adduser --disabled-password --gecos '' appuser
RUN chown -R appuser:appuser /app
USER appuser

EXPOSE 5000

ENV FLASK_APP=app.py
ENV FLASK_ENV=production

CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]
```

### 3. Multi-stage Build (Optimalizácia)

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Kopírovanie iba production dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Kopírovanie built aplikácie z builder stage
COPY --from=builder /app/dist ./dist

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
RUN chown -R nextjs:nodejs /app
USER nextjs

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

## Best Practices a Tipy

### 1. Minimalizácia Vrstiev

```dockerfile
# ❌ Zlé - veľa vrstiev
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git

# ✅ Dobré - jedna vrstva
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*
```

### 2. Využitie Cache

```dockerfile
# Kopírovanie package.json pred zdrojovým kódom
# Cache sa nepreruší ak sa zmení iba zdrojový kód
COPY package*.json ./
RUN npm install

# Až potom kopírovanie zdrojového kódu
COPY . .
```

### 3. .dockerignore Súbor

```dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.nyc_output
```

### 4. Security Best Practices

```dockerfile
# Nepoužívanie root používateľa
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Minimálny base image
FROM node:18-alpine

# Aktualizácia balíkov
RUN apk update && apk upgrade
```

### 5. Environment Variables

```dockerfile
# Použitie ARG pre build-time premenné
ARG NODE_ENV=production
ENV NODE_ENV=$NODE_ENV

# Sensible defaults
ENV PORT=8080
ENV HOST=0.0.0.0
```

## Užitočné Docker Príkazy

```bash
# Build s tagom
docker build -t moja-app:v1.0 .

# Build s argumentami
docker build --build-arg NODE_ENV=development -t moja-app .

# Spustenie s mapovaním portov
docker run -p 8080:3000 moja-app

# Spustenie s environment premennými
docker run -e NODE_ENV=development -p 8080:3000 moja-app

# Spustenie s volume mount
docker run -v $(pwd):/app -p 8080:3000 moja-app

# Zobrazenie informácií o image
docker inspect moja-app

# Zobrazenie histórie vrstiev
docker history moja-app
```

## Debugging a Troubleshooting

### Vstup do bežiaceho kontajnera

```bash
docker exec -it  /bin/sh
```

### Zobrazenie logov

```bash
docker logs 
docker logs -f   # sledovanie v real-time
```

### Overenie build procesu

```bash
# Spustenie intermediate kontajnera pre debugging
docker run -it  /bin/sh
```

## Záver

Docker kontajnery predstavujú moderný spôsob deploymentu aplikácií s mnohými výhodami ako je prenositeľnosť, konzistentnosť prostredia a jednoduchá škálovateľnosť. Správne napísaný Dockerfile s dodržaním best practices zabezpečí efektívne a bezpečné kontajnery.

### Kľúčové takeaways:

- Používajte minimálne base images (Alpine)
- Optimalizujte vrstvy pre lepšie cache využitie
- Nikdy nespúšťajte aplikácie ako root
- Využívajte multi-stage builds pre produkčné aplikácie
- Dbajte na bezpečnosť a aktualizácie

### Ďalšie kroky:

1. Vyskúšajte si príklady v tomto návode
2. Prečítajte si oficiálnu Docker dokumentáciu
3. Experimentujte s pokročilými funkciami ako Docker Compose
4. Naučte sa orchestráciu s Kubernetes

---

*Tento návod poskytuje solídny základ pre prácu s Docker kontajnermi. Pre najnovšie informácie odporúčame navštíviť [oficiálnu Docker dokumentáciu](https://docs.docker.com/).*