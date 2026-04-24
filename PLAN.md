# PLAN — Systém zakázek

## Architektura (dohodnutá)

```
Fáze 1  →  HTML single-file (aktuální stav)
Fáze 2  →  HTML frontend + Node.js/Express + SQLite backend (jeden server PC)
Fáze 3  →  Windows Service (autostart serveru), zástupce na klientských PC
```

Klienti přistupují přes prohlížeč: `http://[IP serveru]:3000`  
GitHub spravuje uživatel sám přes GitHub Desktop.

---

## Fáze 1 — Dokončení HTML frontendu

> Cíl: frontend je feature-complete před přechodem na backend.

### Funkce
- [x] Tmavý motiv
- [x] Terminál — 3-krokový workflow (zakázka → zaměstnanec → stroj)
- [x] Clock-in / clock-out / přepnutí zakázky
- [x] Kontrola obsazenosti stroje (pravidla viceZamestnancu / viceZakazek)
- [x] Sidebary + drag & drop
- [x] Seznam pracujících s elapsed časem + tlačítko Konec
- [x] Záznamy práce — ruční zadání v detailu zakázky
- [x] Přehled hodin / zaměstnanec / stroj
- [x] Náklady zakázky (hodiny × sazba stroje), varování při překročení ceny
- [x] JSON úložiště přes FSA API + IndexedDB persistence handle
- [x] localStorage fallback
- [x] Verze z názvu souboru
- [ ] *(doplnit dle potřeby)*

---

## Fáze 2 — Backend (Node.js + SQLite)

### 2a — Příprava projektu
- [ ] Inicializovat `backend/` složku (`npm init`)
- [ ] Závislosti: `express`, `better-sqlite3`, `cors`
- [ ] Struktura: `server.js`, `db.js`, `routes/`

### 2b — Databázové schéma (SQLite)
- [ ] Tabulka `zakazky`
- [ ] Tabulka `zakaznici`
- [ ] Tabulka `typy`
- [ ] Tabulka `zamestnanci`
- [ ] Tabulka `stroje`
- [ ] Tabulka `zaznamy` (záznamy práce, cizí klíče)
- [ ] Migrace / seed ze stávajícího `zakazky-data.json`

### 2c — REST API
- [ ] `GET/POST/PUT/DELETE /api/zakazky`
- [ ] `GET/POST/PUT/DELETE /api/zakaznici`
- [ ] `GET/POST/PUT/DELETE /api/typy`
- [ ] `GET/POST/PUT/DELETE /api/zamestnanci`
- [ ] `GET/POST/PUT/DELETE /api/stroje`
- [ ] `GET/POST /api/zaznamy` + `PUT /api/zaznamy/:id/konec`
- [ ] `GET /api/working` — aktivní sessions

### 2d — Migrace frontendu
- [ ] Nahradit FSA/IndexedDB vrstvu za `fetch()` volání na API
- [ ] Real-time refresh pracujících (polling nebo SSE)
- [ ] Error handling při nedostupném serveru

### 2e — Build (volitelné)
- [ ] `build.js` — spojení JS/CSS do jednoho HTML (pokud frontend přeroste jeden soubor)
- [ ] Automatické vkládání verze/datumu do buildu

### 2f — Testování v LAN
- [ ] Spustit server na jednom PC, přistoupit z druhého
- [ ] Ověřit souběžný přístup (více klientů najednou)

---

## Fáze 3 — Windows deployment

- [ ] Zabalit server do `.exe` pomocí `pkg` (node binary)
- [ ] Nainstalovat jako Windows Service (`node-windows` nebo NSSM)
- [ ] Autostart při startu Windows
- [ ] Firewall — povolit port 3000 v LAN
- [ ] Zástupce na ploše klientských PC s URL serveru
- [ ] Dokumentace pro IT: instalace, backup SQLite souboru

---

## Otevřené otázky

- Má server vyžadovat přihlášení (login), nebo je LAN dostatečná ochrana?
- Má existovat role admin vs. terminálový uživatel?
- Export dat (Excel/PDF) — požadavek?
- Zálohování SQLite — automatické, nebo manuální?
