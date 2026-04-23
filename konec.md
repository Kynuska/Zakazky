# KONEC — Instrukce pro ukončení session

> Tento soubor čte Claude při příkazu „konec" nebo „konec dne".

---

## Co Claude udělá při příkazu „konec"

1. Zeptat se (nebo shrnout z kontextu):
   - Co bylo dnes hotovo?
   - Co zůstalo nedokončeno?
   - Jsou nějaké otevřené problémy nebo otázky?

2. Aktualizovat `SESSION.md`:
   - Datum dnešní session
   - Aktuální fáze a aktivní soubor
   - Co bylo hotovo
   - Co je příště na řadě
   - Otevřené body

3. Aktualizovat `PLAN.md`:
   - Zaškrtnout splněné položky `[ ]` → `[x]`
   - Doplnit nové položky pokud vznikly

4. Sdělit uživateli:
   - Shrnutí co bylo uděláno
   - Co je logicky příště
   - Připomínka: **zkontrolovat a commitnout v GitHub Desktop**

## Co Claude NEDĚLÁ při konci
- Neprovádí git příkazy (GitHub = uživatel, GitHub Desktop)
- Nemaže žádné soubory
- Nezačíná nové úkoly

## Formát zápisu do SESSION.md

```markdown
## Datum poslední session
YYYY-MM-DD

## Fáze
Fáze X — název

## Aktivní soubor
zakazky_vX.Y-popis.html

## Co bylo v poslední session
- bod 1
- bod 2

## Co je příště na řadě
- bod 1
- bod 2

## Otevřené body / problémy
- bod (nebo "žádné")

## Poznámky
volný text
```
