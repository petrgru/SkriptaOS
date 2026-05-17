# Proxmox RAID1 Mirror – Kompletní průvodce
> Proxmox VE RAID1 Setup Guide

> **Platforma:** Proxmox VE (Debian-based)

## Obsah
1. Úvod
2. Shrnutí řešení
3. Architektura
4. Krok za krokem
    - Krok 1: záloha a kontrola
    - Krok 2: vyčištění druhého disku
    - Krok 3: kopie partition tabulky
    - Krok 4: změna typu partition
    - Krok 5: instalace mdadm
    - Krok 6: vytvoření RAID polí
    - Krok 7: LVM na RAID
    - Krok 8: kopie dat
    - Krok 9: konfigurace boot
    - Krok 10: GRUB a initramfs
    - Krok 11: reboot a přidání druhého disku
5. Údržba
6. Řešení problémů

---

## Úvod

Tento dokument popisuje vytvoření **RAID1 (mirror)** na serveru Proxmox VE s využitím dvou SSD disků.

### Pro koho je toto řešení určeno

Toto řešení cílí na hardware s omezenými zdroji: 4-8 GB RAM a slabší procesor. Na rozdíl od ZFS, které vyžaduje minimálně 1 GB RAM na každý 1 TB úložiště a výkonnější CPU pro kompresi a kontrolní součty, má mdadm RAID1 minimální režii. RAID1 pouze zrcadlí data na oba disky bez režie výpočtu parity (RAID5/6) nebo copy-on-write (ZFS/Btrfs). Výsledkem je stejná ochrana proti selhání disku jako u ZFS mirroru, ale s minimálními nároky na systém. Požadovaná znalost OS je střední: základní zkušenosti s Linuxem (příkazy, LVM, mount, grub) bez nutnosti pokročilé správy úložišť.

### Co jsme vytvořili:

| Komponenta | Typ | Popis |
|------------|-----|-------|
| **md0** | RAID1 | LVM volume group (pve_raid) |
| **md1** | RAID1 | EFI boot partition |
| **pve_raid** | LVM VG | Logical volumes na RAID |

### Výhody RAID1 Mirror:

- **Odolnost proti selhání**: Při výpadku jednoho disku systém jede dál
- **Automatická oprava**: Po opravě disku se automaticky synchronizuje
- **Bezvýpadkový provoz**: Systém běží během synchronizace

---

## Shrnutí řešení

### Hardware:
- 2x SSD disk (ADATA SU630, 240GB)
- Disk sda = původní Proxmox instalace
- Disk sdb = nový disk pro mirror

### Výsledná struktura partition:

```
sda (240GB SSD - původní Proxmox)
├── sda1: BIOS boot (1MB)
├── sda2: EFI boot (1GB) → md1 (RAID1)
└── sda3: LVM (222GB) → md0 (RAID1) → pve_raid

sdb (240GB SSD - nový mirror disk)
├── sdb1: BIOS boot (1MB)
├── sdb2: EFI boot (1GB) → md1 (RAID1)
└── sdb3: LVM (222GB) → md0 (RAID1) → pve_raid
```

---

## Architektura

```
┌─────────────────────────────────────────────────────────────┐
│                        Proxmox VE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐         ┌─────────────┐                 │
│   │   pve_raid  │         │     EFI     │                 │
│   │  (LVM VG)   │         │   (FAT32)   │                 │
│   ├─────────────┤         └─────────────┘                 │
│   │ • root      │                                          │
│   │ • swap      │         ┌─────────────────────────────┐  │
│   │ • data      │         │           md1               │  │
│   └─────────────┘         │     (EFI RAID1)              │  │
│           │               │   ┌───────┬───────┐           │  │
│           │               │   │ sda2  │ sdb2  │           │  │
│           │               └───┴───────┴───────┴───────────┘  │
│           │                                                   │
│   ┌─────────────────────────────┐                           │
│   │           md0               │                           │
│   │     (LVM RAID1)             │                           │
│   │   ┌───────┬───────┐        │                           │
│   │   │ sda3  │ sdb3  │        │                           │
│   └───┴───────┴───────┴────────┘                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Krok za krokem

### Krok 1: záloha a kontrola

Než začnete, zálohujte důležitá data a zaznamenejte aktuální konfiguraci.

```bash
# Záloha aktuální konfigurace
cp /etc/fstab /tmp/fstab.backup
cp /etc/mdadm/mdadm.conf /tmp/mdadm.conf.backup
lvs -a -o +devices > /tmp/lvs_backup.txt

# Kontrola disků
lsblk -f
fdisk -l /dev/sda /dev/sdb
```

**Očekávaný výstup:**
```
NAME               FSTYPE            FSVER    LABEL UUID
sda               
├─sda1             BIOS boot
├─sda2             vfat              FAT32          A19B-6266
└─sda3             LVM2_member       LVM2 001       NzMF0F-Gk1t-KZ6l...
  ├─pve-swap       swap
  ├─pve-root       ext4
  └─pve-data       LVM thin pool

sdb               (prázdný disk)
```

---

### Krok 2: vyčištění druhého disku

Odstraňte staré RAID metadaty z druhého disku.

```bash
# Zastavit případné existující RAID
mdadm --stop /dev/md0 /dev/md1 /dev/md2 2>/dev/null || true

# Vyčistit superbloky
mdadm --zero-superblock /dev/sdb1 /dev/sdb2 /dev/sdb3 2>/dev/null || true

# Ověření
cat /proc/mdstat
```

---

### Krok 3: kopie partition tabulky

Zkopírujte GPT tabulku z prvního disku na druhý.

```bash
# Záloha GPT tabulky
sgdisk --backup=/tmp/sda_gpt_backup /dev/sda

# Kopie na druhý disk
sgdisk --replicate=/dev/sdb /dev/sda

# Nové UUID (aby se disky nekřížily)
sgdisk -G /dev/sdb

# Ověření
fdisk -l /dev/sdb
```

> **⚠️ VAROVÁNÍ:** Parametr `-G` vygeneruje nové náhodné UUID pro disk sdb, což zabrání konfliktům.

---

### Krok 4: změna typu partition

Změňte typ partition na Linux RAID autodetect (kód fd00).

```bash
# Změna typu partition na Linux RAID
sgdisk --typecode=2:fd00 /dev/sdb    # EFI partition
sgdisk --typecode=3:fd00 /dev/sdb    # LVM partition

# Ověření
fdisk -l /dev/sdb | grep -E "sdb2|sdb3"
```

**Očekávaný výstup:**
```
/dev/sdb2     2048   2099199   2097152     1G Linux RAID
/dev/sdb3  2099200 468862094 466762895 222.6G Linux RAID
```

---

### Krok 5: instalace mdadm

Nainstalujte nástroj pro správu Linux RAID.

```bash
# Aktualizace repozitářů
apt-get update

# Instalace mdadm
apt-get install -y mdadm --allow-unauthenticated
```

> **ℹ️ POZNÁMKA:** Parametr `--allow-unauthenticated` je nutný kvůli problémům s Proxmox repozitáři.

---

### Krok 6: vytvoření RAID polí

Vytvořte degradovaná RAID1 pole (pouze s druhým diskem, první disk se přidá později).

```bash
# EFI partition RAID (sdb2)
yes | mdadm --create /dev/md1 --level=1 --raid-devices=2 --bitmap=internal missing /dev/sdb2 --force

# LVM partition RAID (sdb3)
yes | mdadm --create /dev/md0 --level=1 --raid-devices=2 --bitmap=internal missing /dev/sdb3 --force

# Kontrola
cat /proc/mdstat
```

**Parametry:**
- `--level=1`: RAID1 (mirror)
- `--bitmap=internal`: Interní bitmap pro rychlou re-sync po výpadku
- `missing`: Vytvoření degradovaného pole (bez prvního disku)

**Očekávaný výstup:**
```
md0 : active raid1 sdb3[1]
      233249344 blocks super 1.2 [2/1] [_U]
      bitmap: 2/2 pages [8KB]

md1 : active raid1 sdb2[1]
      1046528 blocks super 1.2 [2/1] [_U]
```

---

### Krok 7: LVM na RAID

Vytvořte LVM strukturu na novém RAID poli.

```bash
# Formátování EFI partition
mkfs.vfat -F32 /dev/md1

# Vytvoření PV na RAID
pvcreate -y /dev/md0

# Vytvoření VG
vgcreate pve_raid /dev/md0

# Vytvoření LV (velikosti podle potřeby)
lvcreate -L 7g -n swap pve_raid
lvcreate -L 65g -n root pve_raid
lvcreate -L 120g -n data pve_raid

# Vytvoření thin pool pro data (volitelné)
lvcreate -L 1g -n data_tmeta pve_raid
lvconvert --type thin-pool pve_raid/data --poolmetadata pve_raid/data_tmeta

# Formátování
mkswap /dev/pve_raid/swap
mkfs.ext4 -F /dev/pve_raid/root

# Kontrola
lvs -a
```

---

### Krok 8: kopie dat

Zkopírujte celý systém na nové RAID pole pomocí rsync.

```bash
# Mount nového root
mkdir -p /mnt/newroot
mount /dev/pve_raid/root /mnt/newroot

# Rsync celého systému (bez některých adresářů)
rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /mnt/newroot/

# Mount EFI
mkdir -p /mnt/newroot/boot/efi
mount /dev/md1 /mnt/newroot/boot/efi

# Kopie EFI dat
cp -r /boot/efi/* /mnt/newroot/boot/efi/

# Kontrola
ls /mnt/newroot
```

---

### Krok 9: konfigurace boot

Nastavte nový fstab s UUID nových zařízení.

```bash
# Získání nových UUID
NEW_ROOT_UUID=$(blkid -s UUID -o value /dev/pve_raid/root)
NEW_SWAP_UUID=$(blkid -s UUID -o value /dev/pve_raid/swap)
NEW_EFI_UUID=$(blkid -s UUID -o value /dev/md1)

# Nový fstab
cat > /mnt/newroot/etc/fstab << EOF
UUID=$NEW_ROOT_UUID /                ext4     errors=remount-ro  0  1
UUID=$NEW_SWAP_UUID none             swap     sw                       0  0
UUID=$NEW_EFI_UUID /boot/efi         vfat     umask=0077               0  1
EOF
```

---

### Krok 10: GRUB a initramfs

Připravte boot z RAID pole.

```bash
# Připravit chroot prostředí
mount --bind /proc /mnt/newroot/proc
mount --bind /sys /mnt/newroot/sys
mount --bind /dev /mnt/newroot/dev

# Konfigurace GRUB
cat > /mnt/newroot/etc/default/grub << "EOF"
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=Proxmox
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX="root=/dev/mapper/pve_raid-root"
GRUB_ENABLE_CRYPTODISK=0
GRUB_PRELOAD_MODULES="lvm mdraid1x"
EOF

# Konfigurace mdadm
mdadm --detail --scan > /etc/mdadm/mdadm.conf
cp /etc/mdadm/mdadm.conf /mnt/newroot/etc/mdadm/mdadm.conf

# Aktualizace initramfs
chroot /mnt/newroot update-initramfs -u

# Instalace GRUB na oba disky
chroot /mnt/newroot grub-install /dev/sda
chroot /mnt/newroot grub-install /dev/sdb
chroot /mnt/newroot update-grub

# Odpojení
umount /mnt/newroot/proc /mnt/newroot/sys /mnt/newroot/dev
```

---

### Krok 11: reboot a přidání druhého disku

Po restartu se systém spustí z nového RAID pole a přidáte první disk.

```bash
# Reboot
reboot
```

Po nabootování:

```bash
# Přidání disku do RAID polí
mdadm /dev/md0 --add /dev/sda3
mdadm /dev/md1 --add /dev/sda2

# Sledování synchronizace
watch cat /proc/mdstat
```

> **ℹ️ POZNÁMKA:** Synchronizace 222GB disku trvá přibližně 30-60 minut při běžném zatížení.

**Očekávaný výstup během sync:**
```
md0 : active raid1 sda3[2] sdb3[1]
      233249344 blocks super 1.2 [2/1] [_U]
      [============>.......]  recovery = 50.0% (116624672/233249344) finish=25.0min
```

**Po dokončení synchronizace:**
```
md0 : active raid1 sda3[2] sdb3[1]
      233249344 blocks super 1.2 [2/2] [UU]
```

---

## Údržba

### Kontrola stavu RAID

```bash
# Zobrazení stavu RAID
cat /proc/mdstat

# Detailní informace
mdadm --detail /dev/md0
mdadm --detail /dev/md1

# Informace o členech RAID
mdadm --examine /dev/sda3
mdadm --examine /dev/sdb3
```

### Pravidelná údržba

```bash
# SMART test disků (doporučeno jednou měsíčně)
smartctl -a /dev/sda
smartctl -a /dev/sdb

# Kontrola konzistence RAID (doporučeno jednou týdně)
echo check > /sys/block/md0/md/sync_action

# Zobrazení event logu
mdadm --detail /dev/md0 | grep Events
```

### Automatické kontroly

Vytvořte cron úlohu pro pravidelné kontroly:

```bash
# Přidat do /etc/cron.d/raid-check
0 3 * * 0 root /usr/share/mdadm/checkarray --all --cron
```

---

## Řešení problémů

### Problém: Systém nebootuje

**Příznaky:**
- GRUB error
- "No such device" při bootu

**Řešení:**

1. Boot z Proxmox Live USB/DVD
2. Sestavit RAID pole manuálně:
   ```bash
   mdadm --assemble --scan
   ```
3. Opravit GRUB:
   ```bash
   mount /dev/mapper/pve_raid-root /mnt
   mount /dev/md1 /mnt/boot/efi
   mount --bind /proc /mnt/proc
   mount --bind /sys /mnt/sys
   mount --bind /dev /mnt/dev
   chroot /mnt
   grub-install /dev/sda
   grub-install /dev/sdb
   update-grub
   update-initramfs -u
   ```

### Problém: RAID degradovaný po restartu

**Příčina:** mdadm.conf neobsahuje správné UUID polí

**Řešení:**
```bash
# Získat správné UUID
mdadm --detail --scan > /etc/mdadm/mdadm.conf

# Restartovat RAID
mdadm --stop /dev/md0
mdadm --assemble --scan
```

### Problém: LVM nemůže najít VG

**Řešení:**
```bash
# PV rescan (fyzických svazků)
pvscan --cache

# Obnovit VG
vgchange -ay
```

### Problém: Pomalá synchronizace

**Řešení:**
```bash
# Nastavit rychlost sync (v KiB/s, 0 = neomezená)
echo 0 > /proc/sys/dev/raid/speed_limit_max

# Nebo nastavit konkrétní hodnotu (např. 200000 KiB/s)
echo 200000 > /proc/sys/dev/raid/speed_limit_max
```

### Problém: Disk selhal

**Příznaky:**
- `md0 : active raid1 sda3[1] sdb3[F]` (F = Faulty)
- Varovné emaily od mdadm

**Řešení:**
```bash
# 1. Zkontrolovat SMART
smartctl -a /dev/sdX

# 2. Označit jako selhavší (pokud ještě není)
mdadm /dev/md0 --fail /dev/sdX

# 3. Fyzicky vyměnit disk

# 4. Zkopírovat partition tabulku
sgdisk --replicate=/dev/sdX /dev/sdY
sgdisk -G /dev/sdY

# 5. Přidat nový disk do RAID
mdadm /dev/md0 --add /dev/sdY

# 6. Sledovat synchronizaci
cat /proc/mdstat
```

---

## Rychlá referenční karta

| Příkaz | Popis |
|--------|-------|
| `cat /proc/mdstat` | Stav RAID polí |
| `mdadm --detail /dev/md0` | Detail RAID pole md0 |
| `lsblk -f` | Struktura disků a LVM |
| `vgs` | Volume groups |
| `lvs -a` | Logical volumes |
| `update-initramfs -u` | Aktualizace initramfs |
| `update-grub` | Aktualizace GRUB |
| `smartctl -a /dev/sda` | SMART data disku |

---

## Důležitá upozornění

1. **záloha**: Před jakoukoliv manipulací s RAID zálohujte důležitá data
2. **Testování**: Nikdy netestujte příkazy pro selhání disku na produkčním serveru
3. **Konzistence**: Zajistěte, aby oba disky měly stejnou nebo větší kapacitu
4. **Monitoring**: Nastavte monitoring RAID stavu (např. mdadm --monitor)

---

## Užitečné zdroje

- [Linux RAID Wiki](https://raid.wiki.kernel.org/)
- [mdadm Manual](https://man7.org/linux/man-pages/man8/mdadm.8.html)
- [Proxmox Wiki](https://pve.proxmox.com/wiki/Storage)

---

➡️ [Zpět na přehled](README.md)

*Dokument vytvořen: Březen 2026*  
*Testováno na: Proxmox VE s Debian Trixie*