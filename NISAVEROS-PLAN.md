# NISAVEROS — PLAN.md

> **NISA + VEROS** (Výrobní Ekonomický Řídící Operační Systém)
> Repozitář: `_NISAVEROS` | Server port: `:3001`

Tento soubor je zdrojem pravdy o stavu projektu.
Agent ho čte při každém startu a aktualizuje při každém milníku.

---

## Architektura

```
PC Dílna        → :3001/dilna        role: dilna
PC Technologie  → :3001/technologie  role: technolog
PC Ekonom       → :3001/ekonom       role: ekonom
Libovolné PC    → :3001/admin        role: admin

Výdejna Nisaform (:3000) ←→ NISAVEROS (:3001)
  pouze 4 API endpointy (viz sekce Integrace)
```

### Struktura repozitáře

```
_NISAVEROS/
├── server.js               ← Node.js/Express, port 3001
├── db.js                   ← SQLite init, tabulky, migrace
├── auth.js                 ← session cookie, PIN/heslo, middleware rolí
├── routes/
│   ├── zakazky.js
│   ├── zamestnanci.js
│   ├── stroje.js
│   ├── zakaznici.js
│   ├── dodavatele.js
│   ├── objednavky.js
│   ├── dodavky.js
│   └── faktury.js
├── _dev/
│   ├── build.js            ← VERSION konstanta zde
│   └── src/
│       ├── body.html
│       ├── style.css
│       └── js/
│           ├── 01-auth.js
│           ├── 10-dilna.js
│           ├── 20-zakazky.js
│           ├── 30-objednavky.js
│           ├── 40-dodavky.js
│           ├── 50-faktury.js
│           └── 90-admin.js
├── public/                 ← build output — neupravovat přímo
├── data/
│   └── nisaveros.sqlite
├── Zaloha/                 ← mikrozálohy HTML + archivní milníky
├── SESSION.md              ← stav aktuální/poslední session
├── PLAN.md                 ← tento soubor
├── start.md                ← vstupní bod pro agenta
├── install.bat
├── start-server.vbs
└── package.json
```

---

## Role a přístup

| Role | Přístup |
|---|---|
| `dilna` | Clock-in/out přes čtečku, výběr zakázky/stroje |
| `technolog` | Správa zakázek, záznamy práce, přehledy výroby |
| `ekonom` | Objednávky, dodávky, fakturace (vstupní i výstupní) |
| `admin` | Vše + správa uživatelů, číselníků, rolí |

Auth: session cookie, přihlášení jménem + PINem (dílna) nebo heslem (ostatní).

---

## Číslování dokladů

| Doklad | Formát | Příklad | Reset |
|---|---|---|---|
| Zakázka | `RR-NNNN` | `26-0025` | každý rok k 1. 1. |
| Objednávka | `NNNN/MMRR` | `0529/0526` | průběžné, bez resetu |
| Faktura výstupní | `RRMM/NNNN` | `2608/0256` | každý rok k 1. 1. |
| Faktura vstupní | `RRMM/NNNN` | `2608/0101` | každý rok k 1. 1. |

Čítače v tabulce `sekvence` (typ, rok, posledni_cislo).

---

## Databázové schéma

### Číselníky

```sql
stroje        (id, nazev, sazba_kc_hod, aktivni)
zamestnanci   (id, jmeno, carovy_kod, pin, role, aktivni)
zakaznici     (id, nazev, ico, dic, adresa, email, telefon)
dodavatele    (id, nazev, ico, dic, adresa, email)
sekvence      (typ TEXT PK, rok INT, posledni_cislo INT)
```

### Výroba

```sql
zakazky (
  id, cislo TEXT UNIQUE,       -- "26-0025"
  zakaznik_id,
  nazev, popis,
  stav TEXT,                   -- nabidka|potvrzena|vyroba|dokoncena|fakturovana|zrusena
  datum_zalozeni, datum_zahajeni, datum_dokonceni,
  cena_nabidka, cena_limit, poznamka
)

zaznamy_prace (
  id, zakazka_id, zamestnanec_id, stroj_id,
  start DATETIME, konec DATETIME,
  hodiny REAL, sazba_snapshot REAL
)
```

### Nákup

```sql
objednavky (
  id, cislo TEXT UNIQUE,       -- "0529/0526"
  typ TEXT,                    -- zakazka|stock
  zakazka_id,                  -- NULL pokud typ=stock
  dodavatel_id,
  stav TEXT,                   -- draft|potvrzena|odeslana|castecne_prijata|prijata|zrusena
  datum_vytvoreni, datum_potvrzeni, poznamka
)

objednavky_polozky (id, objednavka_id, popis, mnozstvi, jednotka, cena_ks)
```

### Dodávky

```sql
dodavky (
  id, cislo_dl TEXT,           -- číslo dodacího listu od dodavatele
  objednavka_id,               -- NULL = dodávka bez objednávky
  dodavatel_id,
  datum_prijmu,
  stav TEXT,                   -- prijata|zkontrolovana|zauctovaana
  poznamka
)
-- Zadání čísla DL: primárně ručně, sekundárně scan čárového kódu na DL

dodavky_polozky (
  id, dodavka_id, popis,
  mnozstvi_objednano, mnozstvi_prijato,
  jednotka, cena_ks
)
```

### Fakturace

```sql
faktury (
  id, cislo TEXT UNIQUE,       -- "2608/0256"
  typ TEXT,                    -- vystupni|vstupni
  partner_id,                  -- zakaznici.id nebo dodavatele.id
  partner_typ TEXT,            -- zakaznik|dodavatel
  datum_vystaveni, datum_splatnosti, datum_zaplaceni,
  stav TEXT,                   -- draft|vystavena|odeslana|zaplacena|po_splatnosti|stornovana
  variabilni_symbol, poznamka
)

faktury_polozky (id, faktura_id, popis, mnozstvi, jednotka, cena_ks, sazba_dph)
-- DPH sazby: 0 | 12 | 21

faktury_vazby (faktura_id, typ_vazby TEXT, ref_id)
-- typ_vazby: "zakazka" nebo "dodavka"
```

---

## Integrace s Výdejnou Nisaform

Výdejna = klient, NISAVEROS = server. Pouze 4 endpointy:

```
GET  /api/objednavky/next-number        ← Výdejna: dej volné číslo objednávky
POST /api/objednavky                    ← Výdejna: vytvoř objednávku (typ: stock)
POST /api/objednavky/:id/naskladneni    ← Výdejna: zboží přijato, aktualizuj stav
GET  /api/objednavky/:id                ← Výdejna: ověř stav objednávky
```

### Kompatibilita dodavatelů (Výdejna ↔ NISAVEROS)

Před propojením systémů:
1. Exportovat dodavatele z Výdejny (`muj_sklad.json` → pole `suppliers`)
2. Importovat do NISAVEROS tabulky `dodavatele` (master)
3. Výdejna bude aktualizována — uloží `nisaveros_dodavatel_id` místo lokálního ID
4. Párování: primárně přes IČO, sekundárně normalizací názvu
5. Ruční ověření před spuštěním

Stroje: master v NISAVEROS, Výdejna odkazuje na `stroj_id` read-only.

---

## Toky dokladů

```
VÝROBA → výstupní faktura zákazníkovi:
  Zakázka → Záznamy práce → [Ekonom] Výstupní faktura
  faktury_vazby.typ_vazby = "zakazka"

NÁKUP NA ZAKÁZKU → vstupní faktura od dodavatele:
  Zakázka → Objednávka (typ:zakazka) → Dodávka → Vstupní faktura
  faktury_vazby.typ_vazby = "dodavka"

NÁKUP DO SKLADU (Výdejna):
  Výdejna → /api/objednavky/next-number
           → POST /api/objednavky (typ:stock)
           → POST /api/objednavky/:id/naskladneni
           → [Ekonom] Vstupní faktura ← číslo DL
```

---

## Faktury — výstup

- HTML tisk z prohlížeče (Ctrl+P → tisk do PDF jako virtuální tiskárna)
- Email zákazníkovi: manuálně — uložit PDF, odeslat emailem mimo systém
- Přímé odesílání emailu (SMTP): odloženo na pozdější fázi

## Live update (více PC)

Polling — stejný vzor jako Výdejna Nisaform.
Interval: 5–10 s pro dílnu (aktivní stavy), 30 s pro ostatní moduly.
WebSocket/SSE není plánován.

---

## Fáze vývoje a stav

### Fáze 0 — Setup projektu
- [ ] Vytvořit adresář `C:\_Projekty\_NISAVEROS`
- [ ] Inicializovat git repozitář, remote `https://github.com/Kynuska/_NISAVEROS`
- [ ] `npm init`, závislosti: `express`, `better-sqlite3`, `cors`, `cookie-session`
- [ ] `db.js` — všechny tabulky, seed číselníků
- [ ] `auth.js` — přihlášení, session, middleware rolí
- [ ] `server.js` — kostra, `GET /api/ping`
- [ ] `install.bat`, `start-server.vbs`
- [ ] `start.md`, `SESSION.md`

### Fáze 1 — Zakázky a výroba
- [ ] Routes: `zakazky`, `zamestnanci`, `stroje`, `zakaznici`
- [ ] Frontend dílna: clock-in/out (PIN + čtečka → zakázka + stroj)
- [ ] Frontend technologie: správa zakázek, záznamy práce, přehledy
- [ ] Migrace dat z `zakazky-data.json`

### Fáze 2 — Objednávky a dodávky
- [ ] Routes: `objednavky`, `dodavky`
- [ ] Frontend ekonom: tvorba objednávek, příjem dodávek
- [ ] Zadání čísla DL: ruční pole + volitelný scan
- [ ] Integrační endpointy pro Výdejnu (4 endpointy)

### Fáze 3 — Fakturace
- [ ] Routes: `faktury`, `faktury_polozky`, `faktury_vazby`, čítače
- [ ] Frontend ekonom: vystavování faktur, splatnosti
- [ ] Výstupní faktura ← zakázka (hodiny × sazba + materiál)
- [ ] Vstupní faktura ← dodávka (párování přes číslo DL)
- [ ] HTML tisk (Ctrl+P → PDF)

### Fáze 4 — Výdejna integrace
- [ ] Synchronizace dodavatelů (export Výdejna → import NISAVEROS)
- [ ] Aktualizace Výdejny: uložení `nisaveros_dodavatel_id`
- [ ] Implementace 4 integračních endpointů
- [ ] Testování propojení

### Fáze 5 — Deployment
- [ ] Windows Service (NSSM)
- [ ] Autostart, firewall port 3001
- [ ] Denní záloha `nisaveros.sqlite` do `Zaloha/`
- [ ] Dokumentace IT: instalace, záloha, obnova

---

## Rozhodnuté otázky (archiv)

| Otázka | Rozhodnutí |
|---|---|
| Email faktur | Manuálně — PDF z tisku, odeslat mimo systém. SMTP odloženo. |
| Live update dílna/více PC | Polling (5–10 s dílna, 30 s ostatní). WebSocket/SSE není plánován. |
| Scan dodacího listu | Primárně ruční zadání čísla DL, scan jako volitelná možnost. |
| Faktury PDF | HTML tisk → virtuální tiskárna → PDF. Přímé PDF odloženo. |
| Dodavatelé kompatibilita | NISAVEROS = master. Párování přes IČO, pak název. Výdejna bude aktualizována v Fázi 4. |
