# NISAVEROS — SESSION.md

> Přepisuje se při každém „konec". Zachycuje stav na konci poslední session.
> Agent čte tento soubor při startu — musí být vždy aktuální.

---

## Datum poslední session

2026-05-29

## Fáze

Fáze 0 — Setup projektu (nezahájeno)

## Co bylo v poslední session

- Navržena architektura NISAVEROS (multi-PC, role-based, sdílený backend)
- Rozhodnuto: Výdejna zůstává separátní, integrace pouze přes 4 API endpointy
- Vytvořen `PLAN.md`, `start.md`, `SESSION.md` jako šablony pro nový projekt
- Soubory zatím uloženy v `C:\_Projekty\_Zakazky\` — přesunout do `C:\_Projekty\_NISAVEROS\` při setupu

## Co je příště na řadě

Fáze 0 — Setup projektu:
1. Vytvořit adresář `C:\_Projekty\_NISAVEROS`
2. Přesunout `NISAVEROS-PLAN.md` → `PLAN.md`, `NISAVEROS-start.md` → `start.md`, `NISAVEROS-SESSION.md` → `SESSION.md`
3. `git init`, remote `https://github.com/Kynuska/_NISAVEROS`
4. `npm init` + závislosti (`express`, `better-sqlite3`, `cors`, `cookie-session`)
5. Napsat `db.js` (tabulky dle PLAN.md schématu)
6. Napsat `auth.js` (session, PIN, middleware rolí)
7. Napsat základní `server.js` + `GET /api/ping`

## Otevřené body / problémy

- Repozitář `_NISAVEROS` na GitHubu zatím neexistuje — vytvořit při setupu
- Otevřené otázky z PLAN.md: email (SMTP nebo manuálně?), polling vs SSE

## Poznámky

Šablony PLAN.md / start.md / SESSION.md vznikly v session Zakázky 2026-05-29.
Při přesunu do nového projektu: odstranit prefix `NISAVEROS-` z názvů souborů.
