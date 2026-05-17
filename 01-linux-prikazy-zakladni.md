# Základní Linux příkazy

## Úvod

Tato sekce pokrývá nejdůležitější příkazy pro každodenní práci v Linuxové příkazové řádkě. Každý příkaz je popsán s příkladem a očekávaným výstupem. Cílem je rychlá reference pro studenty operačních systémů.

---

## Soubory a adresáře

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `ls` | Seznam souborů v adresáři | `ls -la` | `total 48`<br>`drwxr-xr-x 5 user group 4096 Jan 1 12:00 .`<br>`-rw-r--r-- 1 user group 1024 Jan 1 12:00 file.txt` |
| `cd` | Změna aktuálního adresáře | `cd /home/user/Documents` | (žádný výstup) |
| `pwd` | Vytisknout aktuální adresář | `pwd` | `/home/user/Documents` |
| `mkdir` | Vytvořit nový adresář | `mkdir projekt` | (žádný výstup) |
| `rmdir` | Smazat prázdný adresář | `rmdir projekt` | (žádný výstup) |
| `cp` | Zkopírovat soubor | `cp file.txt backup.txt` | (žádný výstup) |
| `mv` | Posunout nebo přejmenovat soubor | `mv file.txt document.txt` | (žádný výstup) |
| `rm` | Smazat soubor | `rm document.txt` | (žádný výstup) |

### Užitečné příznaky

| Příkaz | Příznak | Význam |
|--------|---------|--------|
| `ls` | `-l` | Detailní seznam (long format) |
| `ls` | `-a` | Zobrazení skrytých souborů |
| `ls` | `-h` | Lidsky čitelné velikosti |
| `cp` | `-r` | Rekurzivní kopírování |
| `rm` | `-r` | Rekurzivní mazání |
| `rm` | `-f` | Nucené mazání (bez dotazu) |

---

## Textové soubory

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `cat` | Spojit a vypsat obsah souboru | `cat file.txt` | `Toto je obsah souboru.` |
| `echo` | Vypsat text na výstup | `echo "Ahoj světe"` | `Ahoj světe` |
| `head` | Zobrazit začátek souboru | `head -n 5 file.txt` | (prvních 5 řádků) |
| `tail` | Zobrazit konec souboru | `tail -n 5 file.txt` | (posledních 5 řádků) |
| `wc` | Počítat řádky, slova, znaky | `wc file.txt` | `15 42 250 file.txt` |

### Příklady použití

```bash
# Zapsat text do souboru
echo "první řádek" > output.txt

# Přidat text do souboru
echo "druhý řádek" >> output.txt

# Zřetězení příkazů
cat file1.txt file2.txt > combined.txt

# Sledovat log v reálném čase
tail -f /var/log/syslog
```

---

## Síťová připojení

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `ping` | Otestovat síťovou spojitost | `ping google.com` | `64 bytes from google.com: icmp_seq=1 ttl=113 time=15.2 ms` |
| `wget` | Stáhnout soubor z webu | `wget https://example.com/file.zip` | `file.zip           100%[============>] 1.02M  2.15MB/s` |
| `curl` | Poslat HTTP požadavek | `curl https://example.com` | `<!DOCTYPE html><html>...` |

### Příklady použití

```bash
# Omezit počet pingů
ping -c 4 google.com

# Stáhnout soubor do konkrétního adresáře
wget -O /tmp/download.zip https://example.com/file.zip

# Získat pouze hlavičku HTTP odpovědi
curl -I https://example.com

# Stáhnout soubor s tichým výstupem
curl -o output.json https://api.example.com/data
```

---

## Systémové informace

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `uname` | Informace o jádře systému | `uname -a` | `Linux hostname 5.15.0 #1 SMP x86_64 GNU/Linux` |
| `hostname` | Názov stroje | `hostname` | `my-laptop` |
| `free` | Volná paměť (RAM) | `free -h` | `              total   used   free`<br>`Mem:           16Gi    8Gi    8Gi` |
| `df` | Volné místo na disku | `df -h` | `Filesystem      Size  Used Avail Use%`<br>`/dev/sda1        50G   30G   20G  60%` |

### Příklady použití

```bash
# Pouze verze jádra
uname -r
# 5.15.0-generic

# Pouze název stroje
hostname -s
# my-laptop

# Podrobnější informace o paměti
free -m
#               total     used     free
# Mem:          16384     8192     8192

# Zobrazení všech disků
df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        50G   30G   20G  60% /
```

---

## Shrnutí

- **Soubory a adresáře** jsou základ každé práce v Linuxu. Pamatujte rozdíly mezi `cp` (kopírování) a `mv` (posun/přejmenování).
- **Textové soubory** se nejčastěji čtou pomocí `cat`, `head`, a `tail`. Příkaz `wc` je rychlý způsob, jak zjistit délku souboru.
- **Síťová připojení** se testují pomocí `ping`. Soubory se stahují přes `wget` nebo `curl`.
- **Systémové informace** získáte rychle přes `uname`, `hostname`, `free`, a `df`.

Tyto příkazy tvoří základ pro práci v Linuxové příkazové řádce. Ovládněte je nejdřív a pak přejděte k rozšířeným příkazům.

---

➡️ [Zpět na přehled](README.md)
