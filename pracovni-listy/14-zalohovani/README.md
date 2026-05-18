# Pracovní list: Zálohování a obnova

> Procvičení archivace, komprese, synchronizace a vytváření bitových kopií v Linuxu.

---

## Zadání

### Úkol 1: Archivace a komprese

1. Vytvoř adresář `backup_test` se 3 soubory (`soubor1.txt`, `soubor2.txt`, `soubor3.txt`)
2. Vytvoř archiv `archive.tar` obsahující celý adresář
3. Zkomprimuj archiv pomocí gzip (`archive.tar.gz`)
4. Vytvoř rovnou komprimovaný archiv (`tar -czf`)
5. Vypiš obsah archivu bez extrakce (`tar -tf`)
6. Extrahuj archiv do adresáře `obnoveno/`
7. Porovnej velikosti: originál vs gzip vs bz2 vs xz

### Úkol 2: Synchronizace rsync

1. Vytvoř adresář `zdroj/` s několika soubory
2. Vytvoř adresář `cil/`
3. Synchronizuj `zdroj/` do `cil/` pomocí `rsync -av`
4. Přidej nový soubor do `zdroj/` a spusť rsync znovu — sleduj, že zkopíruje jen nový soubor
5. Smaž soubor v `zdroj/` a spusť rsync s `--delete` — co se stane?
6. Nejdřív použij `--dry-run` — proč je to užitečné?

### Úkol 3: dd a integrity

1. Vytvoř soubor `vstup.txt` s libovolným textem
2. Vytvoř jeho bitovou kopii: `dd if=vstup.txt of=kopie.txt`
3. Ověř, že jsou soubory identické (`diff` nebo `sha256sum`)
4. Vytvoř obraz disku: `dd if=/dev/zero of=disk.img bs=1M count=10`
5. Zjisti hash obrazu: `sha256sum disk.img`
6. Jaké je 3-2-1 pravidlo zálohování?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-2. Příprava a archivace
mkdir backup_test
echo "data1" > backup_test/soubor1.txt
echo "data2" > backup_test/soubor2.txt
echo "data3" > backup_test/soubor3.txt
tar -cf archive.tar backup_test/

# 3-4. Komprese
gzip archive.tar  # → archive.tar.gz
# Nebo rovnou:
tar -czf archive.tar.gz backup_test/

# 5. Výpis obsahu
tar -tf archive.tar.gz

# 6. Extrakce
mkdir obnoveno
tar -xzf archive.tar.gz -C obnoveno/

# 7. Porovnání velikostí
du -sh backup_test/ archive.tar.gz
# zkusit: tar -cjf test.tar.bz2 backup_test
# zkusit: tar -cJf test.tar.xz backup_test
```

### Úkol 2 — Řešení

```bash
# 1-3. Základní rsync
mkdir zdroj cil
echo "soubor1" > zdroj/soubor1.txt
echo "soubor2" > zdroj/soubor2.txt
rsync -av zdroj/ cil/

# 4. Pouze nové soubory
echo "soubor3" > zdroj/soubor3.txt
rsync -av zdroj/ cil/
# Zkopíruje jen soubor3.txt

# 5. --delete
rm zdroj/soubor1.txt
rsync -av --delete zdroj/ cil/
# Smaže soubor1.txt i v cíli

# 6. --dry-run = suchý běh, jen náhled bez provedení změn
rsync -av --dry-run --delete zdroj/ cil/
```

### Úkol 3 — Řešení

```bash
# 1-3. dd kopie
echo "Testovaci data" > vstup.txt
dd if=vstup.txt of=kopie.txt
diff vstup.txt kopie.txt  # žádný výstup = stejné
sha256sum vstup.txt kopie.txt  # stejný hash

# 4. Obraz disku
dd if=/dev/zero of=disk.img bs=1M count=10
# Vytvoří 10MB soubor naplněný nulami

# 5. Hash
sha256sum disk.img

# 6. 3-2-1 pravidlo:
# 3 kopie dat
# 2 různá média (např. disk + cloud)
# 1 kopie mimo lokalitu (off-site)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš vytvořit a extrahovat archiv (tar + gzip/bz2/xz)
- [ ] Dokážeš synchronizovat adresáře pomocí rsync
- [ ] Víš, k čemu slouží `--delete` a `--dry-run` u rsync
- [ ] Rozumíš 3-2-1 pravidlu zálohování

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../14-zalohovani.md) · [Zpět na přehled](../../README.md)
