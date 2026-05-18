# Pracovní list: Bash skriptování

> Procvičení tvorby bash skriptů — proměnné, podmínky, cykly, funkce a přesměrování.

---

## Zadání

### Úkol 1: Zálohovací skript

Napiš bash skript `backup.sh`, který:
1. Vytvoří adresář `backup/` (pokud neexistuje)
2. Zkopíruje všechny `.txt` soubory z aktuálního adresáře do `backup/`
3. Pokud žádný `.txt` soubor neexistuje, vypíše hlášku "Žádné soubory k zálohování"
4. Použije funkci `log()` pro výpis časově označených hlášek ve formátu `[2026-05-18 10:00] Zpráva`

### Úkol 2: Analyzátor logů

Vytvoř skript `analyze.sh`, který zpracuje soubor `access.log` (simulovaný):
- Spočítá počet řádků obsahujících "200 OK"
- Spočítá celkový počet řádků
- Vypíše procentuální úspěšnost
- Ošetří případ, kdy soubor neexistuje (vypíše chybu a skončí s kódem 1)

### Úkol 3: Hádání čísla

Napiš interaktivní skript `guess.sh`, který:
1. Vygeneruje náhodné číslo mezi 1 a 100 (`$RANDOM`)
2. Opakovaně se ptá uživatele na tip
3. Řekne, jestli je tip moc vysoký, nebo moc nízký
4. Při uhodnutí vypíše počet pokusů a skončí
5. Ošetří neplatný vstup (není číslo)

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
#!/bin/bash

log() {
    echo "[$(date '+%Y-%m-%d %H:%M')] $1"
}

mkdir -p backup

txt_files=(*.txt)
if [ -f "${txt_files[0]}" ]; then
    for file in *.txt; do
        cp "$file" backup/
        log "Zkopírován: $file"
    done
else
    log "Žádné soubory k zálohování"
fi
```

### Úkol 2 — Řešení

```bash
#!/bin/bash

log_file="access.log"

if [ ! -f "$log_file" ]; then
    echo "Chyba: Soubor $log_file neexistuje" >&2
    exit 1
fi

total=$(wc -l < "$log_file")
ok=$(grep -c "200 OK" "$log_file")

echo "Celkem řádků: $total"
echo "Úspěšných: $ok"

if [ "$total" -gt 0 ]; then
    percent=$((ok * 100 / total))
    echo "Úspěšnost: ${percent}%"
else
    echo "Log je prázdný."
fi
```

### Úkol 3 — Řešení

```bash
#!/bin/bash

secret=$((RANDOM % 100 + 1))
attempts=0

echo "Hádej číslo mezi 1 a 100!"

while true; do
    read -p "Tvůj tip: " guess

    if ! [[ "$guess" =~ ^[0-9]+$ ]]; then
        echo "Zadej platné číslo!"
        continue
    fi

    ((attempts++))

    if [ "$guess" -lt "$secret" ]; then
        echo "Moc nízké!"
    elif [ "$guess" -gt "$secret" ]; then
        echo "Moc vysoké!"
    else
        echo "Správně! Uhodl jsi na $attempts pokusů."
        break
    fi
done
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] `backup.sh` vytvoří `backup/` a zkopíruje `.txt` soubory
- [ ] `backup.sh` vypíše hlášku, pokud žádné `.txt` neexistují
- [ ] `analyze.sh` přežije neexistující soubor s chybovou hláškou a exit kódem 1
- [ ] `analyze.sh` správně spočítá procenta
- [ ] `guess.sh` správně reaguje na vysoký/nízký tip
- [ ] `guess.sh` ošetří nečíselný vstup

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../04-bash-skriptovani.md) · [Zpět na přehled](../../README.md)
