---
title: Ako som si vytvoril lacný blog za 6,89 EUR ročne
date: 2025-09-08T10:00:00+02:00
draft: false
tags:
  - blog
  - hugo
  - azure
  - cloudflare
  - návod
categories:
  - technológie
  - web development
summary: Kompletný návod ako si vytvoriť moderný blog za cenu len jednej domény ročne pomocou Hugo, Azure Static Web Apps a Cloudflare.
slug: ako-som-si-vytvoril-lacny-blog-za-6-89-eur-rocne
---
<p style="text-align:block;">Dlho som sa pohrával s myšlienkou vytvoriť si vlastný blog, ale
vždy ma odradili vysoké mesačné poplatky za hosting a komplikované nastavenia. Chcel som niečo jednoduché, rýchle a hlavne lacné - ideálne zadarmo alebo za minimálne náklady. Po dlhšom bádaní som objavil kombináciu moderných technológií,  ktoré mi umožnili vytvoriť profesionálny blog prakticky zadarmo. Tradičné hostingové služby často stoja 5-15 EUR mesačne, čo je pre hobby blog neúmerne veľa. Rozhodol som sa preto hľadať alternatívy v cloudových službách, ktoré ponúkajú štedré free tier plány. Moja filozofia bola jasná: minimálne náklady, maximálna funkcionalita a spoľahlivosť. Po výskume som sa rozhodol pre kombináciu statického site generátora a cloudového hostingu. Výsledok? Moderný, rýchly blog za cenu len jednej domény ročne.</p>

## Použité technológie a náklady

* **Vlastná doména:** 6,89 EUR/rok
* **Cloudflare (DNS):** 0,00 EUR
* **Azure Static Web Apps:** 0,00 EUR  
* **Hugo (generátor stránok):** 0,00 EUR
* **GitHub (verzovanie a CI/CD):** 0,00 EUR

**Celkové náklady: 6,89 EUR ročne**

## Krok za krokom návod

### 1. Kúpa domény

* Vyberte si registrátora domén (ja som použil forpsi.sk)

* Zadajte požadované doménové meno a dokončite nákup (meno mojej domeny je kuri11a.eu)
* Získate prístupové údaje na správu DNS nastavení

### 2. Nastavenie Cloudflare

* Zaregistrujte sa na [cloudflare.com](https://cloudflare.com)

* Pridajte vašu doménu do Cloudflare
* Zmeňte nameservery u vášho registrátora na tie, ktoré poskytne Cloudflare
* Počkajte na aktiváciu (zvyčajne 24 hodín)

### 3. Vytvorenie GitHub repozitára

* Zaregistrujte sa na [github.com](https://github.com) (ak nemáte účet)

* Vytvorte nový verejný repozitár pre váš blog
* Klonujte repozitár na váš počítač pomocou `git clone [URL]`

### 4. Inštalácia Hugo

* Stiahnite Hugo z [gohugo.io](https://gohugo.io/installation/)

* Pre Windows: stiahnite ZIP, rozbaľte a pridajte do PATH
* Pre macOS: `brew install hugo`
* Pre Linux: `sudo apt install hugo` alebo stiahnite .deb balík
* Overte inštaláciu: `hugo version`

### 5. Vytvorenie Hugo stránky

* Prejdite do priečinka s repozitárom

* Vytvorte novú Hugo stránku: `hugo new site .`
* Vyberte a nainštalujte tému (napríklad): `git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke`
* Upravte `hugo.toml` a nastavte tému: `theme = 'ananke'`

### 6. Konfigurácia Hugo

* Otvorte `hugo.toml` a nastavte základné parametre:

  ```toml
  baseURL = 'https://kuri11a.eu'
  languageCode = 'sk'
  title = 'Názov vášho blogu'
  theme = 'ananke'
  ```

* Prispôsobte menu, footer a ďalšie nastavenia podľa témy

### 7. Vytvorenie Azure Static Web App

* Zaregistrujte sa na [portal.azure.com](https://portal.azure.com)

* Vytvorte novú "Static Web App"
* Pripojte k vášmu GitHub repozitáru
* Nastavte build settings: Source: `/`, App location: `/`, Output location: `/public`
* Azure automaticky vytvorí GitHub Action pre deployment

### 8. Nastavenie DNS v Cloudflare

* V Cloudflare prejdite na DNS nastavenia

* Vytvorte CNAME záznam (v mojom pripade blog nie www): `www` → `[váš-azure-url].azurestaticapps.net`
* Vytvorte A záznam pre root doménu alebo CNAME alias
* Nastavte SSL/TLS na "Full" v Cloudflare
<br>pozn. nechcelo mi to fungovať (pri zapnutom proxy posielalo len ipv6 adresy, po vypnuti to začalo fungovat správne už to posielalo správnu ipv4 adresu)

### 9. Písanie obsahu

* Vytvorte nový príspevok: `hugo new posts/moj-prvy-prispevok.md`

* Editujte súbor v `content/posts/`
* Nastavte `draft: false` v front matter
* Použite Markdown syntax pre formátovanie

### 10. Publikovanie

* Otestujte lokálne: `hugo server -D`

* Commitnite zmeny: `git add .`, `git commit -m "Nový príspevok"`
* Pushujte na GitHub: `git push origin main`
* Azure automaticky spustí deployment a publikuje zmeny

## Výhody tohto riešenia

* **Rýchlosť:** Statické súbory sa načítajú bleskovo
* **Bezpečnosť:** Žiadna databáza = žiadne bezpečnostné problémy
* **Škálovateľnosť:** Cloudflare a Azure zvládnu aj vysokú návštevnosť
* **Nízke náklady:** Len cena domény
* **Verzovanie:** Celá história zmien v Git
* **Automatické SSL:** Zadarmo HTTPS certifikáty

Tento setup mi perfektne vyhovuje a môžem ho odporučiť každému, kto hľadá lacné a spoľahlivé riešenie pre osobný blog či portfólio.
