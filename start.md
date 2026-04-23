# START — Instrukce pro zahájení session

> Tento soubor čte Claude na začátku každé session při příkazu „start".

---

## Co Claude udělá při příkazu „start"

1. Přečíst `SESSION.md` → sdělit kde jsme skončili a co je příště na řadě
2. Přečíst `PLAN.md` → sdělit aktuální fázi a nejbližší otevřené úkoly
3. Ověřit, že aktivní HTML soubor existuje: `ls zakazky_v*.html`
4. Zeptat se: **„Co budeme dnes dělat?"** nebo navrhnout logické pokračování

## Co Claude NEDĚLÁ při startu
- Neprovádí git příkazy (GitHub spravuje uživatel přes GitHub Desktop)
- Nespouští server ani build
- Nezačíná editovat kód bez zadání od uživatele

## Kontext projektu (stručně)
- Webová HTML aplikace pro sledování zakázek v malé firmě
- Vstup primárně přes čtečky čárových kódů (simulují klávesnici)
- Provoz: LAN intranet, budoucí backend Node.js + SQLite
- Vše v jednom HTML souboru, data v `zakazky-data.json`
- Zálohy do `Zaloha/`, archiv bez timestampů se nemaže
- Před každou editací HTML: mikro-záloha do `Zaloha/` se timestampem

## Pravidla práce (přehled z CLAUDE.md)
- Aktivní soubor: jediný `zakazky_v*.html` v kořeni — nepřejmenovávat
- Mikro-záloha před každou změnou: `cp zakazky_vX.Y-popis.html Zaloha/zakazky_vX.Y-popis-YYYYMMDD-HHMMSS-popis.html`
- Milník = nová verze v kořeni + předchozí do `Zaloha/` + smazat mikro-zálohy s timestampem
