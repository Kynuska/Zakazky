# CLAUDE.md — Projekt ZAKÁZKY

Tento soubor se načítá automaticky při každém spuštění Claude Code v tomto workspace.

## Popis projektu

Webová HTML aplikace pro sběr dat z čteček čárových kódů na lokální síti firmy.
- Uložena na firemním serveru, přístupná z více PC přes prohlížeč
- Vstup primárně přes čtečky čárových kódů (simulují klávesnici)
- Komunikace v rámci lokální sítě (LAN), bez internetu

## Aktivní soubor

Jediný soubor v kořeni repozitáře s prefixem `zakazky_v*.html`.

Při každé session ověř skutečný název: `ls zakazky_v*.html`

## Pravidla před každou úpravou HTML

**Nejdřív mikro-záloha**, pak úprava:

```
cp zakazky_vX.Y-popis.html Zaloha/zakazky_vX.Y-popis-YYYYMMDD-HHMMSS-popis-zmeny.html
```

Aktivní soubor v kořeni se **nesmí přejmenovávat** během běžné editace.

## Příkaz „milník"

1. Vytvořit novou aktivní verzi v kořeni (navýšit verzi, zachovat popis)
2. Předchozí aktivní soubor přesunout do `Zaloha/`
3. Smazat mikro-zálohy s timestampem (`-YYYYMMDD-HHMMSS-`) ve `Zaloha/`
4. Archivní soubory bez timestampu **nemazat**

Formát: `zakazky_vX.Y-popis.html` (příklad: `v1.0-zaklad.html` → `v1.1-...`)

## Git identita (pouze lokálně, bez --global)

```
git config user.name "Aleš Hnyk"
git config user.email ales.hnyk@centrum.cz
```

## Git workflow

**Remote:** `https://github.com/Kynuska/Zakazky.git`

**Začátek session:**
```
git fetch origin
git status -sb
```

**Konec dne** — při příkazu „konec dne" proveď tento postup:

1. `git fetch origin` + `git status -sb` — zjistit stav
2. Pokud jsou změny: navrhnout zprávu commitu podle diffu, pak:
   ```
   git add -A
   git commit -m "vX.Y: popis změn"
   git push origin master
   ```
3. Zkontrolovat, že aktivní `zakazky_v*.html` je v kořeni a `Zaloha/` je v pořádku
4. Krátce shrnout: co bylo commitnuto, zda je push OK, případné otevřené body

Destruktivní příkazy (`reset --hard`, `clean -fd`) — **pouze po výslovném souhlasu** uživatele.

## Struktura repozitáře

- `Zaloha/` — archivní verze a mikro-zálohy
- `Zdroje/` — podkladové soubory (xlsx, vzory, exporty)
- `CLAUDE.md` — tento soubor
