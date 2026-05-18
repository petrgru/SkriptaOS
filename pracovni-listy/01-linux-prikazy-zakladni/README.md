# Pracovní list: Základní Linux příkazy

> Navigace, soubory, výstupní proudy, práva

---

## Zadání

### Úkol 1: Práce se soubory a adresáři

Vytvoř následující strukturu:

```bash
mkdir -p ~/pokus/dokumenty ~/pokus/obrazky
cd ~/pokus/dokumenty
echo "Hello World" > hello.txt
cp hello.txt hello_backup.txt
mv hello.txt pozdrav.txt
ls -la
```

Po spuštění:
1. Kolik souborů je v `~/pokus/dokumenty/`?
2. Jakou velikost má `pozdrav.txt`?
3. Smaž `hello_backup.txt` a ověř, že zmizel.

### Úkol 2: Práva souborů

Vytvoř soubor a změň jeho práva:

```bash
echo "#!/bin/bash\necho 'Ahoj'" > ~/pokus/script.sh
chmod 644 ~/pokus/script.sh
ls -l ~/pokus/script.sh
```

1. Jaká práva má soubor po `chmod 644`?
2. Může vlastník soubor spouštět? A ostatní?
3. Změň práva tak, aby soubor byl spustitelný (755). Ověř `ls -l`.

### Úkol 3: Výstupní proudy

```bash
ls /existuje 2>/dev/null
ls /neexistuje 2>/dev/null
echo "Ahoj" > ~/pokus/vystup.txt
cat ~/pokus/vystup.txt
```

1. Který příkaz vrátí chybu a proč?
2. Kam přesměrovává `2>/dev/null`?
3. Co udělá `>` oproti `>>`?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

1. V adresáři jsou **2 soubory**: `pozdrav.txt` a `hello_backup.txt` (po prvních krocích).
2. Velikost `pozdrav.txt` je 12 bajtů (obsahuje "Hello World\n").
3. Po `rm hello_backup.txt` a `ls` zůstane pouze `pozdrav.txt`.

### Úkol 2 — Řešení

1. Po `chmod 644`: `rw-r--r--` (vlastník čte+zapisuje, skupina čte, ostatní čtou).
2. Vlastník **nemůže** spouštět (chybí `x`). Ostatní také ne.
3. Po `chmod 755`: `rwxr-xr-x` (vlastník vše, skupina čte+spouští, ostatní čtou+spouští).

### Úkol 3 — Řešení

1. `ls /neexistuje` vrátí chybu — adresář neexistuje.
2. `2>/dev/null` přesměrovává **stderr** (chybový výstup) do `/dev/null` (zahodí ho).
3. `>` přepíše soubor, `>>` připojí na konec.

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Všechny příkazy proběhly bez chyby (exit code 0)
- [ ] Adresářová struktura odpovídá zadání
- [ ] Soubor `pozdrav.txt` obsahuje text "Hello World"
- [ ] Po `chmod 755` je soubor spustitelný (`-rwxr-xr-x`)
- [ ] Víš, jaký je rozdíl mezi stdout a stderr

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../01-linux-prikazy-zakladni.md) · [Zpět na přehled](../../README.md)
