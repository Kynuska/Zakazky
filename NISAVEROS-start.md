# NISAVEROS — start.md

> Vstupní bod pro agenta. Čti tento soubor + SESSION.md + PLAN.md.
> Po přečtení víš přesně kde jsme a co dělat.

---

## Projekt

**NISAVEROS** — Výrobní Ekonomický Řídící Operační Systém (Nisaform)
Repozitář: `C:\_Projekty\_NISAVEROS` | GitHub: `https://github.com/Kynuska/_NISAVEROS`
Server: Node.js/Express + SQLite | Port: `:3001`

Samostatný systém. Výdejna Nisaform (`:3000`) zůstává oddělená — komunikuje pouze přes 4 API endpointy.

---

## Startovní postup (provést vždy)

1. Přečíst `SESSION.md` → sdělit stav poslední session a co je na řadě
2. Přečíst sekci **Fáze vývoje a stav** v `PLAN.md` → sdělit aktuální fázi a nejbližší nehotové úkoly
3. Ověřit aktivní soubory (viz níže)
4. Navrhnout logické pokračování nebo se zeptat: **„Co budeme dnes dělat?"**

### Ověření aktivních souborů

```powershell
ls "C:\_Projekty\_NISAVEROS\public\*.html"   # build output
ls "C:\_Projekty\_NISAVEROS\_dev\build.js"   # verze
cat "C:\_Projekty\_NISAVEROS\data\*.sqlite"  # DB existuje?
```

---

## Kde se co upravuje

| Co | Soubor |
|---|---|
| HTML struktura, navigace | `_dev/src/body.html` |
| CSS styly | `_dev/src/style.css` |
| Frontend logika | `_dev/src/js/XX-nazev.js` |
| Verze aplikace | `_dev/build.js` → `const VERSION` |
| API endpointy | `routes/XX.js` |
| DB schéma | `db.js` |
| Auth a role | `auth.js` |

**`public/` je build output — neupravovat přímo.**

Po změně zdrojů spustit build:
```bat
cd _dev && node build.js
```
nebo
```bat
Build.bat
```

---

## Pravidla práce

### Mikrozáloha (před každou změnou HTML nebo buildem)

```
Zaloha/NISAVEROS-YYYYMMDD-HHMMSS-popis-zmeny.html
```

Příkaz:
```powershell
$ts = Get-Date -Format "yyyyMMdd-HHmmss"
cp "public\nisaveros.html" "Zaloha\NISAVEROS-$ts-popis.html"
```

### Příkaz „milník"

Provést v tomto pořadí:

1. Navýšit verzi v `_dev/build.js` (`const VERSION = 'vX.Y'`)
2. Spustit build (`cd _dev && node build.js`)
3. Vytvořit archivní kopii do `Zaloha/`:
   ```
   Zaloha/NISAVEROS-milnik-vX.Y-popis.html
   ```
4. Zaškrtnout dokončené úkoly v `PLAN.md` (sekce Fáze vývoje)
5. Aktualizovat `start.md`:
   - sekce **Aktuální verze**
   - sekce **Aktuální stav aplikace**
6. Aktualizovat `SESSION.md` (viz příkaz „konec")
7. Smazat timestampové mikrozálohy (`-YYYYMMDD-HHMMSS-`) ze `Zaloha/`
8. Git commit + push:
   ```
   git add -A
   git commit -m "vX.Y: popis milníku"
   git push origin master
   ```

Milníkové soubory bez timestampu **nesmazat**.

### Příkaz „konec" (konec dne / konec session)

1. `git fetch origin && git status -sb`
2. Aktualizovat `SESSION.md`:
   - datum session
   - co bylo uděláno
   - co je příště na řadě
   - otevřené body / problémy
3. Commit + push:
   ```
   git add SESSION.md PLAN.md start.md
   git add public/ _dev/src/ routes/ db.js auth.js server.js
   git commit -m "vX.Y: popis změn"
   git push origin master
   ```
4. Shrnout: co bylo commitnuto, zda push proběhl, otevřené body

**Necommitovat:** `data/nisaveros.sqlite`, `Zaloha JSON/`, timestampové mikrozálohy.

---

## Git identita (pouze lokálně, bez --global)

```powershell
git config user.name "Aleš Hnyk"
git config user.email "ales.hnyk@centrum.cz"
```

---

## Aktuální verze

`v0.0` — projekt zatím neexistuje, čeká na setup (Fáze 0)

---

## Aktuální stav aplikace

*Bude doplněno po prvním milníku.*

---

## Integrace s Výdejnou Nisaform

Výdejna volá NISAVEROS na 4 endpointech:
```
GET  /api/objednavky/next-number
POST /api/objednavky
POST /api/objednavky/:id/naskladneni
GET  /api/objednavky/:id
```
Implementace v Fázi 4. Do té doby jsou systémy nezávislé.

---

## Kontext: co NISAVEROS nahrazuje / rozšiřuje

| Systém | Stav | Vztah k NISAVEROS |
|---|---|---|
| Zakázky v1.4 (`C:\_Projekty\_Zakazky`) | aktivní | data budou migrována do NISAVEROS (Fáze 1) |
| Výdejna Nisaform v16.x (`C:\_Projekty\_Vydejna_Nisaform`) | aktivní, beze změny | propojení přes API v Fázi 4 |
