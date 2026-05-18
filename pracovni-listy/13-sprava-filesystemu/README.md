# Pracovní list: Správa souborového systému

> Procvičení práce s disky, oddíly, souborovými systémy, mountem, fstab a LVM.

---

## Zadání

### Úkol 1: Analýza disků a oddílů

1. Zobraz všechna bloková zařízení pomocí `lsblk` (stromově)
2. Zjisti UUID a typ souborového systému všech oddílů (`blkid`)
3. Zobraz využití disků v čitelném formátu (`df -h`)
4. Zjisti využití inodů (`df -hi`)
5. Zobraz podrobné info o prvním disku (`sudo fdisk -l /dev/sda | head -20`)
6. K čemu slouží UUID u disku a proč je lepší než `/dev/sda1`?

### Úkol 2: Práce s mountem

1. Vytvoř mount point `sudo mkdir /mnt/test`
2. Vytvoř souborový systém v loop zařízení (simulace):
   ```bash
   dd if=/dev/zero of=/tmp/test.img bs=1M count=100
   mkfs.ext4 /tmp/test.img
   ```
3. Připoj obraz: `sudo mount -o loop /tmp/test.img /mnt/test`
4. Ověř připojení: `mount | grep /mnt/test`
5. Vytvoř v něm soubor a odpoj ho (`sudo umount /mnt/test`)
6. Zkus připojit s read-only (`-o ro`) — co se stane při pokusu o zápis?

### Úkol 3: fstab

1. Zobraz aktuální `/etc/fstab`
2. Vysvětli jednotlivé sloupce (filesystem, mount point, type, options, dump, pass)
3. Co znamená možnost `noatime` a proč se používá?
4. Přidej do `/etc/fstab` záznam pro `/tmp/test.img` (loop zařízení)
   - Filesystem: `/tmp/test.img`
   - Mount point: `/mnt/test`
   - Type: `ext4`
   - Options: `defaults,noatime,loop`
5. Ověř syntaxi: `sudo mount -a`
6. Jaký je rozdíl mezi `dump 1` a `dump 0`? A `pass 1` vs `pass 2`?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-3. Základní přehled
lsblk
blkid
df -h

# 4. Inody
df -hi

# 5. Detail disku
sudo fdisk -l /dev/sda | head -20

# 6. UUID je unikátní identifikátor oddílu, nemění se
# /dev/sda1 se může změnit (přidání/odebrání disků)
# UUID je stabilní napříč bootováním
```

### Úkol 2 — Řešení

```bash
# 1-2. Vytvoření obrazu
sudo mkdir /mnt/test
dd if=/dev/zero of=/tmp/test.img bs=1M count=100
mkfs.ext4 /tmp/test.img

# 3. Připojení
sudo mount -o loop /tmp/test.img /mnt/test

# 4. Ověření
mount | grep /mnt/test
# /tmp/test.img on /mnt/test type ext4 (rw,relatime)

# 5-6. Read-only test
sudo umount /mnt/test
sudo mount -o loop,ro /tmp/test.img /mnt/test
touch /mnt/test/test.txt
# touch: cannot touch '/mnt/test/test.txt': Read-only file system
```

### Úkol 3 — Řešení

```bash
# 1. Zobrazení fstab
cat /etc/fstab

# 2. Formát: <device> <mountpoint> <type> <options> <dump> <pass>
# /dev/sda1  /  ext4  defaults  0  1

# 3. noatime = neaktualizovat čas přístupu k souboru
# Šetří zápisy na disk, zvyšuje výkon (SSD i HDD)

# 4. Přidání záznamu
echo '/tmp/test.img /mnt/test ext4 defaults,noatime,loop 0 0' | sudo tee -a /etc/fstab

# 5. Ověření
sudo mount -a

# 6. dump: 0 = nezálohovat, 1 = zálohovat
# pass: 1 = root FS (kontrola první), 2 = ostatní FS, 0 = nekontrolovat
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš zobrazit disky, oddíly, UUID a využití
- [ ] Dokážeš vytvořit obraz disku, naformátovat ho a připojit
- [ ] Rozumíš struktuře `/etc/fstab` a významu options
- [ ] Víš, proč je UUID lepší než `/dev/sdaX`

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../13-sprava-filesystemu.md) · [Zpět na přehled](../../README.md)
