# Pracovní list: Rozšířené Linux příkazy

> Pipe, grep, sed, awk, find, xargs

---

## Zadání

### Úkol 1: grep — hledání v textu

Vytvoř soubor `data.txt` s obsahem:

```
apple 5
banana 3
cherry 8
apple 2
grape 1
```

Použij grep k:

1. Najdi všechny řádky obsahující "apple"
2. Spočti, kolik řádků obsahuje "apple"
3. Najdi řádky, které **ne** obsahují "apple" (použij `-v`)

### Úkol 2: awk — zpracování sloupců

Použij stejný soubor `data.txt`:

1. Vytiskni pouze první sloupec (názvy): `awk '{print $1}' data.txt`
2. Vytiskni řádky, kde hodnota ve druhém sloupci > 3
3. Spočti průměr hodnot ve druhém sloupci
4. Seřaď výstup podle druhého sloupce sestupně

### Úkol 3: find + xargs — dávkové operace

```bash
mkdir -p ~/pokus2 && cd ~/pokus2
touch soubor.txt soubor.log data.txt temp.log script.sh
```

1. Najdi všechny `.log` soubory a smaž je pomocí `find` + `-delete`
2. Najdi všechny `.txt` soubory a pomocí `xargs` zjisti jejich velikost
3. Najdi soubory změněné za posledních 5 minut (`-mmin -5`)

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
grep "apple" data.txt          # apple 5, apple 2
grep -c "apple" data.txt       # 2
grep -v "apple" data.txt       # banana 3, cherry 8, grape 1
```

### Úkol 2 — Řešení

```bash
awk '{print $1}' data.txt      # apple banana cherry apple grape
awk '$2 > 3' data.txt          # apple 5, cherry 8
awk '{sum+=$2; count++} END {print sum/count}' data.txt  # 3.8
awk '{print $2, $1}' data.txt | sort -rn  # 8 cherry, 5 apple, ...
```

### Úkol 3 — Řešení

```bash
find ~/pokus2 -name "*.log" -delete
find ~/pokus2 -name "*.txt" | xargs ls -l
find ~/pokus2 -mmin -5
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Všechny grep příkazy vrátily očekávaný výstup
- [ ] awk spočítal průměr = 3.8
- [ ] .log soubory po find -delete zmizely
- [ ] Rozumíš syntaxi `awk '{print $1}'` (co je $1, $2, $0)
- [ ] Víš, k čemu slouží xargs (předání výstupu jako argument)

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../02-linux-prikazy-rozirene.md) · [Zpět na přehled](../../README.md)
