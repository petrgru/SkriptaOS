# RAID a Redundance disků

> RAID and Disk Redundancy Solutions

---

## Úvod

Redundance disků je technika, která chrání data před ztrátou při selhání fyzického disku (HDD nebo SSD). Zatímco klasické zálohování (viz [kapitola 14](14-zalohovani.md)) řeší obnovu dat v čase – například po smazání souboru, ransomware útoku nebo fyzickém zničení – redundance řeší **dostupnost**. Redundantní uložení dat zajistí, že systém zůstane v provozu i při výpadku jednoho nebo více disků.

Je důležité chápat, že redundance **nenahrazuje zálohování**. Redundantní pole vás neochrání před:
- omylem smazaným souborem,
- ransomwarem, který zašifruje připojené disky,
- požárem nebo krádeží zařízení.

Na tyto případy slouží záloha – ideálně podle 3-2-1 pravidla popsaného v [kapitole 14](14-zalohovani.md).

Tato kapitola pokrývá pět hlavních technologií pro redundantní ukládání dat v Linuxu (Ubuntu/Debian):
- **mdadm** – klasické softwarové RAID pole přímo v jádře
- **LVM+RAID** – Logical Volume Manager s vrstvou dm-raid
- **ZFS** – pokročilý souborový systém a správce svazků (OpenZFS)
- **BTRFS** – moderní copy-on-write souborový systém
- **SnapRAID** – paritní ochrana pro nenáročná úložiště

Všechny příkazy v této kapitole vyžadují `sudo`.

---

## Srovnání technologií redundance

| Vlastnost | mdadm | LVM+RAID | ZFS | BTRFS | SnapRAID |
|-----------|-------|----------|-----|-------|----------|
| Úroveň RAID | 0,1,5,6,10 | 0,1,5,6,10 | mirror,RZD1/2/3 | 0,1,10,5,6,DUP | single/dual |
| Výpočet parity | md/XOR | CPU | CPU (COW) | CPU (COW) | CPU (sync) |
| Copy-on-Write | ✗ | ✗ | ✓ | ✓ | ✗ |
| Komprese | ✗ | ✗ | LZ4,ZSTD,gzip | ZSTD,LZO,ZLIB | ✗ |
| Snapshoty | ✗ | ✓ | ✓ | ✓ | ✗ |
| Deduplikace | ✗ | ✗ | ✓ (volitelně) | ✗ | ✗ |
| Samooprava | ✗ | ✗ | ✓ | ✓ | ✓ |
| Online rozšiř. | ✓ | ✓ | ✓ | ✓ | ✓ |
| Různé velikosti | omezeně | ✓ | ✓ | ✓ | ✓ |
| TRIM/discard | RAID1/10 jen | ✗ | ✓ | ✓ | ✓ |
| Cache vrstva | ✗ | ✗ | ARC,L2ARC,SLOG | ✗ | ✗ |
| Náročnost RAM | Nízká | Nízká | Vysoká (~1GB/TB) | Střední | Nízká |
| Výkon zápisu | Vysoký | Vysoký | Střední | Střední | Vysoký |
| Výkon čtení | Vysoký | Vysoký | Vysoký (ARC) | Vysoký | Rychlý |
| Max. kapacita | dle limitu md | dle VG/LV | 256 ZiB/pool | 16 EiB/FS | dle konf. |
| Vhodné použití | Servery,NAS | Flexibilní disky | Enterprise NAS,DB | NAS, root FS | Media,archiv |
| Licence | GPL | GPL | CDDL | GPL | GPLv3 |

---

## Software RAID (mdadm)

Instalace: `apt install mdadm`

Vytvoření RAID 1:
```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
sudo mkfs.ext4 /dev/md0
sudo mount /dev/md0 /mnt/raid1
```

> **⚠️ VAROVÁNÍ:** `mdadm --create` zničí všechna existující data na zadaných discích. Pro čisté disky ve VM je použití `/dev/sdX` v pořádku; v produkci používejte `/dev/disk/by-id/` (stabilní názvy napříč rebooty).

Vytvoření RAID 5:
```bash
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
```

| Úroveň | Min. disků | Kapacita | Odolnost |
|--------|-----------|----------|----------|
| RAID 0 | 2 | N × min(disk) | ✗ (0 disků) |
| RAID 1 | 2 | 1 × min(disk) | ✓ (1 disk) |
| RAID 5 | 3 | (N-1) × min(disk) | ✓ (1 disk) |
| RAID 6 | 4 | (N-2) × min(disk) | ✓ (2 disky) |
| RAID 10 | 4 | N/2 × min(disk) | ✓ (1-2 disky) |

Monitorování:
```bash
cat /proc/mdstat
mdadm --detail /dev/md0
```

Uložení konfigurace:
```bash
sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

Simulace selhání a obnova:
```bash
sudo mdadm --manage /dev/md0 --fail /dev/sdb
sudo mdadm --manage /dev/md0 --remove /dev/sdb
sudo mdadm --manage /dev/md0 --add /dev/sdb
```

---

## LVM (Logical Volume Manager)

Instalace: `apt install lvm2` (součástí jádra Linuxu)

**Překryv s kap. 13**: Základy LVM (PV→VG→LV) jsou popsány v [kapitole 13](13-sprava-filesystemu.md#6-lvm-základy). Zde uvádíme pouze rozšiřující techniky.

LVM snapshoty:
```bash
sudo lvcreate -L 10G -s -n snap_home /dev/vg_data/lv_home

# Obnova ze snapshotu (LV musí být odmountovaný!)
sudo umount /dev/vg_data/lv_home
sudo lvconvert --merge /dev/vg_data/snap_home
```
> **⚠️ VAROVÁNÍ:** `lvconvert --merge` vyžaduje odmountovaný filesystém a nenávratně nahradí obsah LV obsahem snapshotu.

LVM striping (RAID0 přes LV):
```bash
sudo lvcreate -L 100G -n lv_stripe vg_data -i 2 /dev/sdb /dev/sdc
```

LVM mirroring (RAID1):
```bash
sudo lvcreate -m1 -L 50G -n lv_mirror vg_data
```

dm-raid: LVM + RAID kombinace (využívá mdadm vrstvu pod LVM):
```bash
# Vytvoření RAID1 pomocí dm-raid
sudo lvcreate --type raid1 -L 100G -n lv_raid1 vg_data
```

Tabulka LVM příkazů:
| Příkaz | Popis | Příklad |
|--------|-------|---------|
| `lvcreate -s` | Vytvoří snapshot | `lvcreate -L 10G -s -n snap /dev/vg/lv` |
| `lvconvert --merge` | Sloučí snapshot zpět | `lvconvert --merge /dev/vg/snap` |
| `lvcreate -i N` | Striping (RAID0) | `lvcreate -i 2 -L 100G -n stripe vg` |
| `lvcreate -m1` | Mirroring (RAID1) | `lvcreate -m1 -L 50G -n mirror vg` |
| `lvcreate --type raid1` | RAID1 přes dm-raid | `lvcreate --type raid1 -L 100G -n raid vg` |
| `lvremove` | Smaže LV | `lvremove /dev/vg/lv` |

> **⚠️ VAROVÁNÍ:** `lvremove`, `pvremove`, `vgremove` nenávratně ničí data. Vždy zálohujte před destruktivními operacemi.

---

## ZFS (OpenZFS)

Instalace: `apt install zfsutils-linux` (OpenZFS, vyžaduje DKMS modul)

Základní koncepty: **pool** (zpool) → **dataset** → **filesystem**

Vytvoření poolu:
> **⚠️ VAROVÁNÍ:** `zpool create` nenávratně zničí data na discích. V produkci vždy používejte `/dev/disk/by-id/` (např. `/dev/disk/by-id/ata-WDC-...`), protože `/dev/sdX` se může po rebootu změnit a ZFS by nenašlo disk.
```bash
# Mirror (RAID1) – 2 disky
sudo zpool create tank mirror /dev/sdb /dev/sdc

# RAIDZ1 (jako RAID5, 1 paritní disk) – 3 disky
sudo zpool create tank raidz /dev/sdb /dev/sdc /dev/sdd

# RAIDZ2 (jako RAID6, 2 paritní disky) – 4 disky
sudo zpool create tank raidz2 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

Vytvoření datasetu a komprese:
```bash
sudo zfs create tank/data
sudo zfs set compression=lz4 tank/data  # LZ4 – doporučená, téměř bez režie
```

Snapshoty a rollback:
```bash
sudo zfs snapshot tank/data@20250101
sudo zfs list -t snapshot
sudo zfs rollback tank/data@20250101
```

Scrubbing a monitoring:
```bash
sudo zpool scrub tank        # Kontrola integrity všech dat
zpool status                  # Stav poolu
zpool list                    # Seznam poolů
zfs list                      # Seznam datasetů
```

Cache a SLOG:
```bash
sudo zpool add tank cache /dev/sde  # L2ARC – SSD cache pro čtení
sudo zpool add tank log /dev/sdf     # SLOG – rychlé ZIL pro zápis
```

Výměna disku:
```bash
sudo zpool replace tank /dev/sdb /dev/sdg
```

| Příkaz | Popis | Příklad |
|--------|-------|---------|
| `zpool create` | Vytvoří pool | `zpool create tank mirror /dev/sdb /dev/sdc` |
| `zfs create` | Vytvoří dataset | `zfs create tank/data` |
| `zfs set compression=lz4` | Nastaví kompresi | `zfs set compression=lz4 tank/data` |
| `zfs snapshot` | Vytvoří snapshot | `zfs snapshot tank/data@snap1` |
| `zfs rollback` | Vrátí dataset ke snapshotu | `zfs rollback tank/data@snap1` |
| `zpool scrub` | Kontrola integrity | `zpool scrub tank` |
| `zpool status` | Stav poolu | `zpool status` |
| `zpool replace` | Výměna disku | `zpool replace tank old new` |
| `zpool add cache` | Přidá L2ARC cache | `zpool add tank cache /dev/sdX` |

Požadavky na RAM:
- ZFS ARC (Adaptive Replacement Cache) defaultně zabírá až **50 % RAM**
- Doporučení: min 1 GB RAM na 1 TB úložiště
- Omezení ARC na systémech s 4–8 GB RAM:
```bash
echo "options zfs zfs_arc_max=2147483648" | sudo tee /etc/modprobe.d/zfs.conf  # 2 GB limit
```

> **ℹ️ POZNÁMKA:** OpenZFS je licencován pod CDDL, která není kompatibilní s GPL. ZFS modul proto není součástí jádra Linuxu a je nutné jej instalovat samostatně (DKMS).

---

## BTRFS

Instalace: `apt install btrfs-progs` (součástí jádra Linuxu)

Vytvoření filesystému:
```bash
# Single disk – metadata DUP (doporučeno pro HDD)
sudo mkfs.btrfs -m dup /dev/sdb

# RAID1 – 2 disky (metadata i data zrcadleny)
sudo mkfs.btrfs -d raid1 -m raid1 /dev/sdb /dev/sdc

# RAID10 – 4 disky (striping + mirror)
sudo mkfs.btrfs -d raid10 -m raid10 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

Subvolumes (oddělené "kořeny" v rámci FS):
```bash
sudo mount /dev/sdb /mnt/data
sudo btrfs subvolume create /mnt/data/@home
sudo btrfs subvolume create /mnt/data/@var
```

Snapshoty:
```bash
sudo btrfs subvolume snapshot /mnt/data /mnt/data/snap-20250101
sudo btrfs subvolume snapshot -r /mnt/data /mnt/data/snap-ro  # read-only
```

Komprese (při mount):
```bash
sudo mount -o compress=zstd /dev/sdb /mnt/data
# Nebo v /etc/fstab:
# UUID=xxx /mnt/data btrfs compress=zstd 0 0
```

Scrub a defragmentace:
```bash
sudo btrfs scrub start /mnt/data    # Kontrola integrity
sudo btrfs filesystem defragment -r /mnt/data  # Defragmentace
```

Přidání disku a rebalance:
```bash
sudo btrfs device add /dev/sdd /mnt/data                    # Přidá disk
sudo btrfs balance start -dconvert=raid1 /mnt/data          # Změní RAID úroveň na RAID1
```

Zjištění stavu:
```bash
btrfs filesystem show
btrfs filesystem df /mnt/data
```

| Příkaz | Popis | Příklad |
|--------|-------|---------|
| `mkfs.btrfs` | Vytvoří BTRFS | `mkfs.btrfs -d raid1 -m raid1 /dev/sdb /dev/sdc` |
| `btrfs subvolume create` | Vytvoří subvolume | `btrfs subvolume create /mnt/data/@home` |
| `btrfs subvolume snapshot` | Vytvoří snapshot | `btrfs subvolume snapshot /mnt/data /mnt/data/snap` |
| `btrfs scrub start` | Kontrola integrity | `btrfs scrub start /mnt/data` |
| `btrfs balance` | Změna RAID profilu | `btrfs balance start -dconvert=raid1 /mnt/data` |
| `btrfs device add` | Přidá disk do FS | `btrfs device add /dev/sdd /mnt/data` |
| `btrfs filesystem show` | Zobrazí všechny BTRFS | `btrfs filesystem show` |

> **⚠️ VAROVÁNÍ:** RAID5 a RAID6 v BTRFS jsou stále experimentální. Při selhání disku kombinovaném s výpadkem napájení může dojít ke ztrátě dat. Pro produkční použití preferujte RAID1, RAID10 nebo DUP.

Vhodné použití: root filesystém, běžné NAS, automatické snapshoty (snapper pro apt/yum).

---

## SnapRAID

Instalace: `apt install snapraid`

Koncepce: SnapRAID není klasický RAID v reálném čase. Pracuje na principu **periodického počítání parity** – data zůstávají na jednotlivých discích v běžných souborových systémech (ext4, XFS atd.) a paritní informace se počítají při spuštění `snapraid sync`.

Výhody oproti klasickému RAID:
- **Disky nemusí být stejně velké** – každý disk se využívá v plné kapacitě
- Při selhání více disků, než je parita, data na ostatních discích zůstávají v pořádku (na rozdíl od RAID5/6, kde je vše ztraceno)
- Smazání souboru nepoškodí paritu – na rozdíl od RAID5/6, kde chybějící data při re-sync způsobí poškození parity
- Lze kombinovat disky libovolných velikostí a typů souborových systémů

Konfigurace `/etc/snapraid.conf`:
```bash
# Data disky (každý má vlastní řádek)
data d1 /dev/sdb
data d2 /dev/sdc
data d3 /dev/sdd

# Paritní disky
parity /dev/sde
2-parity /dev/sdf        # Druhá parita (volitelné, pro ochranu 2 disků)

# Obsahový soubor (metadaty)
content /var/snapraid/snapraid.content
```

Inicializace a synchronizace:
```bash
sudo snapraid init        # Prvotní inicializace (vytvoří parity)
sudo snapraid sync        # Pravidelná synchronizace (aktualizuje parity)
```

> **⚠️ VAROVÁNÍ:** `snapraid init` přepíše existující data na paritních discích. Před inicializací zkontrolujte, že paritní disky neobsahují důležitá data.

Scrubbing a obnova:
```bash
sudo snapraid scrub       # Kontrola konzistence parity a dat
sudo snapraid fix         # Interaktivní obnova (zeptá se co obnovit)
sudo snapraid -d d1 fix   # Obnova konkrétního disku (d1)
```

Status:
```bash
sudo snapraid status
```

Automatizace (cron):
```bash
# Denní synchronizace v 2:00, týdenní scrub v neděli 3:00
0 2 * * * /usr/bin/snapraid sync
0 3 * * 0 /usr/bin/snapraid scrub
```

> **ℹ️ POZNÁMKA:** SnapRAID není real-time – parita se počítá pouze při `snapraid sync`. Data zapsaná mezi jednotlivými synchronizacemi nejsou chráněna. Vhodné pro data, která se často nemění (media server, archiv).

| Příkaz | Popis | Příklad |
|--------|-------|---------|
| `snapraid init` | Inicializace parity | `sudo snapraid init` |
| `snapraid sync` | Synchronizace parity | `sudo snapraid sync` |
| `snapraid scrub` | Kontrola konzistence | `sudo snapraid scrub` |
| `snapraid fix` | Obnova dat | `sudo snapraid -d d1 fix` |
| `snapraid status` | Stav SnapRAIDu | `sudo snapraid status` |

Vhodné pro: media servery, archivy, zálohy, data s převážně čtením.

---

## Shrnutí

### Volba technologie podle scénáře

| Scénář | Doporučení | Zdůvodnění |
|--------|-----------|------------|
| Rychlé úložiště, VM datastore | RAID10 (mdadm) | Nejvyšší výkon zápisu i čtení |
| Flexibilní oddíly, live resize | LVM | Snadné rozšiřování, snapshoty |
| Enterprise NAS, databáze | ZFS | Silná integrita dat, komprese, ARC cache |
| Běžný NAS, root FS, desktop | BTRFS | Nativní v jádře, snapshoty, komprese |
| Media server, archiv | SnapRAID | Různé velikosti disků, nízká režie |
| Maximum integrity (až 3 parity) | ZFS RAIDZ2/3 | End-to-end kontrolní součty |
| Minimum nákladů na kapacitu | SnapRAID / RAID5 | Efektivní využití místa |
| Proxmox VE server | RAID1 (mdadm) + LVM | Viz kapitola 18 – praktický návod |

### Doporučené kombinace technologií

| Technologie | Použití | Ideální kombinace |
|-------------|---------|-------------------|
| mdadm RAID1 | Operační systém, boot | LVM na RAID pro flexibilitu oddílů |
| ZFS | Data, NAS, databáze | rsync záloha na jiný stroj (kap. 14) |
| BTRFS | Root FS, vývojový server | snapper pro automatické snapshoty |
| SnapRAID | Media archiv, zálohy | mergerfs pro pooling disků |

### Klíčové body

1. **Redundance není záloha** – redundance řeší dostupnost při selhání hardwaru, nechrání před smazáním, ransomwarem nebo živelnou pohromou. Pro tyto případy použijte 3-2-1 strategii z [kapitoly 14](14-zalohovani.md).
2. **Žádná technologie není univerzální** – každá má silné a slabé stránky. Výběr závisí na požadavcích na výkon, kapacitu, integritu dat a dostupný hardware.
3. **Pravidelná údržba je klíčová** – monitorujte stav RAID polí, provádějte scrub a včas vyměňujte předpovězené selhávající disky (SMART).
4. **Pro produkční nasazení** vždy používejte `/dev/disk/by-id/` místo `/dev/sdX` – předejdete problémům při přeuspořádání disků po restartu.
5. **Praxe**: Pro většinu domácích a malých firemních NAS je ideální volbou BTRFS (jednoduchost) nebo ZFS (maximální integrita). Pro servery s flexibilními oddíly je osvědčená kombinace mdadm RAID1 + LVM.

### Praktický příklad

Kompletní návod na nastavení Proxmox VE s RAID1 + LVM najdete v [kapitole 18](18-proxmox-raid.md).

---

➡️ [Zpět na přehled](README.md)
