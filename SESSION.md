# SESSION — Aktuální stav projektu

> Tento soubor se přepisuje při každém „konec dne".  
> Zachycuje stav na konci poslední session.

---

## Datum poslední session
2026-05-29

## Fáze
Fáze 1 — HTML frontend (aktivní, ale vývoj pozastaven — přechod na NISAVEROS)

## Aktivní soubor
`zakazky_v1.4-dark-json-pravidla.html`

## Co bylo v poslední session
- Navržena architektura nového systému NISAVEROS (nový projekt)
- Rozhodnutí: Výdejna Nisaform zůstává separátní (režijní sklad), propojení pouze přes 4 API endpointy
- Vytvořeny plánovací soubory v kořeni Zakázek: `NISAVEROS-PLAN.md`, `NISAVEROS-start.md`, `NISAVEROS-SESSION.md`
- **Fáze 0 NISAVEROS dokončena** — projekt rozjetý v `C:\_Projekty\_NISAVEROS`

## Co je příště na řadě
- **Zakázky v1.4 se dále nevyvíjí** — data budou migrována do NISAVEROS v Fázi 1
- Případné opravy/záplaty v Zakázkách jsou přijatelné do doby migrace
- Nový vývoj probíhá v `C:\_Projekty\_NISAVEROS` — spustit Claude Code tam

## Otevřené body / problémy
- NISAVEROS GitHub repozitář `_NISAVEROS` zatím neexistuje — uživatel vytvoří ručně:
  ```
  git remote add origin https://github.com/Kynuska/_NISAVEROS.git
  git push -u origin master
  ```
- Plánovací soubory `NISAVEROS-*.md` v kořeni Zakázek lze po ověření smazat (jsou zkopírovány do NISAVEROS)

## Poznámky
NISAVEROS: `C:\_Projekty\_NISAVEROS` | server port `:3001` | `proved start.md` pro zahájení session
