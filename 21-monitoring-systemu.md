# Monitoring systemu

> System Monitoring

---

## Úvod

**Monitoring systemu** je klicova soucast spravy kazdeho Linuxoveho serveru. Bez sledovani vykonu a vytizeni zdroju nemuzete odhalit uzka hrdla, predpovedet selhani ani optimalizovat vykon.

Linux poskytuje dve virtualni filesystemy, ktere jsou zakladnim zdrojem informaci o systemu:

- **`/proc`** -- procesovy virtualni filesystem. Obsahuje soubory jako `/proc/cpuinfo`, `/proc/meminfo`, `/proc/loadavg`, `/proc/stat`, `/proc/diskstats` a `/proc/net/dev`.
- **`/sys`** -- sysfs, strukturovany pohled na jadro a hardware.

Vetsina CLI monitorovacich nastroju cte prave z `/proc` a `/sys`. Napriklad `top` cte `/proc/stat` a `/proc/PID/stat`, `free` cte `/proc/meminfo`.

Tato kapitola se venuje **vylucne CLI nastrojum**. Zadne Grafana, zadny Prometheus -- pouze terminal a prikazy, ktere mate k dispozici na kazdem Linuxovem serveru.

---

## 1. Procesy a CPU

Sledovani vytizeni CPU je prvni krok pri diagnostice pomaleho systemu. Je dulezite rozlisit, zda je system omezen vykonem CPU (CPU-bound) nebo ceka na I/O (I/O-bound).

### top a htop

Zakladni `top` znate z kapitoly 6. V monitorovacim kontextu vyuzijete pokrocile interaktivni funkce:

```bash
# Spusteni s razenim podle CPU
$ top -o %CPU
# Interaktivne: 1 = jadra, P = serazeni CPU, M = MEM, k = signal, d = interval
```

Vychozi interval 3s zkratte pro sledovani spicku na 0.5-1s. `htop` nabizi barevne rozliseni a stromovy pohled:

```bash
$ sudo apt install htop
$ htop -t                 # stromovy pohled
```

V `htop` je CPU zelena (user), cervena (kernel), modra (nizka priorita), zluta (IRQ).

### ps s vlastnim formatem

Standardni `ps aux` je prilis obecny. Pro monitoring pouzijte vlastni format a razeni:

```bash
$ ps -eo pid,user,%cpu,%mem,rsz,cmd --sort=-%cpu | head -15
$ ps -U petrg -o pid,%cpu,%mem,cmd --sort=-%cpu
```

Kombinace s `watch` umoznuje periodicke obnovovani:

```bash
$ watch -n 2 'ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -20'
```

### mpstat (per-jadro)

Nastroj `mpstat` z baliku `sysstat` ukazuje vytizeni kazdeho jadra:

```bash
$ sudo apt install sysstat
$ mpstat -P ALL 1 3
```

Sledujte sloupec `%iowait` -- pokud je nad 10 %, system ceka na disk. Sloupec `%steal` je dulezity u VPS -- vysoka hodnota znamena, ze hypervisor odebira CPU.

### /proc/loadavg a /proc/stat

Primy pohled do /proc bez rezie nastroju:

```bash
# Load average (1, 5, 15 minut)
$ cat /proc/loadavg
2.45 1.80 1.20 3/456 12345

# Cas jadra (user, system, idle, iowait...)
$ head -1 /proc/stat
cpu  245678 12345 198765 12345678 9876 5432 2109 0 0 0
```

Interpretace load average: na 4-jadrovem CPU je load 3.5 v pohode, na 2-jadrovem uz pretizeni.

---

## 2. Pamet

Sledovani pameti je dulezite pro detekci memory leku a neprimerene spotreby.

### free -h

Nejzakladnejsi nastroj, ktery je treba spravne cist:

```bash
$ free -h
               total        used        free      shared  buff/cache   available
Mem:            15Gi       6,2Gi       2,1Gi       0,5Gi       7,0Gi       8,3Gi
Swap:           2,0Gi       0,3Gi       1,7Gi
```

Nejcennejsi sloupec je **available** -- odhad pameti skutecne dostupne pro nove procesy (vcetne uvolnitelne cache). Nenechte se zmast nizkym `free`. Cilem je mit `buff/cache` vysoke -- system efektivne vyuziva RAM pro I/O cache.

### vmstat 1

`vmstat` poskytuje komplexni pohled na pamet, swap a I/O:

```bash
$ vmstat 1 5
procs ---------memory---------- ---swap-- -----io----
 r  b   swpd   free   buff  cache   si   so    bi    bo
 3  1   320M  2,1G   500M  6,5G    0    0    50    45
 2  0   320M  2,1G   500M  6,5G    0    0     0     8
```

Pokud jsou `si` (swap in) a `so` (swap out) trvale nenulove, system **thrashes** -- drasticky zpomaluje.

### /proc/meminfo

Nejpodrobnejsi zdroj informaci:

```bash
$ grep -E "MemTotal|MemFree|MemAvailable|Cached|Dirty|Shmem" /proc/meminfo
MemTotal:       16384832 kB
MemFree:         2156740 kB
MemAvailable:    8712340 kB
Cached:          6534210 kB
Dirty:              5670 kB
Shmem:            512340 kB
```

Klicove polozky: `Dirty` rostouc = disk nestiha zapisovat. `Shmem` = sdilena pamet.

### smem (PSS/RSS/USS)

Standardni `ps` ukazuje RSS, ktery u sdilenych knihoven pocita kazdou plne. Realistickejsi je PSS (Proportional Set Size):

```bash
$ sudo apt install smem
$ smem -t -p                                           # souhrn
$ smem -t -k -c "pid user pss rss command" | sort -k3 -rn | head -20
```

### Detekce memory leku

Sledujte RES v case. Pokud trvale roste i bez zateze, jedna se o memory leak:

```bash
$ watch -n 2 'ps -p 1234 -o pid,%mem,rss,cmd --no-headers'
$ watch -n 1 'grep VmRSS /proc/1234/status'
```

---

## 3. Disk I/O

Disk je obvykle nejpomalejsi komponenta systemu.

### iostat (sysstat)

```bash
$ iostat -x 1 3
Device     r/s    w/s    rkB/s    wkB/s  await  %util
sda      45,2  120,5  1234,5  5678,9   3,45   12,5
sdb     120,0  890,0 12345,0 45678,0   8,90   90,2
```

Klicove metriky: `%util` nad 80 % = problem (u NVMe sledujte `await`). `await` nad 10 ms (SSD) nebo 50 ms (HDD) = pretizeni.

### iotop (per-process I/O)

```bash
$ sudo iotop --only
$ sudo iotop -b -n 2
```

### df -h a du -sh

```bash
# Zaplneni disku
$ df -h

# Nejvetsi adresare
$ du -sh --exclude=/proc --exclude=/sys /* 2>/dev/null | sort -rh | head -10

# Velke soubory (nad 100 MB)
$ find / -xdev -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -20
```

### lsof +L1

Bezny problem: `df` ukazuje plny disk, ale `du` nenasel pricinu. Proces drzi otevreny smazany soubor:

```bash
$ sudo lsof +L1
apache2 1234 www-data 5w REG 8,1 2147483648 0 12345 /var/log/access.log (deleted)
```

Reseni: restart procesu.

---

## 4. Sit

### ss (moderni nahrada netstat)

```bash
$ ss -tulpn
tcp   LISTEN 0 128  0.0.0.0:22   0.0.0.0:*  users:(("sshd",pid=1024))
tcp   LISTEN 0 128  0.0.0.0:443  0.0.0.0:*  users:(("nginx",pid=2048))

$ ss -tunp         # aktivni spojeni
$ ss -s            # souhrn
```

Sledujte `Recv-Q` -- pokud se hromadi, aplikace nestiha.

### Statistiky rozhrani

```bash
$ netstat -i
Iface  MTU    RX-OK RX-ERR RX-DRP  TX-OK TX-ERR TX-DRP
eth0  1500 12345678  0      1     9876543  0      0

$ ip -s link
```

Rostouci `RX-ERR` = problem s kabelazi nebo prepinacem.

### Sledovani bandwidth

```bash
# Per-process (vyzaduje root)
$ sudo nethogs eth0

# Per-spojeni
$ sudo iftop -n -P

# Interaktivni ncurses monitor
$ sudo iptraf-ng
```

### /proc/net/dev

```bash
$ cat /proc/net/dev
eth0: 1234567890 12345678 0 0 0 0 0 0 987654321 9876543 0 0 0 0 0 0
```

---

## 5. Systemova zatez

### uptime a load average

```bash
$ uptime
14:23:45 up 12 days, 3:45, 3 users, load average: 1.25, 1.40, 1.10
```

Load pod poctem jader = rezerva, load nad poctem jader = procesy cekaji, load 2x jadera = vazne pretizeni.

### dmesg

```bash
$ dmesg --level=err,warn
$ sudo dmesg --level=err --follow    # real-time
```

Sledujte: `softreset failed` (disk), `Out of memory` (OOM Killer), `EXT4-fs error`.

### sar

```bash
$ sar -u 1 3           # CPU
14:23:45  CPU  %user  %system  %iowait  %steal  %idle
14:23:46  all  15,20    8,30    2,50    0,00   74,00
$ sar -r 1 3           # memory
$ sar -b 1 3           # I/O
$ sar -n DEV 1 5       # site
```

### tload (ASCII graf loadu)

```bash
$ tload
```

---

## 6. Dlouhode sledovani

Jednorazove mereni nestaci. Sysstat sbira data v pravidelnem intervalu pomoci cronu.

### Konfigurace

```bash
$ cat /etc/cron.d/sysstat
*/10 * * * * root /usr/lib/sysstat/sadc -F 600 /var/log/sysstat/sa$(date +%d)

$ ls /var/log/sysstat/
sa17  sa16  sa15
```

Soubory `saDD` jsou binarni, cistelne jen pres `sar -f`.

### Historicka data

```bash
$ sar -u                                    # dnes
$ sar -u -f /var/log/sysstat/sa$(date +%d -d yesterday)   # vcera
$ sar -u -s 08:00:00 -e 18:00:00            # casove okno
$ sar -r -f /var/log/sysstat/sa$(date +%d -d yesterday)   # pamet vcera
```

### sadf (export)

Pro zpracovani v externich nastrojich:

```bash
$ sadf -d -- -u /var/log/sysstat/sa17     # CSV
$ sadf -j -- -r /var/log/sysstat/sa17     # JSON
```

### pidstat

```bash
$ pidstat -p ALL 2 5        # CPU per proces
$ pidstat -r -p 1234 2 5    # memory
$ pidstat -C nginx 2 5      # konkretni prikaz
```

Sledujte `majflt/s` -- vysoka = proces cte ze swapu.

---

## 7. Uzivatele a prihlaseni

### w

```bash
$ w
14:23:45 up 12 days, 3:45, 3 users, load average: 1.25, 1.40, 1.10
USER   TTY   FROM           LOGIN@  IDLE  WHAT
petrg  pts/0 192.168.1.5   09:15   5:00  vim monitoring.md
root   pts/1 10.0.0.1      10:30   1:20m htop
```

Sledujte `IDLE` -- dlouhy idle + podivny proces = varovani.

### last a lastb

```bash
$ last -20                           # historie prihlaseni
petrg  pts/0  192.168.1.5   Sun May 17 09:15   still logged in
root   pts/1  10.0.0.1      Sun May 17 10:30   still logged in

$ sudo lastb                         # neuspesne pokusy
BAD    ssh:notty  192.168.1.100  Sun May 17 14:20 - 14:20
```

Tisice neuspesnych pokusu = brute force utok.

### lastlog a users

```bash
$ lastlog                             # kdy se kazdy naposledy prihlasil
root     pts/1  10.0.0.1   Sun May 17 10:30:00 +0000 2026
www-data ** Never logged in **

$ users                               # prave prihlaseni
petrg root www-data
$ who -a
```

Pokud se "Never logged in" uzivatel objevi v `lastlog`, mohlo dojit k naruseni.

---

## 8. Monitorovaci skripty

### Jednoradkove prikazy

```bash
# Top 5 procesu podle CPU
$ watch -n 1 'ps aux --sort=-%cpu | head -6'

# Top 5 podle pameti
$ watch -n 2 'ps aux --sort=-%mem | head -6'

# Nejcastejsi prikazy (pocet instanci)
$ ps aux | awk '{print $11}' | sort | uniq -c | sort -rn | head -10
```

### Kontrola disku

```bash
# Zaplneni nad 80 %
$ df -h | awk 'NR>1 {if ($5+0 > 80) print $6 " je z " $5 " plny"}'

# S emailem
$ df -h | awk 'NR>1 {if ($5+0 > 80) printf "%s je %s plny\n", $6, $5}' | \
  mail -s "ALERT: Misto na disku" admin@example.com
```

### Kontrola pameti

```bash
$ free -m | awk 'NR==2 {printf "Pamet: %.1f%% vyuzita (%s MB z %s MB)\n", $3*100/$2, $3, $2}'
Pamet: 40.5% vyuzita (6324 MB z 15600 MB)

# Alert nad 90 %
$ free -m | awk 'NR==2 {if ($3*100/$2 > 90) print "ALERT: Vysoke vyuziti"}'
```

### Sledovani logu

```bash
# Chyby v systemovem logu v realnem case
$ tail -f /var/log/syslog | grep -i error

# Pokusy o SSH
$ tail -f /var/log/auth.log | grep "Failed password"
```

### Cron monitoring

```bash
# /etc/cron.d/system-monitor
*/30 * * * * root df -h | awk 'NR>1 {if ($5+0 > 85) print}' | mail -s "Disk Alert" admin@example.com
0 8 * * *   root sar -u -f /var/log/sysstat/sa$(date +%d -d yesterday) | mail -s "CPU Report" admin@example.com
```

---

## Shrnuti

| Kategorie | Nastroj | Co meri | Priklad |
|-----------|---------|---------|---------|
| CPU | `top -o %CPU` | Vytizeni CPU per proces | `top -o %CPU` |
| CPU | `htop -t` | Stromovy pohled, barevne | `htop -t` |
| CPU | `ps --sort=-%cpu` | Seznam procesu razeny CPU | `ps aux --sort=-%cpu | head` |
| CPU | `mpstat -P ALL 1` | Vytizeni per jadro | `mpstat -P ALL 1 3` |
| CPU | `/proc/loadavg` | Load average 1/5/15 min | `cat /proc/loadavg` |
| Pamet | `free -h` | Celkove vyuziti RAM + swap | `free -h` |
| Pamet | `vmstat 1` | Pamet, swap, I/O v case | `vmstat 1 5` |
| Pamet | `smem -t -p` | PSS/RSS/USS per proces | `smem -t -k -c "pid user pss rss"` |
| Pamet | `/proc/meminfo` | Detailni rozpad pameti | `grep MemAvailable /proc/meminfo` |
| Disk | `iostat -x 1` | Rozsirene I/O statistiky | `iostat -x 1 3` |
| Disk | `iotop` | I/O per proces | `sudo iotop --only` |
| Disk | `df -h` | Zaplneni disku | `df -h` |
| Disk | `lsof +L1` | Smazane, drzene soubory | `sudo lsof +L1` |
| Sit | `ss -tulpn` | Naslouchajici sockety | `ss -tulpn` |
| Sit | `ip -s link` | Statistiky rozhrani | `ip -s link` |
| Sit | `nethogs` | Bandwidth per proces | `sudo nethogs eth0` |
| Sit | `iftop` | Bandwidth per spojeni | `sudo iftop -n -P` |
| Zatez | `uptime` | Load average | `uptime` |
| Zatez | `dmesg --level=err` | Kernelove chyby | `dmesg --level=err --follow` |
| Zatez | `sar -u 1 5` | Komplexni CPU report | `sar -u 1 5` |
| Historie | `sar -f saDD` | Historicka data sysstat | `sar -u -f /var/log/sysstat/sa17` |
| Historie | `pidstat -C nginx` | Per-process v case | `pidstat -C nginx 2 5` |
| Uzivatele | `w` | Kdo je online, load, idle | `w` |
| Uzivatele | `last` | Historie prihlaseni | `last -20` |
| Uzivatele | `lastb` | Neuspela prihlaseni | `sudo lastb` |
| Skripty | `watch -n 1` | Periodicke spousteni | `watch -n 1 'ps aux | head'` |
| Skripty | `df -h | awk` | Vlastni kontrola disku | `df -h | awk '$5+0 > 80'` |
| Skripty | `tail -f | grep` | Sledovani logu | `tail -f /var/log/syslog | grep -i error` |

> **POZNAMKA:** Pro optimalizaci vykonu na zaklade zjistenych metrik viz [kapitolu 9 -- Optimalizace vykonu](09-optimalizace-vykony.md).

---

➡️ [Zpet na prehled](README.md)
