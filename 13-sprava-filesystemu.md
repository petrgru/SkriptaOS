# Správa souborového systému

> Filesystem Management in Linux

---

## Úvod

Správa souborového systému v Linuxu zahrnuje práci s disky, oddíly, vytvářením a připojováním filesystémů, kontrolou integrity a pokročilými technikami jako LVM. Tato kapitola pokrývá praktické nástroje a postupy pro každodenní správu úložišť.

---

## 1. Zobrazení disků a oddílů

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `lsblk` | Bloková zařízení (stromově) | `lsblk` | `NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT`<br>`sda      8:0    0  100G  0 disk`<br>`├─sda1   8:1    0   50G  0 part /` |
| `blkid` | UUID a typ FS | `blkid` | `/dev/sda1: UUID="abc-123" TYPE="ext4"` |
| `fdisk -l` | Výpis oddílů | `sudo fdisk -l` | `Device     Boot  Start      End  Sectors  Size Id Type`<br>`/dev/sda1  *      2048  5000000  2.4G 83 Linux` |
| `df -h` | Využití disků | `df -h` | `/dev/sda1  50G  20G  28G  42% /` |
| `df -T` | Využití s typem FS | `df -hT` | `/dev/sda1  ext4  50G  20G  28G  42% /` |
| `df -i` | Využití inodů | `df -hi` | `/dev/sda1  3.2M  1.1M  2.1M  35% /` |
| `parted -l` | Výpis oddílů (GPT) | `sudo parted -l` | `Number  Start   End     Size   Type     File system`<br>` 1      1049kB  53.7GB  primary  ext4` |

---

## 2. Správa oddílů

**MBR vs GPT:** MBR max 2TB, 4 primary oddíly, BIOS. GPT max 9.4ZB, 128 primary oddílů, UEFI, záložní tabulka.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `fdisk /dev/sda` | Interaktivní nástroj (MBR) | `sudo fdisk /dev/sda` | `Command (m for help):` |
| `gdisk /dev/sda` | Interaktivní nástroj (GPT) | `sudo gdisk /dev/sda` | `Command (? for help):` |
| `parted` | Pokročilá správa (MBR+GPT) | `sudo parted /dev/sda` | `(parted)` |

```bash
# Vytvoření GPT oddílu
sudo parted /dev/sdb --script mklabel gpt
sudo parted /dev/sdb --script mkpart primary ext4 1MiB 100%

# Změna typu na Linux LVM (fdisk: t → 8E → w)
# Smazání oddílu (fdisk: d → w)
```

---

## 3. Vytváření souborových systémů

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `mkfs.ext4` | Ext4 souborový systém | `sudo mkfs.ext4 -L DATA /dev/sdb1` | `Creating filesystem with 2621440 4k blocks ... done` |
| `mkfs.xfs` | XFS souborový systém | `sudo mkfs.xfs -f -L DATA /dev/sdb1` | `meta-data=/dev/sdb1  isize=512  agcount=4` |
| `mkfs.btrfs` | Btrfs souborový systém | `sudo mkfs.btrfs -L DATA /dev/sdb1` | `Btrfs v6.6.3, Creating regular btrfs filesystem` |
| `mkswap` | Swapovací oddíl | `sudo mkswap -L SWAP /dev/sdc1` | `Setting up swapspace version 1, size = 2 GiB` |
| `mkfs.vfat` | FAT32 | `sudo mkfs.vfat -n USB /dev/sdd1` | `mkfs.fat 4.2 (2021-01-31)` |
| `mkfs.ntfs` | NTFS | `sudo mkfs.ntfs -L DATA /dev/sde1` | `Cluster size has been automatically set to 4096 bytes.` |

Parametry `mkfs.ext4`: `-L label` (štítek), `-b 4096` (velikost bloku), `-m 1` (rezervace pro root v %, výchozí 5).

```bash
# Kompletní příprava disku
parted /dev/sdb --script mklabel gpt mkpart primary ext4 0% 100%
mkfs.ext4 -L DATA -m 1 /dev/sdb1
blkid /dev/sdb1
# Výstup: /dev/sdb1: LABEL="DATA" UUID="abcd1234" TYPE="ext4"
```

---

## 4. Připojování FS (mount, /etc/fstab)

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `mount` | Zobrazí připojené FS | `mount` | `/dev/sda1 on / type ext4 (rw,relatime)` |
| `mount [dev] [mountpoint]` | Připojí oddíl | `sudo mount /dev/sdb1 /mnt/data` | (žádný výstup) |
| `mount -t ext4` | Určení typu FS | `sudo mount -t ext4 /dev/sdb1 /mnt` | (žádný výstup) |
| `mount -o noatime,ro` | Možnosti připojení | `sudo mount -o ro /dev/sdb1 /mnt` | (žádný výstup) |
| `umount [mountpoint]` | Odpojení | `sudo umount /mnt/data` | (žádný výstup) |
| `umount -l` | Lazy odpojení | `sudo umount -l /mnt/data` | (žádný výstup) |
| `umount -f` | Vynucené odpojení | `sudo umount -f /mnt/data` | (žádný výstup) |
| `mount -a` | Připojí vše z /etc/fstab | `sudo mount -a` | (žádný výstup) |
| `findmnt` | Strom mount pointů | `findmnt` | `TARGET  SOURCE     FSTYPE OPTIONS`<br>`/       /dev/sda1  ext4   rw,relatime` |

**Mount volby:** `noatime` (neaktualizuje čas přístupu), `relatime` (výchozí), `ro` (read-only), `rw`, `sync`, `discard` (TRIM pro SSD).

**/etc/fstab** struktura: `<file system> <mount point> <type> <options> <dump> <pass>`

```bash
# Příklad /etc/fstab
# UUID=abc123  /        ext4  defaults  0  1
# UUID=def456  /home    ext4  defaults  0  2
# UUID=789abc  none     swap  sw        0  0

# Přidání disku pomocí UUID (stabilní napříč bootováním)
sudo blkid /dev/sdb1    # Získat UUID
echo "UUID=abcd-1234  /mnt/data  ext4  defaults  0  2" | sudo tee -a /etc/fstab
sudo mount -a           # Otestovat bez restartu
```

---

## 5. Kontrola a oprava FS

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `fsck [device]` | Kontrola FS (auto dle typu) | `sudo fsck /dev/sdb1` | `/dev/sdb1: clean, 12/65536 files` |
| `e2fsck -f` | Vynucená kontrola ext | `sudo e2fsck -f /dev/sda1` | `Pass 1: Checking inodes, blocks, and sizes` |
| `e2fsck -p` | Automatická oprava | `sudo e2fsck -fp /dev/sda1` | `check forced.` |
| `e2fsck -n` | Suchý běh (jen čtení) | `sudo e2fsck -fn /dev/sda1` | `Pass 1: Checking inodes...` |
| `xfs_repair` | Oprava XFS (odmountovat!) | `sudo xfs_repair /dev/sdb1` | `Phase 1 - find and verify superblock...` |
| `badblocks` | Vadné sektory | `sudo badblocks -sv /dev/sdb1` | `Checking blocks 0 to 1048575` |
| `dumpe2fs` | Info o ext FS | `sudo dumpe2fs -h /dev/sda1` | `Block count: 6553600` |

```bash
# FS musí být odpojen!
sudo umount /dev/sdb1
sudo e2fsck -f -v /dev/sdb1
# Výstup: 25/65536 files (4.0% non-contiguous), 12345/262144 blocks

sudo xfs_repair /dev/sdc1
# Phase 1 - find and verify superblock... done
```

---

## 6. LVM základy

**Vrstvy:** Fyzické disky → PV (Physical Volumes) → VG (Volume Group) → LV (Logical Volumes) → FS

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `pvcreate` | Vytvoří physical volume | `sudo pvcreate /dev/sdb1` | `Physical volume "/dev/sdb1" successfully created` |
| `vgcreate` | Vytvoří volume group | `sudo vgcreate vg_data /dev/sdb1` | `Volume group "vg_data" successfully created` |
| `lvcreate -L` | Vytvoří logical volume | `sudo lvcreate -L 10G -n lv_home vg_data` | `Logical volume "lv_home" created` |
| `lvextend -L +` | Rozšíří LV | `sudo lvextend -L +5G /dev/vg_data/lv_home` | `Size changed from 10G to 15G` |
| `resize2fs` | Rozšíří ext4 FS po LV | `sudo resize2fs /dev/vg_data/lv_home` | `The filesystem is now 5242880 blocks long` |
| `pvs` | Zkrácené info o PV | `sudo pvs` | `PV         VG     PSize   PFree`<br>`/dev/sdb1  vg_data 100.00g 90.00g` |
| `vgs` | Zkrácené info o VG | `sudo vgs` | `VG      #PV #LV VSize   VFree`<br>`vg_data   1   1 100.00g 90.00g` |
| `lvs` | Zkrácené info o LV | `sudo lvs` | `LV      VG     LSize`<br>`lv_home vg_data 10.00g` |
| `pvdisplay` | Detail PV | `sudo pvdisplay` | `PV Name /dev/sdb1, PV Size 100.00 GiB` |
| `vgdisplay` | Detail VG | `sudo vgdisplay` | `VG Name vg_data, Total PE 25599` |
| `lvdisplay` | Detail LV | `sudo lvdisplay` | `LV Name lv_home, LV Size 10.00 GiB` |

```bash
# Vytvoření LVM a FS
sudo pvcreate /dev/sdb1
sudo vgcreate vg_data /dev/sdb1
sudo lvcreate -L 10G -n lv_home vg_data
sudo mkfs.ext4 /dev/vg_data/lv_home
sudo mount /dev/vg_data/lv_home /mnt/home

# Rozšíření LV i FS jedním příkazem
sudo lvextend -r -L +5G /dev/vg_data/lv_home
# Výstup: Size changed from 10.00 GiB to 15.00 GiB; FS now 5242880 blocks
```

> **⚠️ VAROVÁNÍ:** Příkazy `lvremove`, `pvremove`, `vgremove` nenávratně ničí data. Vždy zálohujte před destruktivními operacemi s LVM.

---

## 7. Swap

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `swapon [device]` | Aktivuje swap | `sudo swapon /dev/sdc1` | (žádný) |
| `swapon -s` | Zobrazí aktivní swapy | `swapon -s` | `Filename  Type  Size  Used  Priority`<br>`/dev/sdc1 partition 2G 0 -2` |
| `swapon -p` | Priorita swapu | `sudo swapon -p 10 /dev/sdc1` | (žádný) |
| `swapoff` | Deaktivuje swap | `sudo swapoff /dev/sdc1` | (žádný) |

**Vytvoření swapfile (dnes běžnější než swap oddíl):**

```bash
# Vytvoření 4GB swapfile
dd if=/dev/zero of=/swapfile bs=1M count=4096
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
swapon --show
# Výstup: NAME       TYPE SIZE USED PRIO
#         /swapfile  file   4G   0B   -2

# Trvalá registrace
echo "/swapfile  none  swap  sw  0  0" | sudo tee -a /etc/fstab

# Priorita v /etc/fstab
# UUID=xxx none swap sw,pri=10 0 0
```

---

## Shrnutí

### Porovnání FS

| Vlastnost | ext4 | xfs | btrfs |
|-----------|------|-----|-------|
| Max velikost souboru | 16 TB | 8 EB | 16 EB |
| Max velikost FS | 1 EB | 8 EB | 16 EB |
| Snapshoty | Ne (LVM) | Ne (LVM) | Ano (nativní) |
| Komprese | Ne | Ne | Ano |
| Vhodné pro | Obecné, boot | Velké soubory, servery | Kontejnery, snapshoty |

### Klíčové body

1. **lsblk a blkid** – první příkazy pro zjištění diskové konfigurace
2. **GPT preferujte** před MBR na nových systémech (UEFI, disky >2TB)
3. **UUID používejte** v /etc/fstab místo /dev/sdX (stabilní napříč rebooty)
4. **FS vždy odpojte** před fsck nebo xfs_repair
5. **LVM** dává flexibilitu při změně velikosti oddílů bez rebootu
6. **Swapfile** je dnes běžnější a flexibilnější než swap oddíl

```bash
# Užitečné příkazy na konec
blkid /dev/sda1          # UUID disku
findmnt --df             # Rychlé zobrazení mount pointů
lsblk -f                 # Typ FS na oddílech
sudo vgs                 # Volné místo v LVM
swapon --show            # Aktivní swap
file -s /dev/sda1        # Typ FS na oddílu
```

---

➡️ [Zpět na přehled](README.md)
