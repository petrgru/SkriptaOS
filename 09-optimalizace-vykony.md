# Optimalizace výkonu OS (Linux Kernel Tuning)

> **Rozsah:** CPU, paměť, disk, síť, kernel parametry, rozdíly server vs workstation  
> **Cíl:** Získat maximum z hardware správným nastavením kernelu a systémových služeb

---

## Úvod

Optimalizace výkonu operačního systému je klíčová pro dosažení maximální efektivity hardwaru. Tato kapitola se zabývá laděním CPU, správy paměti, diskových I/O operací, síťového stacku a kernel parametrů pro různé typy zátěže.

## CPU

Moderní Linux kernel nabízí desítky parametrů pro řízení frekvence, plánování vláken a přidělování jader. Vyladěním CPU lze snížit latenci, zvýšit propustnost nebo ušetřit energii.

### Scaling Governor

Governor rozhoduje, na jaké frekvenci CPU běží v závislosti na zátěži.

| Parametr | Popis | Server | Workstation | Příklad |
|----------|-------|--------|-------------|---------|
| `performance` | Frekvence vždy na maximu – nejnižší latence, vyšší spotřeba | **Ano** (DB, výpočty) | Ne (zbytečně žere baterii) | `cpupower frequency-set -g performance` |
| `powersave` | Frekvence vždy na minimu – úspora, pomalé | Ne | Jen při napájení z baterie | `cpupower frequency-set -g powersave` |
| `schedutil` | Frekvence podle zátěže scheduleru – dynamické | Obvykle ano | **Ano** (nejlepší poměr) | `cpupower frequency-set -g schedutil` |
| `ondemand` | Skokové změny podle utilisation (starší) | Náhrada za schedutil | Náhrada za schedutil | `cpupower frequency-set -g ondemand` |

```bash
# Zjištění aktuálního governoru
cpupower frequency-info
# výstup: current policy: frequency should be within 800 MHz and 4.20 GHz.
#          The governor "schedutil" may decide which speed to use

# Nastavení na performance (pro server)
cpupower frequency-set -g performance

# Trvalé nastavení (systemd)
cat /etc/systemd/system/cpupower.service
# [Service]
# Type=oneshot
# ExecStart=/usr/bin/cpupower frequency-set -g performance
# [Install]
# WantedBy=multi-user.target
```

### nice / renice

Priorita procesu v scheduleru (rozsah -20 až +19, nižší = vyšší priorita).

| Hodnota | Kdy použít | Příklad |
|---------|------------|---------|
| **-20 až -10** | Kritické procesy (real-time audio, DB) | `nice -n -10 ./latency-critical-app` |
| **0** | Výchozí hodnota | Většina procesů |
| **+10 až +19** | Background úlohy (backup, logrotate) | `nice -n 19 ./backup.sh` |

```bash
# Spuštění s nízkou prioritou
nice -n 19 tar czf backup.tar.gz /data

# Změna priority běžícího procesu
renice -n -5 -p 1234

# Změna všech procesů uživatele
renice -n 10 -u www-data
```

### taskset (CPU affinity)

Uzamčení procesu na konkrétní jádra – klíčové pro NUMA systémy a izolaci zátěže.

```bash
# Spuštění na jádrech 0-3
taskset -c 0-3 ./aplikace

# Affinity běžícího procesu
taskset -cp 4-7 5678

# Zobrazení affinity
taskset -p 5678
# výstup: pid 5678's current affinity mask: f0
```

### Kernel Boot Parametry

Parametry předávané kernelu při bootu (GRUB). Upravují se v `/etc/default/grub` na řádce `GRUB_CMDLINE_LINUX`.

| Parametr | Popis | Server | Workstation |
|----------|-------|--------|-------------|
| `isolcpus=` | Vyčlení jádra ze scheduleru – běží na nich jen explicitně přiřazené procesy | **Ano** (Dedikace pro DPDK/RT) | Zřídka |
| `nohz_full=` | Vypne přerušení časovače na izolovaných jádrech | **Ano** (snižuje latenci) | Ne |
| `rcu_nocbs=` | Přesune RCU callbacky z izolovaných jader na housekeeping | **Ano** (nutný doplněk isolcpus) | Ne |
| `intel_pstate=disable` | Vypne intel_pstate driver, vrátí se k acpi-cpufreq | Někdy (kvůli compatibilitě) | Ne |

```bash
# Příklad GRUB_CMDLINE_LINUX pro real-time server
GRUB_CMDLINE_LINUX="isolcpus=2-7 nohz_full=2-7 rcu_nocbs=2-7"

# Aplikace změn
grub-mkconfig -o /boot/grub/grub.cfg

# Ověření po restartu
cat /proc/cmdline
```

### irqbalance

Služba, která distribuuje hardwarová přerušení (IRQ) napříč jádry.

```bash
# Status
systemctl status irqbalance

# Ruční přiřazení IRQ (např. NIC na jádro 2)
echo 4 > /proc/irq/78/smp_affinity

# Zobrazení IRQ
cat /proc/interrupts | head -20
```

---

## Paměť

Správa paměti má zásadní vliv na výkon – od cacheování souborů až po swapování.

### vm.swappiness

Řídí preferenci uvolňování paměti (0 = nevylučovat cache, 100 = agresivní swapování).

| Použití | Hodnota | Zdůvodnění |
|---------|---------|------------|
| **Server (DB, výpočty)** | 10 | Drží data v cache, swap jen když je kritický nedostatek |
| **Workstation** | 30-60 | Vyvážený přístup |
| **Embedded / low-RAM** | 60-100 | Rychle uvolňuje RAM pro aplikace |
| **Desktop s HDD** | 10 | Snižuje čekání na swap (HDD swap = pomalý) |

```bash
# Dočasné nastavení
sysctl -w vm.swappiness=10

# Trvalé (přidat do /etc/sysctl.conf nebo /etc/sysctl.d/99-performance.conf)
echo "vm.swappiness = 10" >> /etc/sysctl.d/99-performance.conf
```

### vfs_cache_pressure

Určuje, jak agresivně kernel uvolňuje cache pro VFS objekty (dentry, inode).

| Hodnota | Chování |
|---------|---------|
| **50** | Kernel udržuje cache déle – vhodné pro fileservery |
| **100** | Výchozí – vyvážené |
| **200** | Agresivnější uvolňování – vhodné, když cache žere příliš RAM |

```bash
sysctl -w vm.vfs_cache_pressure=50
```

### dirty_ratio a dirty_background_ratio

Řízení, kdy kernel zapisuje "špinavé" (modifikované) stránky na disk.

| Parametr | Popis | Server (write-heavy) | Workstation |
|----------|-------|----------------------|-------------|
| `vm.dirty_ratio` | % RAM, kdy proces začne blokovat při zápisu | 40 | 20 |
| `vm.dirty_background_ratio` | % RAM, kdy kernel začne zapisovat na pozadí | 10 | 5 |
| `vm.dirty_expire_centisecs` | Stáří stránky (v setinách s), kdy je forced writeback | 3000 | 3000 |
| `vm.dirty_writeback_centisecs` | Interval wake-up writeback vlákna | 500 | 500 |

```bash
# Server s velkým množstvím zápisů
sysctl -w vm.dirty_ratio=40
sysctl -w vm.dirty_background_ratio=10

# Workstation (nižší = méně dat v cache = nižší riziko ztráty)
sysctl -w vm.dirty_ratio=20
sysctl -w vm.dirty_background_ratio=5
```

### Transparent Hugepages (THP)

Slučování malých stránek (4 KB) do obřích (2 MB / 1 GB). Zlepšuje TLB hit ratio, ale může způsobovat problémy.

| Režim | Popis | Kdy použít |
|-------|-------|------------|
| `always` | Kernel se snaží vždy slévat stránky | Workstation, obecné použití |
| `madvise` | Slévá jen stránky označené `madvise(MADV_HUGEPAGE)` | **Servery** (bezpečnější) |
| `never` | THP vypnuto | DB (MongoDB, PostgreSQL), real-time aplikace |

```bash
# Zjištění stavu
cat /sys/kernel/mm/transparent_hugepage/enabled

# Dočasné nastavení
echo never > /sys/kernel/mm/transparent_hugepage/enabled

# Trvalé přes kernel boot parametry
# Přidat do GRUB_CMDLINE_LINUX:
transparent_hugepage=never
```

> **Upozornění:** Databáze (PostgreSQL, MongoDB, Cassandra) často doporučují THP vypnout. Obří stránky způsobují nečekané pauzy při alokaci.

### Swap, zram, zswap

| Technika | Popis | Server | Workstation |
|----------|-------|--------|-------------|
| **Swap (disk)** | Klasický oddíl/soubor na disku | Ano (pro crash dump) | Ano (fallback) |
| **zram** | Komprimovaný blokový device v RAM (swap do RAM) | Zřídka | **Ano** (notebooky, low-RAM) |
| **zswap** | Kompresní cache pro swap – komprimuje data před zápisem na disk | **Ano** (snižuje I/O) | Ano |

```bash
# Nastavení zram (1 GB komprimovaného swapu)
cat /etc/systemd/zram-generator.conf
# [zram0]
# zram-size = 1024
# compression-algorithm = zstd

# Manuální vytvoření zram
modprobe zram
echo 1G > /sys/block/zram0/disksize
mkswap /dev/zram0
swapon /dev/zram0

# Ověření
swapon --show
# výstup: NAME       TYPE      SIZE   USED PRIO
#          /dev/zram0 partition   1G   0B   100

# Zjištění kompresního poměru
cat /sys/block/zram0/mm_stat
```

---

## Disk

I/O subsystém je často bottleneck. Správný scheduler, mount volby a filesystem dokážou udělat řádové rozdíly.

### I/O Scheduler

Každé blokové zařízení má scheduler, který řadí požadavky na disk.

| Scheduler | Popis | SSD | HDD | NVMe |
|-----------|-------|-----|-----|------|
| **mq-deadline** | Deadline-based – garantuje max latenci pro READ | Dobrý | **Ano** | Dobrý |
| **kyber** | Latency-optimalizovaný – jednoduchý a rychlý | **Ano** | Ne | **Ano** |
| **bfq** | Fair-queuing – spravedlivé rozdělení I/O mezi procesy | Workstation | Workstation | Workstation |
| **none** | Žádné řazení – fast path přímo do NVMe driveru | - | - | **Nejlepší** |

```bash
# Zjištění aktuálního scheduleru
cat /sys/block/sda/queue/scheduler
# výstup: [mq-deadline] kyber bfq none

# Nastavení pro NVMe
echo none > /sys/block/nvme0n1/queue/scheduler

# Trvalé nastavení (udev pravidlo)
cat /etc/udev/rules.d/60-iosched.rules
# ACTION=="add|change", KERNEL=="nvme*", ATTR{queue/scheduler}="none"
# ACTION=="add|change", KERNEL=="sd*", ATTR{queue/scheduler}="bfq"
```

### ionice

Prioritizace I/O operací (třída a úroveň).

| Třída | Hodnota | Popis |
|-------|---------|-------|
| **Idle** | 3 | I/O jen když nikdo jiný nepotřebuje |
| **Best-effort** | 2 (0-7) | Priorita 0 = nejvyšší, 7 = nejnižší |
| **Real-time** | 1 (0-7) | Přednostní I/O (pozor na hladovění) |

```bash
# Backup s minimální I/O prioritou
ionice -c 3 tar czf backup.tar.gz /data

# Změna I/O priority běžícího procesu
ionice -c 2 -n 0 -p 5678
```

### Mount Volby (noatime / nodiratime)

Každý přístup k souboru standardně aktualizuje `atime` (čas přístupu) – to generuje zápis.

| Volba | Efekt | Doporučení |
|-------|-------|------------|
| `noatime` | Neaktualizuje atime vůbec | **Vždy** (výjimka: mailové schránky, Mutt) |
| `nodiratime` | Neaktualizuje atime pro adresáře | Součást noatime |
| `relatime` | Aktualizuje atime jen když je starší než mtime/ctime | Výchozí v moderních kernel |

```bash
# /etc/fstab
UUID=xxxx-xxxx  /data  ext4  defaults,noatime,nodiratime  0  2

# Ověření
mount | grep noatime
```

### Filesystem: ext4 vs xfs vs btrfs

| Vlastnost | ext4 | xfs | btrfs |
|-----------|------|-----|-------|
| **Max velikost souboru** | 16 TB | 8 EB | 16 EB |
| **Max velikost FS** | 1 EB | 8 EB | 16 EB |
| **Defragmentace** | e4defrag | xfs_fsr | online (automatická) |
| **Snapshot** | Ne (LVM) | Ne (LVM/XFS) | **Ano** (nativní) |
| **Komprese** | Ne | Ne | **Ano** (zstd/zlib) |
| **Kontrolní součet metadata** | Metadata | Metadata + data | Metadata + data |
| **Vhodné pro** | Obecné, boot | Velké soubory, servery | Kontejnery, snapshoty |

```bash
# Vytvoření XFS s optimálními parametry pro server
mkfs.xfs -f -m reflink=1 -d agcount=4 /dev/sdb1

# Vytvoření ext4
mkfs.ext4 -O ^has_journal -E stride=256,stripe_width=512 /dev/sdc1

# Btrfs s kompresí
mkfs.btrfs -m raid1 -d raid1 /dev/sda /dev/sdb
mount -o compress=zstd:3 /dev/sda /mnt
```

### fsck Periodicita

Pravidelná kontrola filesystemu. U moderních filesystemů (xfs, btrfs) méně kritická.

```bash
# Zjištění intervalu kontrol (ext4)
tune2fs -l /dev/sda1 | grep -i "check"
# výstup: Check interval: 15552000 (6 months)
#         Maximum mount count: -1  (vypnuto)

# Vypnutí periodických kontrol
tune2fs -c -1 -i 0 /dev/sda1

# Ruční kontrola (FS musí být odmountovaný)
fsck -f /dev/sda1
xfs_repair /dev/sdb1
btrfs check /dev/sdc1
```

### SSD Tuning (discard / fstrim)

SSD disky potřebují TRIM pro oznámení, které bloky jsou volné.

| Metoda | Popis | Doporučení |
|--------|-------|------------|
| **discard** (mount) | Průběžné posílání TRIM při každém smazání | Nedoporučuje se (latence) |
| **fstrim** (cron/systemd timer) | Dávkové posílání TRIM periodicky | **Ano** – 1× týdně |

```bash
# fstrim timer (ve většině distribucí default)
systemctl status fstrim.timer
# výstup: ● fstrim.timer - Discard unused blocks once a week
#          Trigger: Fri 2026-05-22 06:08:34 CEST; 2 days left

# Ruční spuštění
fstrim -av

# Ověření podpory TRIM
lsblk -D
# výstup: NAME    DISC-ALN DISC-GRAN DISC-MAX DISC-ZERO
#          sda           0      512B      2G         0
```

---

## Síť

Síťový stack kernelu lze vyladit pro propustnost (throughput) nebo nízkou latenci.

### Buffer Sizes: rmem_max / wmem_max

Velikost socket bufferů ovlivňuje, kolik dat může kernel držet v paměti před odesláním/přijetím.

| Parametr | Popis | Server | Workstation |
|----------|-------|--------|-------------|
| `net.core.rmem_max` | Max velikost receive bufferu | **16 MB** (16777216) | 2 MB (212992) |
| `net.core.wmem_max` | Max velikost send bufferu | **16 MB** | 2 MB |
| `net.core.rmem_default` | Výchozí receive buffer | 256 KB | 256 KB |
| `net.core.wmem_default` | Výchozí send buffer | 256 KB | 256 KB |
| `net.ipv4.tcp_rmem` | TCP receive: min / default / max | 4096 131072 16777216 | 4096 131072 6291456 |
| `net.ipv4.tcp_wmem` | TCP send: min / default / max | 4096 65536 16777216 | 4096 65536 6291456 |

```bash
# Server nastavení pro vysokou propustnost
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
sysctl -w net.ipv4.tcp_rmem="4096 131072 16777216"
sysctl -w net.ipv4.tcp_wmem="4096 65536 16777216"
```

### TCP Congestion Control

Algoritmus řízení přetížení TCP.

| Algoritmus | Popis | Kdy použít |
|------------|-------|------------|
| **cubic** | Výchozí v Linuxu – stabilní, dobrý pro obecné použití | Výchozí, univerzální |
| **bbr** | Model-based (Google) – lepší využití linky, nižší latence | **Servery, high-BW links** |
| **bbr3** | Vylepšený BBR (kernel 6.x+) – lepší fairness | Pokud je dostupný |
| **htcp** | High-speed TCP – starší alternativa k cubic | Zřídka |

```bash
# Zjištění dostupných algoritmů
sysctl net.ipv4.tcp_available_congestion_control
# výstup: reno cubic bbr

# Nastavení BBR
sysctl -w net.ipv4.tcp_congestion_control=bbr
# BBR vyžaduje i tcp_notification pro přesnější časování:
sysctl -w net.ipv4.tcp_notification=1  # Pokud kernel podporuje

# Ověření
sysctl net.ipv4.tcp_congestion_control
# výstup: net.ipv4.tcp_congestion_control = bbr
```

### tcp_slow_start_after_idle

Resetuje TCP slow-start po idle periodě. Vypnutí zrychluje nové spojení po pauze.

```bash
# Server s udržovanými spojeními
sysctl -w net.ipv4.tcp_slow_start_after_idle=0
```

### netdev_budget

Počet paketů zpracovaných v jednom cyklu přerušení (NAPI poll).

```bash
# Vyšší = více paketů na cyklus = vyšší propustnost, vyšší latence
sysctl -w net.core.netdev_budget=600

# Pro low-latency
sysctl -w net.core.netdev_budget=300
```

### txqueuelen

Délka fronty odesílání na síťovém rozhraní.

```bash
# Zvýšení pro vysokou propustnost
ip link set dev eth0 txqueuelen 10000

# Zjištění aktuální
ip link show eth0 | grep qlen
# výstup: qlen 10000
```

### Další síťové parametry

```bash
# Rychlé ACK (zapnout)
sysctl -w net.ipv4.tcp_sack=1
sysctl -w net.ipv4.tcp_fack=1

# Zvýšení počtu povolených spojení
sysctl -w net.core.somaxconn=65535
sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Rychlejší uvolnění TIME_WAIT (pro servery s mnoha krátkými spojeními)
sysctl -w net.ipv4.tcp_tw_reuse=1

# Keepalive
sysctl -w net.ipv4.tcp_keepalive_time=60
sysctl -w net.ipv4.tcp_keepalive_intvl=10
sysctl -w net.ipv4.tcp_keepalive_probes=6

# Minimalizace fragmentation
sysctl -w net.ipv4.ip_no_pmtu_disc=0  # 0 = zapnuto (doporučeno)
```

---

## Kernel Parametry

### sysctl.conf a sysctl.d

Trvalé nastavení kernel parametrů.

```bash
# Struktura
/etc/sysctl.d/
├── 10-network-security.conf
├── 10-kernel-tune.conf
└── 99-performance.conf

# Aplikace
sysctl --system                     # Všechny konfigurační soubory
sysctl -p /etc/sysctl.d/99-performance.conf  # Jeden konkrétní
```

**Ukázkový soubor `99-performance.conf`:**

```ini
# 99-performance.conf - Performance tuning for Linux server

# --- CPU / Memory ---
vm.swappiness = 10
vm.vfs_cache_pressure = 50
vm.dirty_ratio = 40
vm.dirty_background_ratio = 10

# --- Network ---
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 131072 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_congestion_control = bbr
net.core.netdev_budget = 600
net.core.somaxconn = 65535

# --- Filesystem ---
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288

# --- Security (mírné) ---
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
```

### /etc/security/limits.conf a ulimit

Omezení zdrojů na úrovni uživatele/skupiny.

| Parametr | Popis | Server (doporučeno) |
|----------|-------|---------------------|
| `nofile` | Max počet otevřených souborů | 1048576 |
| `nproc` | Max počet procesů | 262144 |
| `memlock` | Max zamčená paměť (pro ML/DPDK) | unlimited |
| `stack` | Max velikost zásobníku | 8192 (8 MB) |
| `core` | Max velikost core dump | 0 (vypnout) nebo unlimited |

```bash
# /etc/security/limits.conf
*                soft    nofile          1048576
*                hard    nofile          1048576
*                soft    nproc           262144
*                hard    nproc           262144
@dba            soft    memlock         unlimited
@dba            hard    memlock         unlimited

# Ověření aktuálních limitů
ulimit -a

# Změna v aktuální session
ulimit -n 1048576
```

### Kernel Modules (blacklist)

Zakázání nepotřebných modulů zrychluje boot a zvyšuje bezpečnost.

```bash
# Blacklist nepotřebných modulů
cat /etc/modprobe.d/blacklist.conf
# Nepotřebné storage drivery
blacklist floppy
blacklist pata_acpi
blacklist pata_old_ide

# Nepotřebné síťové protokoly
blacklist rds
blacklist tipc
blacklist dccp
blacklist sctp

# Audio (čistě výpočetní server)
blacklist snd_hda_intel
blacklist snd_hda_codec
blacklist snd

# Ověření načtených modulů
lsmod | head -20
```

---

## Server vs Workstation

Hlavní rozdíl: **Server = propustnost (throughput)**, **Workstation = interaktivita (latence)**.

| Oblast | Server | Workstation |
|--------|--------|-------------|
| **CPU Governor** | `performance` | `schedutil` nebo `ondemand` |
| **I/O Scheduler** | `mq-deadline` nebo `none` (NVMe) | `bfq` |
| **swappiness** | 10 | 30-60 |
| **dirty_ratio** | 40 | 20 |
| **THP** | `madvise` nebo `never` | `always` |
| **TCP buffers** | Velké (16 MB) | Střední (2-4 MB) |
| **tcp_congestion** | `bbr` | `cubic` |
| **irqbalance** | Ano (nebo ruční affinity) | Ano |
| **filesystem** | xfs nebo ext4 | ext4 nebo btrfs |
| **atime** | `noatime` | `relatime` (default) |
| **ulimit nofile** | 1048576 | 4096 (default) |
| **Power management** | Vypnuto (max výkon) | Zapnuto (úspora) |

---

## Praktické příklady

### Kompletní profil pro výpočetní server

```bash
#!/bin/bash
# apply-performance-profile.sh

echo "=== Aplikace performance profilu pro server ==="

# 1. CPU Governor
cpupower frequency-set -g performance

# 2. Kernel params dočasně
sysctl -w vm.swappiness=10
sysctl -w vm.vfs_cache_pressure=50
sysctl -w vm.dirty_ratio=40
sysctl -w vm.dirty_background_ratio=10
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
sysctl -w net.ipv4.tcp_congestion_control=bbr
sysctl -w net.core.netdev_budget=600

# 3. I/O Scheduler
for dev in /sys/block/sd*; do
    echo mq-deadline > "$dev/queue/scheduler"
    echo 1024 > "$dev/queue/nr_requests"
done

# 4. Transparent Hugepages
echo madvise > /sys/kernel/mm/transparent_hugepage/enabled

# 5. Zvýšení ulimit
ulimit -n 1048576

echo "=== Hotovo ==="
```

### tuned-adm

Nástroj pro automatické přepínání profilů (součást Project Tuned).

```bash
# Instalace
dnf install tuned tuned-utils   # RHEL/Fedora
apt install tuned               # Debian/Ubuntu

# Seznam profilů
tuned-adm list
# výstup: - balanced
#          - desktop
#          - latency-performance
#          - network-latency
#          - network-throughput
#          - powersave
#          - throughput-performance
#          - virtual-guest
#          - virtual-host

# Aplikace profilu pro maximální výkon
tuned-adm profile throughput-performance

# Pro nízkou latenci
tuned-adm profile latency-performance

# Vlastní profil
cat /etc/tuned/myprofile/tuned.conf
# [main]
# include=throughput-performance
#
# [cpu]
# governor=performance
# energy_perf_policy=performance
# min_perf_pct=100
#
# [vm]
# transparent_hugepages=madvise
#
# [sysctl]
# vm.swappiness=10
# net.core.rmem_max=16777216

# Aplikace
tuned-adm profile myprofile

# Ověření
tuned-adm active
# výstup: Current active profile: myprofile
```

### Benchmarking

#### stress (zátěžový test)

```bash
# Instalace
apt install stress

# Zatížení 4 CPU jader po dobu 60s
stress --cpu 4 --timeout 60

# Zatížení paměti (2 alokátory po 512 MB)
stress --vm 2 --vm-bytes 512M --timeout 30

# Kombinovaný test
stress --cpu 4 --io 2 --vm 2 --vm-bytes 1G --timeout 120
```

#### sysbench (systemový benchmark)

```bash
# Instalace
apt install sysbench

# CPU benchmark (výpočet prvočísel)
sysbench cpu run

# Paměťový benchmark
sysbench memory run

# I/O benchmark (příprava + test)
sysbench fileio --file-total-size=10G prepare
sysbench fileio --file-total-size=10G --file-test-mode=rndrw run
sysbench fileio --file-total-size=10G cleanup
```

#### Použití /proc a perf stat

```bash
# Rychlé měření výkonu příkazu
perf stat -r 5 curl https://example.com

# Sledování kontextových přepnutí (context switches)
watch -n 1 'grep ctxt /proc/stat'

# Sledování přerušení na jádro
watch -n 1 'cat /proc/interrupts | head -5'

# Zjištění bottlenecku pomocí vmstat
vmstat 1 10
# výstup: procs ----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
#          r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
#          2  0      0 1234567  12345 987654    0    0    12    34  567 1234 12  3 85  0  0
#          r = running queue, b = blocked, wa = iowait
```

---

## Shrnutí

Optimalizace Linux kernelu není univerzální recept. Každá úloha vyžaduje jiné nastavení:

1. **Změřte před a po** – bez metrik je tuning hádání
2. **Měňte jeden parametr najednou** – jinak nevíte, co zabralo
3. **Dokumentujte změny** – do komentářů v `/etc/sysctl.d/` a do notepadů projektu
4. **Testujte na stage** – nikdy ne na produkci naslepo
5. **Automatizujte** – použijte `tuned-adm`, Ansible, nebo deployment skripty

### Rychlý checklist

- [ ] CPU governor na `performance` / `schedutil`
- [ ] I/O scheduler podle typu disku
- [ ] `noatime` mount ve `/etc/fstab`
- [ ] `vm.swappiness` na 10 (server) / 60 (desktop)
- [ ] `tcp_congestion_control` na `bbr` (server)
- [ ] Velké TCP buffery na serveru
- [ ] Periodický `fstrim` pro SSD
- [ ] `transparent_hugepage` na `madvise` pro DB
- [ ] Zvýšené `ulimit` (nofile, nproc)
- [ ] Blacklist nepotřebných kernel modulů
- [ ] Profil v `tuned-adm` (nebo vlastní skript)

---

> **Navazující témata:** cgroups (systemd scope), BPF/BCC pro tracing, perf, eBPF, real-time kernel (PREEMPT_RT), DPDK pro síťové aplikace.

---

➡️ [Zpět na přehled](README.md)
