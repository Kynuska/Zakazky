# PLAN — Nisaform ERP (nový projekt)

## Účel

Nový samostatný systém pro výrobu, ekonomiku a provoz firmy Nisaform.
Nahrazuje a rozšiřuje Zakázky v1.4. Výdejna Nisaform zůstává separátní (režijní sklad).

---

## Architektura

### Struktura repozitáře

```
nisaform-erp/
├── server.js               ← Node.js/Express HTTP server
├── db.js                   ← SQLite inicializace, tabulky, migrace
├── auth.js                 ← session autentizace, middleware pro role
├── routes/
│   ├── zakazky.js
│   ├── zamestnanci.js
│   ├── stroje.js
│   ├── zakaznici.js
│   ├── objednavky.js
│   ├── dodavky.js
│   └── faktury.js
├── _dev/                   ← build pipeline (stejný vzor jako Výdejna)
│   ├── build.js
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
├── public/                 ← build output (statické soubory)
├── data/
│   └── erp.sqlite
├── Zaloha/
├── install.bat
├── start-server.vbs
├── package.json
└── start.md
```

### Serverový port

- Nový ERP: `:3001`
- Výdejna Nisaform: `:3000` (beze změny)

---

## Role a přístup

| Role | PC | Přístup |
|---|---|---|
| `dilna` | PC dílna | Clock-in/out, čtečka čárových kódů |
| `technolog` | PC technologie | Správa zakázek, záznamy práce, přehledy výroby |
| `ekonom` | PC ekonom | Objednávky, dodávky, fakturace |
| `admin` | libovolné | Vše + správa uživatelů, číselníků, rolí |

Auth: session cookie, přihlášení jménem + PINem (vhodné pro dílnu) nebo heslem.

---

## Číslování dokladů

| Doklad | Formát | Příklad | Popis |
|---|---|---|---|
| Zakázka | `RR-NNNN` | `26-0025` | 2-místný rok + 4-místné pořadové číslo; reset k 1. 1. |
| Objednávka | `NNNN/MMRR` | `0529/0526` | 4-místné průběžné číslo / 2-místný měsíc + rok |
| Faktura | `RRMM/NNNN` | `2908/0256` | 2-místný rok + 2-místný měsíc / 4-místné pořadové číslo |

Čítače uloženy v DB tabulce `sekvence` (typ, rok, posledni_cislo).
Zakázky a faktury: reset k 1. 1. Objednávky: průběžné (NNNN roste bez resetu).

---

## Databázové schéma (SQLite)

### Číselníky a konfigurace

```sql
stroje (
  id INTEGER PK,
  nazev TEXT,
  sazba_kc_hod REAL,
  aktivni INTEGER DEFAULT 1
)

zamestnanci (
  id INTEGER PK,
  jmeno TEXT,
  carovy_kod TEXT UNIQUE,
  pin TEXT,
  role TEXT,          -- dilna|technolog|ekonom|admin
  aktivni INTEGER DEFAULT 1
)

zakaznici (
  id INTEGER PK,
  nazev TEXT,
  ico TEXT,
  dic TEXT,
  adresa TEXT,
  email TEXT,
  telefon TEXT
)

dodavatele (
  id INTEGER PK,
  nazev TEXT,
  ico TEXT,
  dic TEXT,
  adresa TEXT,
  email TEXT
)

sekvence (
  typ TEXT PK,        -- zakazka|objednavka|faktura_vystupni|faktura_vstupni
  rok INTEGER,
  posledni_cislo INTEGER
)
```

### Výroba

```sql
zakazky (
  id INTEGER PK,
  cislo TEXT UNIQUE,          -- "26-0025"
  zakaznik_id INTEGER,
  nazev TEXT,
  popis TEXT,
  stav TEXT,                  -- nabidka|potvrzena|vyroba|dokoncena|fakturovana|zrusena
  datum_zalozeni TEXT,
  datum_zahajeni TEXT,
  datum_dokonceni TEXT,
  cena_nabidka REAL,
  cena_limit REAL,
  poznamka TEXT
)

zaznamy_prace (
  id INTEGER PK,
  zakazka_id INTEGER,
  zamestnanec_id INTEGER,
  stroj_id INTEGER,
  start TEXT,                 -- ISO datetime
  konec TEXT,
  hodiny REAL,
  sazba_snapshot REAL         -- sazba stroje v okamžiku záznamu
)
```

### Nákup

```sql
objednavky (
  id INTEGER PK,
  cislo TEXT UNIQUE,          -- "0529/0526"
  typ TEXT,                   -- zakazka|stock
  zakazka_id INTEGER,         -- NULL pokud typ=stock
  dodavatel_id INTEGER,
  stav TEXT,                  -- draft|potvrzena|odeslana|castecne_prijata|prijata|zrusena
  datum_vytvoreni TEXT,
  datum_potvrzeni TEXT,
  poznamka TEXT
)

objednavky_polozky (
  id INTEGER PK,
  objednavka_id INTEGER,
  popis TEXT,
  mnozstvi REAL,
  jednotka TEXT,
  cena_ks REAL
)
```

### Dodávky

```sql
dodavky (
  id INTEGER PK,
  cislo_dl TEXT,              -- číslo dodacího listu od dodavatele
  objednavka_id INTEGER,      -- NULL = dodávka bez objednávky
  dodavatel_id INTEGER,
  datum_prijmu TEXT,
  stav TEXT,                  -- prijata|zkontrolovana|zauctovaana
  poznamka TEXT
)

dodavky_polozky (
  id INTEGER PK,
  dodavka_id INTEGER,
  popis TEXT,
  mnozstvi_objednano REAL,
  mnozstvi_prijato REAL,
  jednotka TEXT,
  cena_ks REAL
)
```

### Fakturace

```sql
faktury (
  id INTEGER PK,
  cislo TEXT UNIQUE,          -- "2908/0256"
  typ TEXT,                   -- vystupni|vstupni
  partner_id INTEGER,         -- zakaznik_id nebo dodavatel_id
  partner_typ TEXT,           -- zakaznik|dodavatel
  datum_vystaveni TEXT,
  datum_splatnosti TEXT,
  datum_zaplaceni TEXT,
  stav TEXT,                  -- draft|vystavena|odeslana|zaplacena|po_splatnosti|stornovana
  variabilni_symbol TEXT,
  poznamka TEXT
)

faktury_polozky (
  id INTEGER PK,
  faktura_id INTEGER,
  popis TEXT,
  mnozstvi REAL,
  jednotka TEXT,
  cena_ks REAL,
  sazba_dph REAL              -- 0|12|21
)

faktury_vazby (
  faktura_id INTEGER,
  typ_vazby TEXT,             -- zakazka|dodavka
  ref_id INTEGER              -- zakazky.id nebo dodavky.id
)
```

---

## Integrace s Výdejnou Nisaform

Výdejna volá nový ERP na 4 endpointech (Výdejna = klient, ERP = server):

```
GET  /api/objednavky/next-number        ← Výdejna: dej mi volné číslo objednávky
POST /api/objednavky                    ← Výdejna: vytvoř objednávku (typ: stock)
POST /api/objednavky/:id/naskladneni    ← Výdejna: zboží přišlo (aktualizuj stav)
GET  /api/objednavky/:id                ← Výdejna: ověř stav objednávky
```

Stroje jsou master v novém ERP. Výdejna odkazuje na stroj přes `stroj_id` (read-only).

---

## Toky dokladů

```
VÝROBA (výstupní faktura zákazníkovi):
Zakázka → Záznamy práce → Výstupní faktura
                            └── faktura_vazby.typ_vazby = "zakazka"

NÁKUP NA ZAKÁZKU (vstupní faktura od dodavatele):
Zakázka → Objednávka (typ: zakazka) → Dodávka → Vstupní faktura
                                          └── faktura_vazby.typ_vazby = "dodavka"

NÁKUP DO SKLADU (přes Výdejnu):
Výdejna → next-number API → Objednávka (typ: stock) → Dodávka → Vstupní faktura
                                 ← naskladnení zpět do Výdejny
```

---

## Fáze vývoje

### Fáze 0 — Setup projektu
- [ ] Nový repozitář `Nisaform-ERP` na GitHubu
- [ ] `npm init`, závislosti: `express`, `better-sqlite3`, `cors`, `cookie-session`
- [ ] `db.js` — vytvoření všech tabulek, seed číselníků
- [ ] `auth.js` — přihlášení PIN/heslo, session cookie, middleware role
- [ ] `server.js` — základní kostra, `GET /api/ping`
- [ ] `install.bat`, `start-server.vbs`

### Fáze 1 — Zakázky a výroba
- [ ] REST API: `zakazky`, `zamestnanci`, `stroje`, `zakaznici`
- [ ] Frontend dílna: clock-in/out přes čtečku (PIN + čárový kód)
- [ ] Frontend technologie: správa zakázek, záznamy práce, přehledy
- [ ] Migrace dat z `zakazky-data.json`

### Fáze 2 — Objednávky a dodávky
- [ ] REST API: `objednavky`, `objednavky_polozky`, `dodavky`, `dodavky_polozky`
- [ ] Frontend ekonom: tvorba a správa objednávek
- [ ] Frontend ekonom: příjem dodávek, párování s objednávkami
- [ ] Integrační endpointy pro Výdejnu (4 endpointy)

### Fáze 3 — Fakturace
- [ ] REST API: `faktury`, `faktury_polozky`, `faktury_vazby`, sekvence čísel
- [ ] Frontend ekonom: vystavování výstupních faktur ze zakázek
- [ ] Frontend ekonom: evidence vstupních faktur, párování s dodávkami
- [ ] Přehled splatností, stav plateb

### Fáze 4 — Deployment
- [ ] Windows Service (NSSM)
- [ ] Autostart při startu Windows, firewall port 3001
- [ ] Automatická záloha `erp.sqlite` (denní kopie do `Zaloha/`)
- [ ] Dokumentace pro IT: instalace, záloha, obnova

---

## Otevřené otázky

- Scan dodacího listu: čárový kód na DL (automatické spárování), nebo ruční zadání čísla DL?
- Výstupní faktura: generovat tisknutelné PDF, nebo pouze HTML tisk z prohlížeče?
- Přístup z více PC najednou: stačí polling (jako Výdejna), nebo SSE/WebSocket?
- Sdílení dodavatelů s Výdejnou: vlastní tabulka v ERP + ruční synchronizace, nebo ERP = master a Výdejna volá API?
