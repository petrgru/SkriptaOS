# Aktualizace systému

> System Updates and Maintenance

---

## Úvod

Aktualizace systému je nezbytnou součástí správy každého operačního systému. Tato kapitola pokrývá správu balíčků v různých distribucích Linuxu, aktualizaci jádra, plánování aktualizací a postupy pro rollback v případě problémů.

## 1. Správa balíčků

Spravovat software v Linuxu znamená umět pracovat s balíčkovacími systémy. Různé distribuce pouzívají ruzné nástroje, ale princip je stejný: instalace, aktualizace a odstraňování softwaru.

### 1.1 apt (Debian / Ubuntu)

Nejpoužívanejší balíčkovací systém na Debian-based distribucích.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `apt update` | Aktualizuje seznam dostupných balíčků z repozitáře | `sudo apt update` | `Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease`<br>`Fetched 12.5 MB in 3s (4.2 MB/s)` |
| `apt upgrade` | Aktualizuje všechny nainstalované balíčky na nejnovější verzi | `sudo apt upgrade -y` | `0 upgraded, 0 newly installed, 0 to remove`<br>nebo seznam balíčků k upgrade |
| `apt install` | Nainstaluje nový balíček | `sudo apt install htop` | `Reading package lists... Done`<br>`The following NEW packages will be installed: htop` |
| `apt remove` | Odstraní balíček (ponechá konfiguraci) | `sudo apt remove htop` | `The following packages will be REMOVED: htop` |
| `apt purge` | Odstraní balíček i s konfiguračními soubory | `sudo apt purge htop` | `Purging configuration files for htop...` |
| `apt autoremove` | Odstraní nepotřebné závislosti (osiřelé balíčky) | `sudo apt autoremove` | `The following packages will be REMOVED: libabc0` |
| `apt search` | Prohledá databázi balíčků podle názvu/popisu | `apt search text editor` | `vim - Vi IMproved - enhanced text editor` |
| `apt show` | Zobrazí detailní informace o balíčku | `apt show bash` | `Package: bash`<br>`Version: 5.1-6ubuntu1`<br>`Depends: base-files, debianutils` |
| `apt list --upgradable` | Zobrazí balíčky, které lze aktualizovat | `apt list --upgradable` | `libssl3/jammy-updates 3.0.2-0ubuntu1.18 amd64` |

**Praktický příklad: Kompletní údržba systému**

```bash
# 1. Aktualizovat seznam balíčků
sudo apt update

# 2. Zjistit, co lze upgradovat
apt list --upgradable

# 3. Provést upgrade
sudo apt upgrade -y

# 4. Vyřešit závislosti (občas je potřeba)
sudo apt full-upgrade -y

# 5. Ukliď nepotřebné balíčky
sudo apt autoremove --purge -y

# 6. Vyčistit cache stažených balíčků
sudo apt clean

# Možný výstup:
# Reading package lists... Done
# Building dependency tree... Done
# 5 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

### 1.2 dnf / yum (Fedora / RHEL)

Balíčkovací systém pro Red Hat rodinu distribucí. `dnf` je moderní náhrada za `yum`.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `dnf check-update` | Zkontroluje dostupné aktualizace | `sudo dnf check-update` | `firefox.x86_64 120.0-1.fc39 updates` |
| `dnf update` | Aktualizuje všechny balíčky | `sudo dnf update -y` | `Upgraded: 8 packages` |
| `dnf install` | Nainstaluje balíček | `sudo dnf install htop` | `Installed: htop-3.2.2-1.fc39.x86_64` |
| `dnf groupinstall` | Nainstaluje skupinu balíčků | `sudo dnf groupinstall "Development Tools"` | `Installing Group: Development Tools` |
| `dnf history` | Zobrazí historii transakcí dnf | `dnf history` | `ID | Command | Date`<br>` 7 | install htop | 2025-01-15` |
| `dnf repolist` | Zobrazí povolené repozitáře | `dnf repolist` | `repo-id     repo-name`<br>`fedora      Fedora 39` |
| `dnf clean all` | Vyčistí cache | `sudo dnf clean all` | `Cleaning repos: fedora updates` |

**Rollback balíčku s dnf history:**

```bash
# Zobrazit historii
sudo dnf history

# Výstup:
# ID     | Command line             | Date and time
#--------------------------------------------------------------
#     8 | update                   | 2025-03-10 14:22
#     7 | install htop             | 2025-03-09 10:00
#     6 | remove nano              | 2025-03-08 16:30

# Vrátit transakci č. 8 (aktualizace)
sudo dnf history undo 8

# Výstup:
# Undoing transaction 8, running...
# Removed: firefox-121.0-1.fc39.x86_64
# Installed: firefox-120.0-1.fc39.x86_64
```

### 1.3 pacman (Arch Linux)

Balíčkovací systém pro Arch Linux a jeho deriváty.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `-Syu` | Synchronizace + aktualizace systému | `sudo pacman -Syu` | `:: Synchronizing package databases...`<br>`:: Starting full system upgrade...` |
| `-Ss` | Vyhledávání v repozitářích | `pacman -Ss htop` | `community/htop 3.3.0-1`<br>`    Interactive process viewer` |
| `-Qs` | Vyhledávání mezi nainstalovanými balíčky | `pacman -Qs bash` | `local/bash 5.2.026-1` |
| `-S` | Instalace balíčku | `sudo pacman -S htop` | `Packages (1): htop-3.3.0-1` |
| `-Rns` | Odstranění balíčku + závislostí + konfigurace | `sudo pacman -Rns htop` | `Removing htop...` |
| `-U` | Instalace z lokálního souboru | `sudo pacman -U package.pkg.tar.zst` | `Loading packages...` |

**Praktický příklad: Arch Linux update:**

```bash
# Kompletní upgrade systému
sudo pacman -Syu

# Výstup:
# :: Synchronizing package databases...
#  core is up to date
#  extra   168.6 KiB  1.23 MiB/s 00:00
# :: Starting full system upgrade...
# there is nothing to do

# Pokud došlo k problémům se závislostmi:
sudo pacman -Syu --ignore linux,kernel

# Ignoruje balíčky linux a kernel při upgradu
```

### 1.4 snap a flatpak (univerzální balíčky)

Sandboxované univerzální balíčky fungující napříč distribucí.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `snap install` | Instalace snap balíčku | `sudo snap install htop` | `htop 3.3.0 from Snapcrafters installed` |
| `snap refresh` | Aktualizace všech snap balíčků | `sudo snap refresh` | `All snaps up to date.` |
| `snap list` | Seznam nainstalovaných snapů | `snap list` | `Name  Version  Rev  Tracking  Publisher` |
| `snap revert` | Vrácení snapu na předchozí verzi | `sudo snap revert htop` | `htop reverted to 3.2.2` |
| `flatpak install` | Instalace flatpak balíčku | `flatpak install flathub org.gimp.GIMP` | `Installing: org.gimp.GIMP/x86_64/stable` |
| `flatpak update` | Aktualizace flatpak balíčků | `flatpak update` | `Looking for updates...` |
| `flatpak list` | Seznam nainstalovaných flatpaků | `flatpak list --app` | `GIMP  org.gimp.GIMP  stable  flathub` |

### 1.5 pip a npm (jazyk-specifické správce)

Správci balíčků pro Python a JavaScript ekosystémy.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `pip install` | Instalace Python balíčku | `pip install requests` | `Successfully installed requests-2.31.0` |
| `pip install --upgrade` | Upgrade Python balíčku | `pip install --upgrade pip` | `Successfully installed pip-24.0` |
| `pip list --outdated` | Zastaralé Python balíčky | `pip list --outdated` | `Package  Version  Latest  Type` |
| `pip uninstall` | Odstranění Python balíčku | `pip uninstall requests -y` | `Successfully uninstalled requests-2.31.0` |
| `npm update` | Aktualizace Node.js balíčků | `npm update` | (aktualizuje dle package.json) |
| `npm outdated` | Zastaralé npm balíčky | `npm outdated` | `Package  Current  Wanted  Latest` |
| `npm install -g` | Globální instalace npm balíčku | `sudo npm install -g yarn` | `added 1 package in 2s` |

**Příklad: Batch update pip balíčků:**

```bash
# Vypsat zastaralé balíčky
pip list --outdated --format=columns

# Výstup:
# Package          Version  Latest  Type
# ----------------- -------- ------ -----
# certifi          2023.5   2024.2  wheel
# setuptools       65.5.0   69.5.0  wheel

# Automatický upgrade všeho
pip freeze --local | grep -v '^\-e' | cut -d = -f 1 | xargs -n1 pip install -U

# Upgrade npm globálních balíčků
sudo npm update -g
```


> **Související**: Ověření integrity nainstalovaných balíčků pomocí `debsums` (Debian/Ubuntu)
> nebo `rpm --verify` (RHEL/Fedora) je popsáno v [kapitole 25](25-bezpecnostni-audit.md).

---

## 2. Aktualizace kernelu

Linuxové jádro je srdcem systému. Jeho aktualizace může prinést nové ovladače, opravy bezpečnosti a lepší výkon.

### 2.1 Informace o jádře

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `uname -r` | Zobrazí verzi aktuálně běžícího jádra | `uname -r` | `6.5.0-21-generic` |
| `uname -a` | Všechny informace o jádře | `uname -a` | `Linux hostname 6.5.0-21-generic #22-Ubuntu SMP` |
| `hostnamectl` | Informace o systému (systemd) | `hostnamectl` | `Kernel: Linux 6.5.0-21-generic` |
| `lsb_release -a` | Info o distribuci | `lsb_release -a` | `Distributor ID: Ubuntu`<br>`Release: 22.04` |

### 2.2 Typy jader Linuxu

| Typ | Popis | Použití |
|-----|-------|---------|
| **Mainline** | Nejnovější jádro od Linuse Torvaldse, vychází každé 2-3 měsíce | Testování, nové funkce |
| **Stable** | Mainline + backportované bezpečnostní opravy, stabilní verze | Běžné použití, servery |
| **LTS** | Long Term Support, podpora 5-6 let | Produkční servery, enterprise |
| **Hardware Enablement (HWE)** | Novější jádro pro starší LTS distribuci | Ubuntu LTS s novějším hardwarem |

### 2.3 Aktualizace jádra pomocí správce balíčků

**Debian/Ubuntu:**

```bash
# Zjistit aktuální verzi jádra
uname -r
# Výstup: 6.5.0-21-generic

# Zjistit jaká jádra jsou k dispozici
apt search linux-image

# Nainstalovat nové jádro (automaticky s updaty)
sudo apt update
sudo apt upgrade -y

# Konkrétní jádro
sudo apt install linux-image-6.8.0-35-generic linux-headers-6.8.0-35-generic

# Po restartu zkontrolovat novou verzi
uname -r
# Výstup: 6.8.0-35-generic
```

**Fedora/RHEL:**

```bash
# Zkontrolovat dostupné updaty jádra
sudo dnf check-update kernel

# Nainstalovat nové jádro
sudo dnf update kernel -y

# Výstup:
# kernel.x86_64 6.7.5-200.fc39 updates

# Po restartu
uname -r
# Výstup: 6.7.5-200.fc39.x86_64
```

### 2.4 GRUB - Správce zavádění systému

Po instalaci nového jádra je potřeba aktualizovat GRUB konfiguraci.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `update-grub` | Automatická regenerace GRUB konfigurace (Debian/Ubuntu) | `sudo update-grub` | `Generating grub configuration file ...`<br>`Found linux image: /boot/vmlinuz-6.8.0-35` |
| `grub-mkconfig` | Generování GRUB konfigurace (všeobecné) | `sudo grub-mkconfig -o /boot/grub/grub.cfg` | `Found linux image: /boot/vmlinuz-...` |
| `grub2-mkconfig` | GRUB2 verze (Fedora/RHEL) | `sudo grub2-mkconfig -o /boot/grub2/grub.cfg` | `Found linux image: /boot/vmlinuz-...` |

**Staré jádro jako výchozí (fallback):**

```bash
# Zobrazit všechny nainstalované verze jader
ls /boot/vmlinuz-*
# Výstup:
# /boot/vmlinuz-6.5.0-21-generic
# /boot/vmlinuz-6.8.0-35-generic

# Trvale nastavit staré jádro jako výchozí
# Upravit /etc/default/grub:
# GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.5.0-21-generic"

sudo nano /etc/default/grub
sudo update-grub

# Nebo dočasně při bootu vybrat v GRUB menu
```

### 2.5 Initramfs - Initial RAM Filesystem

Initramfs je dočasný souborový systém používaný při bootu před připojením hlavního kořenového FS.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `update-initramfs` | Regenerace initramfs (Debian/Ubuntu) | `sudo update-initramfs -u` | `update-initramfs: Generating /boot/initrd.img-6.8.0-35` |
| `mkinitcpio` | Generování initramfs (Arch) | `sudo mkinitcpio -p linux` | `==> Building image from preset: /etc/mkinitcpio.d/linux.preset` |
| `dracut` | Generování initramfs (Fedora/RHEL) | `sudo dracut --force` | `Executing: /usr/bin/dracut --force` |

**Příklad: Ruční přegenerování initramfs po změně modulů:**

```bash
# Ubuntu/Debian: regenerovat pro aktuální jádro
sudo update-initramfs -u -k $(uname -r)

# Výstup:
# update-initramfs: Generating /boot/initrd.img-6.8.0-35-generic

# Pro všechny verze jader
sudo update-initramfs -u -k all

# Fedora/RHEL: regenerovat aktuální jádro
sudo dracut --force --kver $(uname -r)

# Arch: regenerovat podle preset
sudo mkinitcpio -p linux
```

### 2.6 Kernel moduly (ovladače)

Moduly jsou části kódu, které lze nahrát do jádra za běhu (např. ovladače zařízení).

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `lsmod` | Seznam nahraných modulů | `lsmod` | `Module       Size  Used by`<br>`nvidia      52488  12` |
| `modprobe` | Nahrání/odstranění modulu včetně závislostí | `sudo modprobe nvidia` | (zádný výstup, modul nahran) |
| `modinfo` | Info o modulu | `modinfo nvidia` | `filename: /lib/modules/.../nvidia.ko` |
| `insmod` | Nahrání modulu (bez závislostí) | `sudo insmod /path/to/module.ko` | (zádný výstup) |
| `rmmod` | Odstranění modulu | `sudo rmmod nvidia` | (zádný výstup) |
| `depmod` | Generování závislostí modulů | `sudo depmod -a` | (zádný výstup, generuje modules.dep) |
| `/etc/modules` | Moduly načítané při bootu | `cat /etc/modules` | `nvidia`<br>`i915` |

**Blokování problematického modulu:**

```bash
# Zjistit jaký modul způsobuje problém
lsmod | grep nouveau
# nouveau    123456  0

# Zakázat modul (přidat do blacklistu)
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf

# Regenerovat initramfs
sudo update-initramfs -u

# Po restartu už modul nebude nahran
```

### 2.7 Vlastní jádro z kompilace

Pro pokročilé uživatele, kteří chtějí jádro na míru.

```bash
# 1. Stáhnout zdrojový kód jádra
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.8.tar.xz
tar -xf linux-6.8.tar.xz
cd linux-6.8

# 2. Konfigurace (vzít stávající konfiguraci)
cp /boot/config-$(uname -r) .config
make olddefconfig

# 3. Kompilace (použít více jader)
make -j$(nproc)

# Výstup:
#   LD      vmlinux.o
#   OBJCOPY arch/x86/boot/vmlinux.bin
#   Building modules, stage 2.

# 4. Instalace modulů
sudo make modules_install

# 5. Instalace jádra
sudo make install

# 6. Aktualizace GRUB
sudo update-grub

# 7. Restart a volba nového jádra v GRUB
sudo reboot
```

### 2.8 DKMS - Dynamic Kernel Module Support

DKMS umožňuje automaticky překompilovat moduly pro nová jádra.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `dkms status` | Stav DKMS modulů | `dkms status` | `nvidia/545.29.06, 6.8.0-35, x86_64: installed` |
| `dkms install` | Instalace modulu do DKMS | `sudo dkms install -m nvidia -v 545.29.06` | `Module nvidia/545.29.06 installed` |
| `dkms remove` | Odstranění DKMS modulu | `sudo dkms remove -m nvidia -v 545.29.06 --all` | `Deleting module nvidia/545.29.06` |

```bash
# Příklad: Po upgrade jádra se DKMS modul automaticky překompluje
sudo apt upgrade
# DKMS: add item nvidia/545.29.06 into kernel 6.8.0-36-generic
# Building for 6.8.0-36-generic
# Module build for kernel 6.8.0-36-generic was successful

# Rucní prekompilace pro nové jádro
sudo dkms autoinstall
```

---

## 3. Správa souborových systémů

Práce s disky, oddíly a souborovými systémy je základní dovednost pro správu systému.

### 3.1 mount a umount

Připojování a odpojování souborových systémů.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `mount` | Zobrazí aktuálně připojené FS | `mount` | `/dev/sda1 on / type ext4 (rw,relatime)` |
| `mount /dev/sdb1 /mnt` | Připojí oddíl do adresáře | `sudo mount /dev/sdb1 /mnt/data` | (zádný výstup) |
| `mount -t ext4` | Připojení s určením typu FS | `sudo mount -t ext4 /dev/sdb1 /mnt` | (zádný výstup) |
| `mount -o loop` | Připojení ISO image | `sudo mount -o loop image.iso /mnt` | (zádný výstup) |
| `umount` | Odpojení FS | `sudo umount /mnt/data` | (zádný výstup) |
| `mount -a` | Připojí vše z /etc/fstab | `sudo mount -a` | (zádný výstup, mountne dle fstab) |
| `findmnt` | Stromová struktura mount pointu | `findmnt` | `TARGET  SOURCE  FSTYPE  OPTIONS` |

**Příklad: Práce s ISO obrazem a USB diskem:**

```bash
# Připojit ISO soubor
sudo mount -o loop /home/user/ubuntu-22.04.iso /mnt/iso
ls /mnt/iso
# Výstup: casper  dists  install  isolinux  pool  preseed  README

# Připojit USB disk (nejprve zjistit název)
lsblk
# Výstup:
# NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
# sda      8:0    0 238.5G  0 disk
# sdb      8:16   1  14.5G  0 disk /media/user/USB

sudo mount /dev/sdb1 /mnt/usb

# Odpojit
sudo umount /mnt/iso /mnt/usb
```

### 3.2 /etc/fstab

Konfigurační soubor pro automatické připojování FS při bootu.

```bash
# Obsah /etc/fstab
cat /etc/fstab

# Výstup:
# # /etc/fstab: static file system information
# UUID=abc123  /     ext4  defaults      0   1
# UUID=def456  /home ext4  defaults      0   2
# UUID=789abc  none  swap  sw            0   0
# /dev/sdb1     /mnt/data ext4  defaults  0   0
# //server/share /mnt/nfs cifs credentials=/etc/smbcred,iocharset=utf8 0 0

# Struktura řádku:
# <file system>  <mount point>  <type>  <options>  <dump>  <pass>
```

| Sloupec | Popis | Hodnoty |
|---------|-------|---------|
| `<file system>` | Zařízení nebo UUID | `/dev/sda1`, `UUID=abc123`, `LABEL=DATA` |
| `<mount point>` | Kam se FS připojí | `/`, `/home`, `/mnt/data` |
| `<type>` | Typ FS | `ext4`, `xfs`, `btrfs`, `ntfs`, `cifs` |
| `<options>` | Možnosti připojení | `defaults`, `noatime`, `ro`, `rw` |
| `<dump>` | Zálohování (0=vypnuto) | `0` nebo `1` |
| `<pass>` | Pořadí kontroly fsck (0=nekontrolovat) | `0`, `1` (root), `2` (ostatní) |

**Přidání nového disku do fstab:**

```bash
# 1. Zjistit UUID disku
sudo blkid /dev/sdb1
# Výstup: /dev/sdb1: UUID="abcd1234" TYPE="ext4" PARTUUID="xyz789"

# 2. Přidat řádek do /etc/fstab
echo "UUID=abcd1234  /mnt/data  ext4  defaults  0  2" | sudo tee -a /etc/fstab

# 3. Otestovat (bez restartu)
sudo mount -a
# Pokud je chyba, ihned opravit fstab, nebo systém nenabootuje!
```

### 3.3 fsck - File System Check

Kontrola a oprava integrity souborových systémů.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `fsck.ext4` | Kontrola ext4 FS | `sudo fsck.ext4 -f /dev/sda1` | `Pass 1: Checking inodes, blocks, and sizes` |
| `fsck.xfs` | XFS má vlastní nástroje (xfs_repair) | `sudo xfs_repair /dev/sda2` | `Phase 1 - find and verify superblock...` |
| `fsck.btrfs` | Btrfs kontrola (většinu dělá sám) | `sudo btrfs check /dev/sda3` | `Checking filesystem on /dev/sda3` |
| `fsck -N` | Suchý běh, nezapíše zmeny | `sudo fsck -N /dev/sda1` | `[/sbin/fsck.ext4 (1) -- /] fsck.ext4 /dev/sda1` |
| `fsck -y` | Automaticky ano na všechny dotazy | `sudo fsck -y /dev/sda1` | (automaticky opraví) |

**Důlezité: FS musí být odpojen před fsck:**

```bash
# Kontrola root oddílu (nelze za běhu)
# Je potřeba nabootovat z live USB nebo použít recovery mód

# Kontrola ne-root oddílu
sudo umount /dev/sdb1
sudo fsck.ext4 -f -v /dev/sdb1

# Výstup:
# e2fsck 1.46.5 (30-Dec-2021)
# Pass 1: Checking inodes, blocks, and sizes
# Pass 2: Checking directory structure
# Pass 3: Checking directory connectivity
# Pass 4: Checking reference counts
# Pass 5: Checking group summary information
# /dev/sdb1: 12/65536 files (0.0% non-contiguous), 8899/262144 blocks

# Vynucená kontrola při příštím bootu
sudo touch /forcefsck
# nebo
sudo shutdown -rF now
```

### 3.4 tune2fs a xfs_admin

Ladění parametrů ext4 a XFS filesystémů.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `tune2fs -l` | Zobrazí superblock informace ext4 | `sudo tune2fs -l /dev/sda1` | `Filesystem volume name: root`<br>`Block count: 62500000` |
| `tune2fs -c` | Nastaví maximální pocet mountů mezi fsck | `sudo tune2fs -c 30 /dev/sda1` | `Setting maximal mount count to 30` |
| `tune2fs -m` | Změní rezervovaný prostor pro root | `sudo tune2fs -m 2 /dev/sda1` | `Setting reserved blocks percentage to 2%` |
| `tune2fs -L` | Nastaví štítek (label) FS | `sudo tune2fs -L ROOT /dev/sda1` | `Setting filesystem label to ROOT` |
| `xfs_admin -l` | Zobrazí label XFS | `sudo xfs_admin -l /dev/sda2` | `label = "data"` |
| `xfs_admin -L` | Nastaví label XFS | `sudo xfs_admin -L DATA /dev/sda2` | `writing all SBs: new label = DATA` |

```bash
# Ext4: optimalizace pro SSD
sudo tune2fs -o discard /dev/sda1   # Povolí TRIM
sudo tune2fs -O ^has_journal /dev/sdb1  # Vypne journálování (riskantní!)

# Ext4: zmensení rezervovaného prostoru na 1% (pro datové oddíly)
sudo tune2fs -m 1 /dev/sdb1
```

### 3.5 Btrfs - subvolumes a snapshoty

Moderní copy-on-write souborový systém s pokročilými funkcemi.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `btrfs subvolume create` | Vytvoří nový subvolume | `sudo btrfs subvolume create /mnt/data/@snapshots` | `Create subvolume '/mnt/data/@snapshots'` |
| `btrfs subvolume list` | Seznam subvolumů | `sudo btrfs subvolume list /mnt/data` | `ID 256 gen 5 top level 5 path @snapshots` |
| `btrfs subvolume snapshot` | Vytvoří snapshot (read-write) | `sudo btrfs subvolume snapshot /mnt/data /mnt/data/@snapshots/backup-1` | `Create a snapshot of '/mnt/data'` |
| `btrfs subvolume snapshot -r` | Vytvoří read-only snapshot | `sudo btrfs subvolume snapshot -r /mnt/data /mnt/data/@snapshots/backup-1` | `Create a readonly snapshot` |
| `btrfs send` | Odesle snapshot (pro zálohování) | `sudo btrfs send /mnt/data/@snapshots/backup-1 > backup.btrfs` | `At subvol /mnt/data/@snapshots/backup-1` |
| `btrfs receive` | Príjme snapshot | `sudo btrfs receive /mnt/backup < backup.btrfs` | `At subvol backup.btrfs` |
| `btrfs delete` | Smaže subvolume/snapshot | `sudo btrfs subvolume delete /mnt/data/@snapshots/backup-1` | `Delete subvolume` |

**Praktický príklad: Snapshot rollback:**

```bash
# Vytvořit snapshot před aktualizací
sudo btrfs subvolume snapshot -r / /@root-pre-update

# Po aktualizaci: pokud se neco pokazí, vrátit
# 1. Restartovat do live USB
# 2. Prejmenovat subvolumes
sudo mv /@root /@root-broken
sudo mv /@root-pre-update /@root

# Nebo pouzít snapper pro automatickou správu snapshotu
sudo snapper -c root create -d "before-update"
sudo apt upgrade
sudo snapper -c root list
# Výstup:
# # | Type  | Pre # | Date                | Description
#---+-------+-------+---------------------+-------------
# 0 | single |       |                     | current
# 1 | pre    |       | 2025-03-10 14:00:00 | before-update
```

### 3.6 LVM - Logical Volume Manager

Pokročilá správa disků s možností dynamické změny velikosti.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `pvcreate` | Inicializuje fyzický svazek | `sudo pvcreate /dev/sdb1` | `Physical volume "/dev/sdb1" successfully created` |
| `vgcreate` | Vytvoří volume group | `sudo vgcreate vg_data /dev/sdb1` | `Volume group "vg_data" successfully created` |
| `lvcreate` | Vytvoří logický svazek | `sudo lvcreate -L 10G -n lv_home vg_data` | `Logical volume "lv_home" created` |
| `lvextend` | Zvětší logický svazek | `sudo lvextend -L +5G /dev/vg_data/lv_home` | `Size of logical volume changed from 10G to 15G` |
| `lvresize` | Zmenší logický svazek | `sudo lvresize -L -2G /dev/vg_data/lv_home` | `Logical volume lv_home successfully resized` |
| `pvdisplay` | Podrobnosti o fyzickém svazku | `sudo pvdisplay` | `PV Name /dev/sdb1`<br>`PV Size 100.00 GiB` |
| `vgdisplay` | Podrobnosti o volume group | `sudo vgdisplay` | `VG Name vg_data`<br>`Total PE 25599` |
| `lvdisplay` | Podrobnosti o logickém svazku | `sudo lvdisplay` | `LV Name lv_home`<br>`LV Size 10.00 GiB` |

**Příklad: Zvětšení LVM oddílu + FS:**

```bash
# 1. Zvetsit logický svazek
sudo lvextend -L +10G /dev/vg_data/lv_home

# 2. Zvetsit souborový systém (ext4)
sudo resize2fs /dev/vg_data/lv_home

# Výstup:
# resize2fs 1.46.5 (30-Dec-2021)
# Filesystem at /dev/vg_data/lv_home is mounted on /home
# The filesystem on /dev/vg_data/lv_home is now 5242880 blocks long

# Alternativa: provést oba kroky najednou
sudo lvextend -r -L +10G /dev/vg_data/lv_home
# -r automaticky provede resize2fs
```

---

## 4. Automatizace aktualizací

Automatické aktualizace jsou klíčové pro bezpečnost, ale vyzadují opatrné nastavení.

### 4.1 unattended-upgrades (Debian/Ubuntu)

Automatické instalace bezpečnostních aktualizací.

```bash
# Instalace
sudo apt install unattended-upgrades

# Konfigurace
sudo dpkg-reconfigure --priority=low unattended-upgrades

# Hlavní konfigurační soubor:
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

```bash
# Příklad konfigurace:
# /etc/apt/apt.conf.d/50unattended-upgrades

Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    "${distro_id}:${distro_codename}-updates";
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";

# Zobrazit stav a logy
cat /var/log/unattended-upgrades/unattended-upgrades.log
```

### 4.2 apt-daily a systemd timery

Systemd timery řídí automatické aktualizace na moderních distribucích.

```bash
# Zobrazit systemd timery související s apt
systemctl list-timers | grep apt

# Výstup:
# NEXT                         LEFT      LAST                         PASSED  UNIT
# Sun 2025-03-11 06:34:17 UTC  6h left   Sat 2025-03-10 18:52:27 UTC  5h ago  apt-daily.timer
# Sun 2025-03-11 06:50:17 UTC  6h left   Sat 2025-03-10 18:52:27 UTC  5h ago  apt-daily-upgrade.timer

# Zobrazit konfiguraci timeru
systemctl cat apt-daily.timer

# Výstup:
# [Unit]
# Description=Daily apt download activities
# [Timer]
# OnCalendar=*-*-* 6:00
# RandomizedDelaySec=12h
# Persistent=true

# Manuální spuštění
sudo systemctl start apt-daily.service

# Zakázání automatických aktualizací (nedoporučeno)
sudo systemctl disable --now apt-daily.timer apt-daily-upgrade.timer
```

### 4.3 dnf-automatic (Fedora/RHEL)

Automatické aktualizace pro dnf.

```bash
# Instalace
sudo dnf install dnf-automatic

# Konfigurace
sudo nano /etc/dnf/automatic.conf

# Příklad konfigurace:
# [commands]
# upgrade_type = security
# random_sleep = 360
# download_updates = yes
# apply_updates = yes
# emit_via = email

# Povolení a spuštění timeru
sudo systemctl enable --now dnf-automatic.timer

# Zobrazit stav
sudo systemctl status dnf-automatic.timer
```

### 4.4 Cron a anacron

Klasické plánování úloh pro periodické aktualizace.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `crontab -l` | Zobrazí cron úlohy | `crontab -l` | `0 3 * * 0 apt update && apt upgrade -y` |
| `crontab -e` | Upraví cron úlohy | `crontab -e` | (otevre editor) |
| `anacron` | Spouští úlohy i kdyz byl systém vypnut | `sudo systemctl status anacron` | `● anacron.service - Run anacron jobs` |

**Příklady cron úloh pro aktualizace:**

```bash
# Každou neděli ve 3:00 aktualizovat systém
0 3 * * 0 apt update && apt upgrade -y >> /var/log/weekly-update.log 2>&1

# Každý den v 2:00 aktualizovat snapy
0 2 * * * snap refresh >> /var/log/snap-update.log 2>&1

# Jednou měsíčně clean
0 4 1 * * apt autoremove --purge -y && apt clean

# Zkontrolovat, zda je potřeba restart
@reboot if [ -f /var/run/reboot-required ]; then echo "Reboot required"; fi

# anacron: úlohy, které se spustí i po vynechaném termínu
# /etc/anacrontab:
# @weekly  7   weekly-update  apt update && apt upgrade -y
```

### 4.5 Kontrolované updaty

Profesionální přístup k aktualizacím v produkčním prostředí.

```bash
#!/bin/bash
# staged-update.sh - kontrolovaný update

set -euo pipefail

LOG="/var/log/staged-update.log"
STAGE=${1:-"check"}

case "$STAGE" in
  check)
    echo "[$(date)] === FÁZE 1: Kontrola ===" | tee -a "$LOG"
    apt update
    apt list --upgradable 2>/dev/null
    echo "Balíčku k upgradu: $(apt list --upgradable 2>/dev/null | grep -c upgradable)"
    ;;

  download)
    echo "[$(date)] === FÁZE 2: Stažení ===" | tee -a "$LOG"
    apt install --download-only -y
    echo "Balíčky staženy, ready k instalaci."
    ;;

  apply)
    echo "[$(date)] === FÁZE 3: Instalace ===" | tee -a "$LOG"
    apt upgrade -y
    echo "Upgrade dokončen."
    ;;

  cleanup)
    echo "[$(date)] === FÁZE 4: Úklid ===" | tee -a "$LOG"
    apt autoremove --purge -y
    apt clean
    ;;

  status)
    echo "Poslední log:"
    tail -20 "$LOG"
    ;;
esac

# Použití:
# ./staged-update.sh check     # Zjistit co je k dispozici
# ./staged-update.sh download  # Stáhnout (neinstalovat)
# ./staged-update.sh apply     # Aplikovat
# ./staged-update.sh cleanup   # Uklidit
```

---

## 5. Praktické scénáře

### 5.1 Rollback balíčku

```bash
# Ubuntu/Debian: downgrade pomoci apt
# Nejdříve povolit staré verze v /etc/apt/sources.list
sudo apt install package-name=verze

# Příklad: vratit firefox na starší verzi
sudo apt install firefox=120.0

# Výstup:
# The following packages will be DOWNGRADED:
#   firefox
# 0 upgraded, 0 newly installed, 1 downgraded

# Nebo pomoci dpkg (pokud máme .deb soubor)
sudo dpkg -i firefox_120.0_amd64.deb
sudo apt-mark hold firefox  # Zamknout verzi (zabrání upgradu)
```

### 5.2 Zadrzení balíčku (hold/pin)

```bash
# apt-mark: označit balíček jako drzený (nebude upgradován)
sudo apt-mark hold firefox
sudo apt-mark showhold
# Výstup: firefox

# Zrušení drzení
sudo apt-mark unhold firefox

# Dnf: vyloučit balíček z upgradu
sudo dnf update --exclude=kernel,nvidia
# Nebo trvale v /etc/dnf/dnf.conf:
# exclude=kernel* nvidia*

# Pacman: ignorovat balíček
sudo pacman -Syu --ignore firefox
# Nebo trvale v /etc/pacman.conf:
# IgnorePkg = firefox
```

### 5.3 Kernel swap (výměna jádra)

```bash
# 1. Zjistit aktuální jádro
uname -r
# Výstup: 6.5.0-21-generic

# 2. Nainstalovat LTS jádro (Ubuntu)
sudo apt install linux-image-generic-hwe-22.04

# 3. Zobrazit dostupná jádra v GRUB
grep menuentry /boot/grub/grub.cfg

# 4. Nastavit výchozí jádro na starou verzi (pokud nove nesedí)
sudo nano /etc/default/grub
# GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.5.0-21-generic"

sudo update-grub

# 5. Odstranit stará jádra (po overení stability)
sudo apt autoremove --purge
# nebo rucně:
sudo apt remove linux-image-6.5.0-21-generic linux-headers-6.5.0-21-generic
```

### 5.4 Snapshot management s Btrfs a snapper

```bash
# Instalace snapper (nástroj pro správu Btrfs snapshotu)
sudo apt install snapper

# Vytvorit konfiguraci pro root
sudo snapper -c root create-config /

# Vytvorit snapshot před/po aktualizaci
sudo snapper -c root create -d "before-kernel-update" --type pre
sudo apt install linux-image-6.8.0-35-generic
sudo snapper -c root create -d "after-kernel-update" --type post

# Zobrazit snapshoty
sudo snapper -c root list

# Výstup:
# # | Type   | Pre # | Date                | Description
#---+--------+-------+---------------------+--------------------
# 0 | single |       |                     | current
# 1 | pre    |       | 2025-03-10 14:00:00 | before-kernel-update
# 2 | post   |   1   | 2025-03-10 14:05:00 | after-kernel-update

# Rollback k snapshotu #1 (pred aktualizací)
sudo snapper -c root undochange 1..0

# Smazat snapshot
sudo snapper -c root delete 2
```

### 5.5 Komplexní update script

```bash
#!/bin/bash
# full-system-maintenance.sh
# Kompletní údrzba systému

set -euo pipefail

LOG_DIR="/var/log/system-maintenance"
mkdir -p "$LOG_DIR"
DATE=$(date +%Y%m%d-%H%M%S)
LOG="$LOG_DIR/maintenance-$DATE.log"

log() {
    echo "[$(date '+%H:%M:%S')] $*" | tee -a "$LOG"
}

# 1. APT
log "=== APT update ==="
sudo apt update >> "$LOG" 2>&1
log "=== APT upgrade ==="
sudo apt upgrade -y >> "$LOG" 2>&1
log "=== APT autoremove ==="
sudo apt autoremove --purge -y >> "$LOG" 2>&1

# 2. SNAP
if command -v snap &>/dev/null; then
    log "=== Snap refresh ==="
    sudo snap refresh >> "$LOG" 2>&1
fi

# 3. FLATPAK
if command -v flatpak &>/dev/null; then
    log "=== Flatpak update ==="
    flatpak update -y >> "$LOG" 2>&1
fi

# 4. PIP (user)
if command -v pip3 &>/dev/null; then
    log "=== pip outdated ==="
    pip3 list --outdated --format=columns >> "$LOG" 2>&1
fi

# 5. Kernel check
log "=== Kernel ==="
uname -r >> "$LOG"

# 6. Disk space
log "=== Disk usage ==="
df -h >> "$LOG" 2>&1

# 7. Reboot check
if [ -f /var/run/reboot-required ]; then
    log "!!! REBOOT REQUIRED !!!"
    cat /var/run/reboot-required >> "$LOG"
fi

log "=== Maintenance complete ==="

echo "Log: $LOG"
```

---

## Shrnutí

### Porovnání balíčkovacích systému

| Systém | Distribuce | Formát | Výhody | Nevýhody |
|--------|-----------|--------|--------|---------|
| **apt** | Debian, Ubuntu | `.deb` | Nejrozsáhlejší repozitáře, stabilní | Pomalejší nekteré operace |
| **dnf** | Fedora, RHEL | `.rpm` | Rychlý, dobré rozlišení závislostí | Mensí repozitáře nez apt |
| **pacman** | Arch, Manjaro | `.pkg.tar.zst` | Rolling release, vždy nejnovější | Méně stabilní, vyzaduje znalosti |
| **snap** | Univerzální | `.snap` | Sandboxing, automatické updaty | Pomalejší spouštění, velikost |
| **flatpak** | Univerzální | `.flatpak` | Sandboxing, nezávislý na distribuci | Zabírá místo, ne pro CLI nástroje |

### Klíčové body

1. **Pravidelnost** - Aktualizace by mely probíhat pravidelně (tydně bezpecnostní, měsíčně plné)
2. **Záložní plán** - Vzdy mít moznost rollbacku (snapshot, záloha, staré jádro v GRUB)
3. **Testování** - Na produkci testovat na stagingu, pouzít staged updates
4. **Bezpečnost** - Bezpecnostní aktualizace jsou priorita (zero-day exploity)
5. **Automatizace vs. kontrola** - Unattended-upgrades pro bezpecnost, rucní pro velké zmeny
6. **Jádro** - Drzet se LTS jader na produkci, testovat nová na vývojových strojích
7. **Souborové systémy** - Před manipulací s FS (fsck, resize) vzdy zalohovat data

### Uzitečné příkazy na konec

```bash
# Kdy byl naposledy aktualizován systém?
ls -lt /var/log/apt/history.log | head -1

# Zjistit velikost balíčkové cache
du -sh /var/cache/apt/archives

# Kdy systém naposledy restartoval?
uptime -s

# Jak dlouho bezi jádro?
uptime

# Která jádra jsou nainstalovaná?
ls -1 /boot/vmlinuz-* | sed 's|/boot/vmlinuz-||'

# Je potřeba restart?
test -f /var/run/reboot-required && echo "YES" || echo "NO"

# Výpis všech service souvisejících s aktualizacemi
systemctl list-units --type=service | grep -E "apt|dnf|snap|flatpak"

---

➡️ [Zpět na přehled](README.md)
```
