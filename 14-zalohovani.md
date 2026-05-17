# Zálohování a obnova

> Backup and Restore in Linux

---

## Úvod

Zálohování je kritická součást správy každého systému. Tato sekce pokrývá nástroje pro archivaci, kompresi, synchronizaci a vytváření bitových kopií v Linuxu. Každý příkaz je doplněn příkladem a očekávaným výstupem.

---

## Archivace (tar)

`tar` je standardní nástroj pro vytváření a extrakci archivů. Původně byl určen pro pásky (Tape ARchive), dnes se používá univerzálně.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `tar -cf` | Vytvoření archivu | `tar -cf archive.tar /dir` | (žádný výstup) |
| `tar -xf` | Extrakce archivu | `tar -xf archive.tar` | (žádný výstup) |
| `tar -tf` | Výpis obsahu archivu | `tar -tf archive.tar` | `dir/file1.txt`<br>`dir/file2.txt` |
| `tar -czf` | Komprimace gzipem | `tar -czf backup.tar.gz /home` | (žádný výstup) |
| `tar -cjf` | Komprimace bzip2 | `tar -cjf backup.tar.bz2 /home` | (žádný výstup) |
| `tar -cJf` | Komprimace xz | `tar -cJf backup.tar.xz /home` | (žádný výstup) |
| `tar --exclude` | Archiv s vyloučením | `tar --exclude='*.log' -czf backup.tar.gz /var` | (žádný výstup) |
| `tar -xzf -C` | Extrakce do konkrétního adresáře | `tar -xzf archive.tar.gz -C /target/dir` | (žádný výstup) |

### Příklady použití

```bash
# Vytvoření gzip archivu celého /home
tar -czf home-backup.tar.gz /home

# Výpis obsahu bez extrakce
tar -tf home-backup.tar.gz

# Extrakce pouze .txt souborů
tar -xzf home-backup.tar.gz --wildcards '*.txt'

# Vyloučení více vzorů
tar --exclude='*.mp4' --exclude='*.iso' -czf data-backup.tar.gz /data
```

---

## Kompresní nástroje

Linux nabízí několik kompresních nástrojů, každý s jiným poměrem rychlosti a kompresního poměru.

| Nástroj | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `gzip` / `gunzip` | Komprese .gz (rychlá, nižší poměr) | `gzip file.txt` | (žádný výstup, vzniká `file.txt.gz`) |
| `bzip2` / `bunzip2` | Komprese .bz2 (pomalejší, lepší poměr) | `bzip2 file.tar` | (žádný výstup, vzniká `file.tar.bz2`) |
| `xz` / `unxz` | Komprese .xz (nejpomalejší, nejlepší poměr) | `xz file.tar` | (žádný výstup, vzniká `file.tar.xz`) |
| `zstd` | Moderní komprese (rychlá + dobrý poměr) | `zstd -3 file.tar` | (žádný výstup, vzniká `file.tar.zst`) |

### Porovnání kompresních nástrojů

| Nástroj | Rychlost | Kompresní poměr | Typické použití |
|---------|----------|-----------------|-----------------|
| gzip | Rychlý | Nižší | Denní zálohy, logy |
| bzip2 | Střední | Střední | Balíčky, distribuce |
| xz | Pomalý | Maximální | Archívy, balíčky (Linux kernel) |
| zstd | Velmi rychlý | Výborný | Moderní aplikace, big data |

### Příklady použití

```bash
# Úrovně komprese gzip (1-9, výchozí 6)
gzip -1 file.txt   # nejrychlejší, nejnižší komprese
gzip -9 file.txt   # nejpomalejší, nejvyšší komprese

# Úrovně komprese zstd (1-19, výchozí 3)
zstd -1 file.tar   # nejrychlejší
zstd -19 file.tar  # nejlepší komprese

# Dekomprese
gunzip file.txt.gz
bunzip2 file.tar.bz2
unxz file.tar.xz
zstd -d file.tar.zst
```

---

## Synchronizace (rsync)

> **⚠️ VAROVÁNÍ:** `dd` pracuje přímo s blokovými zařízeními. Přesměrování na špatný `of=` může nenávratně zničit data na disku. **VŽDY** dvakrát zkontrolujte cílové zařízení (`lsblk`, `blkid`).

`rsync` je výkonný nástroj pro synchronizaci souborů a adresářů. Kopíruje pouze změněné části souborů, což šetří čas i přenosové pásmo.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `rsync -av` | Lokální synchronizace | `rsync -av /source/ /destination/` | `sent 12345 bytes received 456 bytes 12801 bytes/sec` |
| `rsync -avz` | Přes SSH s kompresí | `rsync -avz /source/ user@host:/dest/` | `sent 54321 bytes received 123 bytes ...` |
| `rsync -av --delete` | Smazání chybějících souborů v cíli | `rsync -av --delete /src/ /dst/` | `deleting file.txt` |
| `rsync -av --dry-run` | Suchý běh (pouze náhled) | `rsync -av --dry-run /src/ /dst/` | `would send 12345 bytes` |
| `rsync -av --exclude` | Synchronizace s vyloučením | `rsync -av --exclude='*.tmp' /src/ /dst/` | (výstup bez .tmp souborů) |
| `rsync -avP` | S progress barem | `rsync -avP /source/ /destination/` | `file.txt 1024 100% 10.24MB/s 0:00:00` |

### Příklady použití

```bash
# Synchronizace celého adresáře včetně podadresářů
rsync -av /home/alice/dokumenty/ /backup/dokumenty/

# Synchronizace přes SSH s kompresí
rsync -avz /var/www/ user@server:/var/www/

# Suchý běh pro ověření
rsync -av --dry-run --delete /data/ /backup/data/

# Synchronizace s vyloučením dočasných souborů
rsync -av --exclude='*.log' --exclude='cache/' /var/log/ /backup/logs/

# Záloha s progress barem pro velké soubory
rsync -avP /home/ /mnt/backup/
```

---

## Bitová kopie (dd)

> **⚠️ VAROVÁNÍ:** `dd` pracuje přímo s blokovými zařízeními.
> Přesměrování na špatný `of=` může nenávratně zničit data na disku.
> **VŽDY** dvakrát zkontrolujte cílové zařízení (`lsblk`, `blkid`).

`dd` slouží k vytváření bitových kopií blokových zařízení. Kopíruje data na nejnižší úrovni, vcetně boot sektoru a oddílové tabulky.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `dd if= of=` | Záloha disku do souboru | `dd if=/dev/sda of=/backup/sda.img bs=4M status=progress` | `12345+0 records in`<br>`12345+0 records out` |
| `dd if= of=` | Obnova disku ze souboru | `dd if=/backup/sda.img of=/dev/sda bs=4M status=progress` | `12345+0 records in`<br>`12345+0 records out` |
| `dd conv=noerror,sync` | Ignorování chyb čtení (poškozený disk) | `dd if=/dev/sda of=/backup/sda.img bs=4M conv=noerror,sync status=progress` | `read error: Input/output error` |
| `dd if=/dev/zero` | Vymazání disku | `dd if=/dev/zero of=/dev/sdb bs=4M status=progress` | `12345+0 records in`<br>`12345+0 records out` |

### Příklady použití

```bash
# Záloha celého disku s progress barem
dd if=/dev/sda of=/backup/sda.img bs=4M status=progress

# Záloha s ignorováním chyb (pro vadný disk)
dd if=/dev/sda of=/backup/sda-failed.img bs=4M conv=noerror,sync status=progress

# Klonování disku na disk (přímá kopie)
dd if=/dev/sda of=/dev/sdb bs=4M status=progress

# Bezpečné vymazání disku (přepíše vše nulami)
dd if=/dev/zero of=/dev/sdb bs=4M status=progress

# Ověření zařízení před dd (VŽDY!)
lsblk
blkid
```

---

## Zálohovací strategie

Správná strategie zajistí, že data jsou v bezpeci a v případě havárie je lze obnovit.

### 3-2-1 pravidlo

| Prvek | Popis |
|-------|-------|
| 3 kopie dat | Jedna produkční + dvě záložní |
| 2 různá média | Např. externí disk + cloud (nebo jiné fyzické medium) |
| 1 mimo lokalitu | Záloha uložená na jiném místě |

### Typy zálo

| Typ | Popis | Narocnost | Rychlost obnovy |
|-----|-------|-----------|-----------------|
| Full backup | Úplná záloha všech dat (tar, dd) | Vysoké nároky na místo | Nejrychlejší |
| Incremental backup | Pouze změny od poslední zálohy | Malé nároky na místo | Pomalá (všechny inkrementy) |
| Differential backup | Všechny změny od poslední full zálohy | Střední nároky na místo | Střední (full + poslední diff) |

### Automatizace

```bash
# Denní rsync záloha v 2:00
0 2 * * * /usr/bin/rsync -a --delete /home/ /backup/home/

# Týdenní tar záloha v neděli v 3:00
0 3 * * 0 /usr/bin/tar -czf /backup/weekly-$(date +\%Y\%m\%d).tar.gz /home/

# Měsíční dd záloha boot sektoru (prvního dne v měsíci v 4:00)
0 4 1 * * /usr/bin/dd if=/dev/sda of=/backup/mbr-$(date +\%Y\%m\%d).img bs=512 count=1
```

---

## Shrnutí

- **tar** vytváří archivy, volitelne s kompresí (`-z`, `-j`, `-J`). Vzdy zkontrolujte obsah přes `tar -tf`.
- **gzip/bzip2/xz/zstd** se liší v rychlosti a kompresním poměru. `zstd` je moderní volba pro většinu případů.
- **rsync** synchronizuje pouze změny. Použijte `--dry-run` pro ověření před spuštěním.
- **dd** vytváří bitové kopie na úrovni blokových zařízení. **Vzdy** před použitím zkontrolujte zařízení.
- **3-2-1 pravidlo** je základ spolehlivé zálohovací strategie. Automatizace pomáha zajistit pravidelné zálohy.
- Pro každodenní zálohy preferujte `rsync`. Pro kompletní obraz systemu použijte `dd`.

---

➡️ [Zpět na přehled](README.md)
