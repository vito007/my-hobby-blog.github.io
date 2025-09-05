---
title: "Cloudflare Tunnel na OpenMediaVault 6, prístup bez verejnej IP"
date: 2024-10-05T15:30:00+02:00
draft: false
tags: ["cloudflare", "omv", "docker", "tunnel", "self-hosting", "odroid"]
categories: ["Návody", "Self-hosting"]
author: "Admin"
description: "Návod ako nastaviť Cloudflare Tunnel na OpenMediaVault 6 pre bezpečný vzdialený prístup bez potreby verejnej ip adresy."
featured_image: "/images/cloudflare-omv-docker.jpg"
toc: true
---
Použité technológie

<img src="https://cf-assets.www.cloudflare.com/dzlvafdwdttg/69wNwfiY5mFmgpd9eQFW6j/d5131c08085a977aa70f19e7aada3fa9/1pixel-down__1_.svg"  width="150" style=display:inline;padding:10px;>
<img src="https://www.openmediavault.org/wp-content/uploads/2016/09/header_logo3.png" width="150" style=display:inline;padding:10px;>
<img src="https://www.docker.com/app/uploads/2023/08/logo-guide-logos-1.svg" width="150" style=display:inline;padding:10px;>

## Úvod

Cloudflare Tunnel je skvelé riešenie pre bezpečný prístup k vašemu OpenMediaVault serveru bez potreby vlastnenia verejnej ip adresy. V tomto návode si ukážeme, ako nastaviť Cloudflare Tunnel na OMV6 bežiacom na Odroid HC4 pomocou Docker kontajnera.

<!--more-->

## Prečo Docker?

Použitie Docker kontajnera namiesto priamej inštalácie na host systém má niekoľko výhod:

- **Lepšia izolácia** - kontajner beží oddelene od host systému (vyššia bezpečnosť)
- **Jednoduchšie aktualizácie** - stačí stiahnuť nový image
- **Rýchly rollback** - možnosť rýchleho návratu k predchádzajúcej verzii
- **Minimálne privilégiá** - kontajner beží s obmedzenými právami

## Požiadavky

Pred začatím sa uistite, že máte:

- nainštalovaný OMV6
- Vlastnú doménu zaregistrovanú na Cloudflare 
- Prístup k Cloudflare Dashboard
- SSH prístup k OMV serveru

## Krok 1: Príprava OMV6

### Inštalácia Docker pluginu

1. Prihlásте sa do OMV Web UI (zvyčajne `http://IP_ADRESA`)
2. Prejdite na **System** → **Plugins**
3. Nájdite a nainštalujte **openmediavault-compose**
4. Po inštalácii reštartujte OMV: **System** → **Reboot**

### Aktivácia Docker služby

1. Po reštarte prejdite na **Services** → **Compose**
2. Aktivujte službu ak nie je aktívna

## Krok 2: Vytvorenie Cloudflare Tunnel

### Konfigurácia v Cloudflare Dashboard

1. Prihlaste sa na [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Prejdite na **Zero Trust** → **Networks** → **Tunnels**
3. Kliknite **Create a tunnel**
4. Vyberte **Cloudflared** a zadajte názov (napr. "omv-tunnel")
5. **DÔLEŽITÉ:** Skopírujte tunnel token - začína s "eyJ..."

### Nastavenie DNS záznamu

V tunnel konfigurácii pridajte **Public Hostname**:

- **Subdomain:** omv (alebo názov podľa vášho výberu)
- **Domain:** vasa-domena.com
- **Service:** HTTP://IP_OMV:80  - IP_OMV = najlepsie ak je to statická IP adresa (napriklad 192.168.1.100)

## Krok 3: Docker Compose konfigurácia

V OMV Web UI prejdite na **Services** → **Compose** → **Add**

**Názov:** cloudflare-tunnel

```yaml
version: '3.8'
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared-tunnel
    restart: unless-stopped
    command: tunnel --no-autoupdate run --token TVOJ_TUNNEL_TOKEN_SEM
    network_mode: host
    security_opt:
      - no-new-privileges:true
    read_only: true
    user: "65534:65534"  # nobody user
    environment:
      - TUNNEL_METRICS=0.0.0.0:8080
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:8080/ready"]
      interval: 10s
      timeout: 3s
      retries: 3
```

{{< highlight yaml >}}
# Nezabudnite nahradiť TVOJ_TUNNEL_TOKEN_SEM skutočným tokenom!
{{< /highlight >}}

### Spustenie kontajnera

1. Kliknite **Save**
2. Kliknite **Up** pre spustenie
3. Skontrolujte stav v **Containers** tabe

## Krok 4: Verifikácia a testovanie

### Kontrola stavu kontajnera

```bash
# SSH pripojenie k OMV
ssh root@IP_OMV

# Kontrola bežiaceho kontajnera
docker ps | grep cloudflared

# Kontrola logov
docker logs cloudflared-tunnel
```

### Test pripojenia

Otvorte prehliadač a prejdite na `https://omv.vasa-domena.com`. Malo by sa zobraziť OMV login screen.

## Pokročilá konfigurácia

### Viacero služieb

Ak chcete vystaviť viac služieb, upravte nastavenia v Cloudflare Dashboard:<br>
IP adresa hostu na ktorom bežia služby - je to statická IP adresa (napriklad 192.168.1.100) , môžu to byť aj služby na viacerých hostoch (rôzne ip adresy)

**Tunnel** → **Configure** → **Public Hostnames:**

- `omv.domena.com` → `http://192.168.1.100:80`
- `portainer.domena.com` → `http://192.168.1.100:9443`
- `nextcloud.domena.com` → `http://192.168.1.100:8080`

### Monitoring

Pre monitoring môžete pridať do compose súboru:

```yaml
  tunnel-monitor:
    image: nicolargo/glances:latest
    container_name: tunnel-monitor
    restart: unless-stopped
    ports:
      - "61208:61208"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
```

## Riešenie problémov

### Časté problémy

**Tunnel sa nepripojí:**
- Skontrolujte token
- Verifikujte DNS nastavenia v Cloudflare

**403 Error:**
- Skontrolujte či je doména správne nastavená
- Verifikujte Zero Trust nastavenia

**Kontajner sa reštartuje:**
```bash
docker logs cloudflared-tunnel --tail 50
```

### Užitočné príkazy

```bash
# Reštart tunnel
docker restart cloudflared-tunnel

# Live logy
docker logs -f cloudflared-tunnel

# Kontrola network
docker network ls
```

## Bezpečnostné odporúčania

1. **Aktivujte Cloudflare Access** pre extra zabezpečenie - da sa nastavit multifaktorové overovanie, rôzne obmedzenia (napr geolokacia, ...)
2. **Nastavte firewall pravidlá** - blokujte priamy prístup na port 80/443
3. **Pravidelne aktualizujte** Docker image
4. **Monitorujte logy** pre podozrivú aktivitu
5. **Používajte silné heslá** pre OMV účty

## Záver

Cloudflare Tunnel poskytuje bezpečný a spoľahlivý spôsob prístupu k vášmu OpenMediaVault serveru z ľubovoľného miesta na svete bez potreby verejnej ip adresy. Použitie Docker kontajnera zabezpečuje dodatočnú vrstvu izolácie a uľahčuje údržbu systému.

Tento setup je ideálny pre domácich používateľov, ktorí chcú mať vzdialený prístup k svojim súborom a službám.

---

*Návod vytvoreny pomocou [AI](https://claude.ai/)*