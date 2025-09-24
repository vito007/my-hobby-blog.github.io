---
title: "Git pre začiatočníkov - Kompletný sprievodca"
date: 2025-09-04T10:00:00+02:00
draft: false
tags: ["git", "programovanie", "verzovanie"]
categories: ["Návody", "Technológie"]
author: "Učiteľ"
description: "Kompletný sprievodca pre začiatočníkov - naučte sa základy Gitu krok za krokom s praktickými príkladmi"
---

# Git pre začiatočníkov - Kompletný sprievodca

Ahoj! Ako tvoj učiteľ ťa naučím pracovať s Gitom krok za krokom. Git je nástroj na verzovanie kódu - predstav si ho ako "cestovanie v čase" pre tvoje projekty.

## Krok 1: Čo je Git a prečo ho potrebuješ?

Git ti umožňuje:
- Sledovať zmeny v kóde
- Vrátiť sa k starším verziám
- Spolupracovať s ostatnými programátormi
- Uchovávať zálohy svojej práce

Predstav si, že píšeš esej. S Gitom môžeš uložiť každú verziu, vidieť čo si zmenil a vrátiť sa k predchádzajúcej verzii, ak niečo pokazíš.

## Krok 2: Inštalácia Gitu

**Windows:** Stiahni Git z [git-scm.com](https://git-scm.com)  
**Mac:** `brew install git` alebo stiahni z [git-scm.com](https://git-scm.com)  
**Linux:** `sudo apt install git`

Overiť inštaláciu:
```bash
git --version
```

## Krok 3: Prvotné nastavenie

```bash
git config --global user.name "Tvoje Meno"
git config --global user.email "tvoj@email.com"
```

## Krok 4: Základné pojmy (musíš ich vedieť!)

- **Repository (repo)** = priečinok s projektom pod správou Gitu
- **Commit** = "snímka" projektu v určitom čase
- **Branch** = vetva vývoja projektu
- **Remote** = vzdialené úložisko (napr. GitHub)

## Krok 5: Vytvorenie prvého repozitára

```bash
# Vytvor priečinok pre projekt
mkdir moj-projekt
cd moj-projekt

# Inicializuj Git repozitár
git init
```

Teraz máš prázdny Git repozitár!

## Krok 6: Prvý commit (tvoja prvá "snímka")

```bash
# Vytvor súbor
echo "Ahoj svet!" > index.html

# Pridaj súbor do "staging area"
git add index.html

# Urob commit
git commit -m "Pridaný prvý súbor"
```

**Vysvetlenie:**
- `git add` = priprav súbor na commit
- `git commit -m "správa"` = urob snímku so správou

## Krok 7: Sledovanie stavu

```bash
# Zobraz stav repozitára
git status

# Zobraz históriu commitov
git log

# Zobraz zmeny v súboroch
git diff
```

## Krok 8: Praktický príklad - editácia a nový commit

```bash
# Uprav súbor
echo "<h1>Moja stránka</h1>" > index.html

# Zobraz zmeny
git diff

# Pridaj zmeny
git add index.html

# Urob nový commit
git commit -m "Pridaný nadpis"
```

## Krok 9: Práca s vetvami (branches)

```bash
# Vytvor novú vetvu
git branch nova-funkcia

# Prepni sa na novú vetvu
git checkout nova-funkcia
# ALEBO modernejšie:
git switch nova-funkcia

# Vytvor a prepni sa naraz
git checkout -b ina-vetva
```

{{< alert type="info" >}}
**Prečo vetvy?** Umožňujú ti pracovať na novej funkcii bez ovplyvnenia hlavnej vetvy.
{{< /alert >}}

## Krok 10: Zlučovanie vetiev (merge)

```bash
# Prepni sa na hlavnú vetvu
git switch main

# Zlúč vetvu
git merge nova-funkcia
```

## Krok 11: Pripojenie na GitHub/vzdialené úložisko

najprv na stranke https://github.com/settings/tokens vygenerujem token ktory pouzijem v prikaze
```bash
# Pripoj vzdialené úložisko
git remote set-url origin https://TOKEN@github.com/USERNAME/REPO/

# Nahraj kód na GitHub
git push -u origin main
```

## Krok 12: Sťahovanie zmien

```bash
# Stiahni zmeny zo vzdialeného úložiska
git pull origin main
```

## Krok 13: Praktické cvičenie

1. Vytvor súbor `style.css`
2. Pridaj ho do Gitu (`git add`)
3. Urob commit
4. Vytvor novú vetvu `styling`
5. Uprav CSS súbor
6. Commitni zmeny
7. Prepni sa na `main`
8. Zlúč vetvy

```bash
echo "body { color: blue; }" > style.css
git add style.css
git commit -m "Pridaný CSS súbor"
git checkout -b styling
echo "body { color: red; background: yellow; }" > style.css
git add style.css
git commit -m "Zmenené farby"
git checkout main
git merge styling
```

## Krok 14: Riešenie konfliktov

Keď dva ľudia zmenia ten istý riadok, Git nevie ktorú verziu použiť:

```bash
# Git označí konflikt v súbore takto:
<<<<<<< HEAD
Tvoja verzia
=======
Verzia z inej vetvy
>>>>>>> branch-name
```

{{< alert type="warning" >}}
**Riešenie:** Uprav súbor ručne, zmaž značky `<<<<<<<`, `=======`, `>>>>>>>>` a urob commit.
{{< /alert >}}

## Krok 15: Užitočné príkazy na záver

```bash
# Zobraz všetky vetvy
git branch -a

# Zmaž vetvu
git branch -d nazov-vetvy

# Vráť zmeny v súbore
git checkout -- nazov-suboru

# Zobraz krásnu históriu
git log --oneline --graph

# Zruš posledný commit (ale ponechaj zmeny)
git reset --soft HEAD~1
```

## Tippy pre každodenné používanie

{{< alert type="success" >}}
1. **Commituj často** - malé, logické zmeny
2. **Píš jasné commit správy** - napr. "Opravená chyba v prihlásení"
3. **Používaj vetvy** - pre každú novú funkciu
4. **Testuj pred commitom** - uisti sa, že kód funguje
5. **Sťahuj zmeny pravidelne** - `git pull` pred začiatkom práce
{{< /alert >}}

## Záver

Pamätaj si: Git je ako záchranná sieť pre tvoj kód. Čím viac ho používaš, tím väčšiu istotu máš, že o svoju prácu nikdy neprídeš!

---

*Tento tutoriál ti poskytol základy práce s Gitom. Pre pokročilejšie témy odporúčam oficiálnu dokumentáciu na [git-scm.com](https://git-scm.com/doc).*
*Návod vytvoreny pomocou [AI](https://claude.ai/)*
