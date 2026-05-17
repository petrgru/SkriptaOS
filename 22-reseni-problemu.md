# Řešení problémů

> Troubleshooting

---

## Úvod

Troubleshooting je systematický proces identifikace, analýzy a odstranění příčin selhání systému. Nejde o nahodilé zkoušení příkazů, ale o strukturovaný postup.

### 5-kroková metodologie

1. **Definuj problém** — Co přesně nefunguje? Zapiš přesnou chybovou hlášku.
2. **Shromáždi informace** — Logy, výstupy příkazů, konfigurace, poslední změny.
3. **Identifikuj příčinu** — Analyzuj data, otestuj hypotézy.
4. **Oprav** — Aplikuj řešení (změna konfigurace, restart služby).
5. **Ověř** — Zkontroluj, že problém zmizel a že oprava nezpůsobila nové problémy.

### Základní principy

**Reprodukovatelnost.** Pokud problém nejde spolehlivě reprodukovat, nelze ho opravit. Zapisuj si postup, jak chybu vyvolat.

**"What changed?"** Vetsina problému má původ v nějaké změně — aktualizace, změna konfigurace, nový hardware. Používej git, `/var/log/dpkg.log`, `history` ke zjištění, co se změnilo.

**Rozděl a panuj.** Složité problémy rozděl na menší části. Pokud služba neběží a disk je plný, řeš pravděpodobnější příčinu nejdřív.

---

## 1. Diagnostické nástroje

| Nástroj | Kdy použít | Příklad |
|---------|-----------|---------|
| `dmesg` | Kernel problémy, hardware, OOM | `dmesg --level=err,warn` |
| `journalctl` | Systémové služby | `journalctl -u sshd -n 50 --no-pager` |
| `strace` | Sledování syscallů | `strace -p PID` |
| `lsof` | Otevřené soubory/porty | `lsof -i :80` |
| `tcpdump` | Síťová analýza | `tcpdump -i eth0 port 80` |
| `vmstat` | Výkon (CPU, RAM, I/O) | `vmstat 1 5` |

**dmesg** — Kernel ring buffer. Zobrazí zprávy o hardware, selhání disku, OOM Killer:

```bash
$ dmesg --level=err,warn
[    1.234567] ata1.00: failed command: READ FPDMA QUEUED
[12345.678901] Out of memory: Killed process 5678 (mysqld)
```

**journalctl** — Logy systemd služeb s pokročilým filtrováním:

```bash
$ journalctl -u nginx -n 50 --no-pager
$ journalctl --since "1 hour ago" -p err
$ journalctl -b -p err
```

**strace** — Sleduje systémová volání. Když proces padá, ukáže selhávající `open()` nebo `connect()`:

```bash
$ strace -p 1234
openat(AT_FDCWD, "/etc/nginx/nginx.conf", O_RDONLY) = 4
```

**lsof** — Který proces používá port nebo soubor. Důležité pro hledání smazaných souborů držených v paměti.

**tcpdump** — Packetová analýza na úrovni TCP/IP. Flag `[R.]` znamená RST (odmítnuté spojení).

**vmstat** — Rychlý přehled CPU, pameti, swapu. Sloupec `wa` nad 10 % = I/O bottleneck, `r` nad počtem jader = CPU zahlcení.

Dalsi zakladni nastroje: `top`/`htop` (kap. 6), `iostat -x 1` (vytížení disků), `ping`, `traceroute`, `nslookup`/`dig`.

---

## 2. Disk je plný

Jeden z nejčastějších problému na serverech.

```bash
# Identifikace
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        49G   46G  2.1G  96% /

# Největší adresáře
$ du -sh /* 2>/dev/null | sort -rh | head -10
12G     /usr
8.5G    /var
4.2G    /home

# Logy jsou nejčastější viník
$ du -sh /var/log/* | sort -rh | head
3.2G    /var/log/journal
1.8G    /var/log/nginx

# Velké soubory nad 100 MB
$ find / -xdev -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

Někdy `df` ukazuje plno, ale `du` nic nenajde. Proces může mít otevřený smazaný soubor:

```bash
$ sudo lsof | grep '(deleted)'
nginx 1234 www-data 4w REG 8,1 2147483648 /var/log/nginx/access.log (deleted)
# Řešení: restart procesu
$ sudo systemctl restart nginx
```

Interaktivní průzkum: `ncdu /`. Prevence: `logrotate`, monitoring zaplnění (kap. 21), `sudo apt autoremove --purge`.

```bash
# Manuální rotace logů
$ sudo logrotate -f /etc/logrotate.conf
```

---

## 3. Služba neběží

Postup krok za krokem při selhání služby.

```bash
# 1. Stav
$ systemctl status nginx
○ nginx.service - A high performance web server
     Active: failed (Result: exit-code) since Mon 2026-05-11 14:23:45
    Process: 1234 ExecStart=/usr/sbin/nginx (code=exited, status=1/FAILURE)
May 11 14:23:45 server nginx[1234]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)

# 2. Logy služby
$ journalctl -u nginx -n 50 --no-pager
$ journalctl -u nginx -p err

# 3. Kontrola konfigurace
$ systemctl cat nginx | grep ExecStart
ExecStart=/usr/sbin/nginx -g 'daemon on;'
$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

# 4. Závislosti
$ systemctl list-dependencies nginx
nginx.service
├─network.target
├─system.slice
└─sysinit.target

# 5. Port binding
$ ss -tulpn | grep :80
tcp LISTEN 0 128 0.0.0.0:80 0.0.0.0:* users:(("apache2",pid=5678))

# 6. Reset failed stavu (po opakovaném selhání)
$ sudo systemctl reset-failed nginx
```

Příklad: nginx nejde spustit, port 80 drží apache:

```bash
$ sudo systemctl stop apache2
$ sudo systemctl disable apache2
$ sudo systemctl start nginx
$ systemctl is-active nginx
active
```

Dalsi syntax checky: `sshd -t`, `apache2ctl configtest`, `named-checkconf`.

---

## 4. Port je obsazený

Chyba "Address already in use". Jeden port muze naslouchat jen jedna služba.

```bash
# Kdo drží port?
$ ss -tulpn | grep :80
tcp LISTEN 0 128 0.0.0.0:80 0.0.0.0:* users:(("apache2",pid=5678))

$ lsof -i :80
COMMAND   PID   USER   FD   TYPE DEVICE NODE NAME
apache2  5678   root    4u  IPv4  65432  TCP *:http (LISTEN)

$ fuser 80/tcp
80/tcp:              5678
```

Možnosti řešení:

```bash
# 1. Zastavit konfliktní službu
$ sudo systemctl stop apache2

# 2. Zmenit port v konfiguraci (napr. nginx na 8080)
$ sudo sed -i 's/listen 80;/listen 8080;/' /etc/nginx/sites-enabled/default
$ sudo systemctl restart nginx

# 3. Kill proces (poslední možnost)
$ sudo kill -9 5678
```

Po uvolnění portu ověř:

```bash
$ ss -tulpn | grep :80
tcp LISTEN 0 128 0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=1234))
```

---

## 5. Pomalý systém

Cílem je najít bottleneck — místo, kde systém nestíhá.

```bash
# Load average (8.15 na 4jadrovém stroji = pretížení)
$ uptime
 15:30:45 up 12 days, 4:23, 2 users, load average: 8.15, 6.30, 3.45

# top — živý náhled
$ top -bn1 | head -10
%Cpu(s): 45.2 us, 12.8 sy, 0.0 ni, 35.0 id, 6.5 wa, 0.3 hi, 0.2 si
MiB Mem : 7956.5 total, 456.2 free, 4234.1 used, 3266.2 buff/cache
MiB Swap: 2048.0 total, 1345.6 used, 702.4 free

  PID USER      %CPU  %MEM COMMAND
 3456 mysql     89.3   5.7 mysqld
```

Kritické hodnoty v `vmstat 1 5`:

| Sloupec | Kdy je problém | Význam |
|---------|---------------|--------|
| `r` | > počet jader | CPU bottleneck |
| `b` | > 1 | I/O bottleneck |
| `si`/`so` | > 0 | Málo RAM (swapuje) |
| `wa` | > 10 % | Pomalý disk |

```bash
$ vmstat 1 3
 r  b   swpd   free   si   so   wa
 8  2 104856 45678   34   56    7    # CPU + I/O bottleneck

# iostat — vytížení disku
$ iostat -x 1 3
Device   r/s    w/s  await  %util
sda     145.2  23.4  12.34  98.5    # %util > 80 = saturovaný disk

# free — paměť
$ free -h
               total  used  free  buff/cache  available
Mem:           7.8Gi  4.2Gi 456Mi   3.2Gi      3.0Gi
Swap:          2.0Gi  1.3Gi 702Mi
```

Pokud `available` klesá k nule nebo swap neni prázdný, systém má málo RAM. Když `kswapd0` žere CPU, kernel zoufale shání paměť. Pokud OOM Killer zabil proces, je nutné přidat RAM nebo omezit paměť náročným procesům.

---

## 6. Síťové problémy

Síť je komplexní — problém může být kdekoli na cestě.

```bash
# 1. Základní connectivity
$ ping -c 4 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=12.3 ms
4 packets transmitted, 4 received, 0% packet loss

# Pokud ping neprojde — traceroute
$ traceroute 8.8.8.8
 1  10.0.0.1  0.5 ms
 2  192.168.1.1  2.3 ms
 3  * * *      # další hop neodpovídá

# 2. Naslouchá služba?
$ ss -tulpn
tcp LISTEN 0 128 0.0.0.0:80   0.0.0.0:*   users:(("nginx",pid=1234))
tcp LISTEN 0 128 0.0.0.0:22   0.0.0.0:*   users:(("sshd",pid=1024))
tcp LISTEN 0 128 127.0.0.1:5432  *:*       users:(("postgres",pid=2048))

# 3. Lokální test (obejde firewall)
$ curl -v http://localhost:80
* Connected to localhost (127.0.0.1) port 80
> GET / HTTP/1.1
< HTTP/1.1 200 OK

# TCP spojení k externímu serveru
$ telnet example.com 80
Trying 93.184.216.34...
Connected to example.com.

# 4. Packet-level analýza (flag [R.] = RST, odmítnuto)
$ sudo tcpdump -i any port 80 -nn
12:34:56.789 IP 10.0.0.1.54321 > 10.0.0.2.80: Flags [S]
12:34:56.790 IP 10.0.0.2.80 > 10.0.0.1.54321: Flags [R.]

# 5. DNS
$ nslookup google.com
Server:  8.8.8.8
Address: 142.250.185.78

$ dig google.com
;; ANSWER SECTION:
google.com. 300 IN A 142.250.185.78
```

Pokud DNS selže, zkontroluj `/etc/resolv.conf`:

```bash
$ cat /etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1
```

**Firewall:**

```bash
# iptables
$ sudo iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
1    DROP  tcp  --  0.0.0.0/0  0.0.0.0/0  tcp dpt:80

# ufw
$ sudo ufw status
80/tcp                     DENY      0.0.0.0/0
$ sudo ufw allow 80/tcp
```

---

## 7. Problémy s oprávněním

"Permission denied" — druhá nejčastější chyba. Prícin může být několik.

```bash
# 1. Vlastník a práva
$ ls -la /var/log/nginx/access.log
-rw-r----- 1 www-data adm 12345 May 15 10:30 access.log

$ id
uid=1000(petrg) groups=1000(petrg),4(adm),27(sudo)
# Pokud uživatel není ve skupině www-data, nemůže číst log.

# 2. Cesta — chyba muže být na adresáři, ne na souboru
$ namei -l /var/log/nginx/access.log
drwxr-xr-x root  root  /
drwxr-xr-x root  root  var
drwxr-xr-x root  root  log
drwx--x--- nginx adm   nginx        # <-- mimo skupinu adm nemáš přístup
-rw-r----- www-data adm  access.log

# 3. Sudo oprávnění
$ sudo -l
User petrg may run: (root) NOPASSWD: /usr/bin/systemctl restart nginx

# 4. ACL
$ getfacl /var/log/nginx/access.log
user:petrg:r--        # explicitní přístup pro petrg

# 5. Capabilities
$ getcap /usr/bin/ping
/usr/bin/ping = cap_net_raw+ep

# 6. SELinux (Fedora/RHEL) — blokuje i při správných právech
$ ls -Z /var/www/html/index.html
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
$ sudo audit2why < /var/log/audit/audit.log 2>/dev/null
# (Podrobně v kapitole 23)

# 7. AppArmor (Ubuntu/Debian)
$ sudo aa-status
28 profiles are in enforce mode.
   /usr/sbin/nginx
$ sudo grep DENIED /var/log/syslog | tail -5
apparmor="DENIED" profile="/usr/sbin/nginx" name="/etc/nginx/ssl/private.key"
# (Podrobně v kapitole 15)
```

---

## 8. Obecný postup

Když narazíš na neznámý problém, použij tento univerzální checklist:

1. **Co se změnilo?** — `git log --oneline -10`, `/var/log/dpkg.log`, `history | tail -20`
2. **Zkontroluj logy** — `journalctl -p err -b`, `dmesg --level=err,warn`, `/var/log/syslog`
3. **Je problém reprodukovatelný?** — Pokud ne, zkus cron / scheduled tasks.
4. **Zkontroluj zdroje** — `df -h`, `free -h`, `uptime`, `systemctl --failed`
5. **Izoluj proměnné** — Minimal reproduce case, test na jiném stroji, vypni polovinu věcí a zjisti, kdy problém zmizí.
6. **Hledej řešení** — Copy-paste error message do Google / Stack Overflow / ChatGPT, zkontroluj known issues.
7. **Aplikuj opravu a over** — Jedna zmena najednou, dokumentuj, over že problém zmizel a nic se nerozbilo.

### Pravidlo 5 proč

Technika root cause analysis. Ptej se "proč", dokud nenajdeš skutečnou prícinu:

```
1. Proc není web dostupný? → Nginx nebeží.
2. Proc nginx nebeží? → Spadl kvuli out of memory.
3. Proc out of memory? → MySQL žere 6 GB RAM.
4. Proc MySQL žere 6 GB RAM? → Není omezená v my.cnf.
5. Proc není omezená? → Nikdo nenastavil innodb_buffer_pool_size.
```

Skutečná příčina: chybějící konfigurace limitu paměti. Restart nginx by problém vyřešil jen docasně.

### Důležité zásady

- **Jedna změna najednou** — jinak nepoznáš, co problém vyřešilo.
- **Dokumentuj** — až se problém vrátí za půl roku, budeš mít návod.
- **Nepanikař** — většina problémů je opravitelná. I kdybys smazal konfiguraci, balíček se dá přeinstalovat.

---

## Shrnutí

| Problém | Nástroj | Příkaz |
|---------|---------|--------|
| Plný disk | `df` / `du` / `lsof` | `df -h; du -sh /* 2>/dev/null \| sort -rh` |
| Smazané soubory držené procesy | `lsof` | `lsof \| grep '(deleted)'` |
| Služba neběží | `systemctl` / `journalctl` | `systemctl status; journalctl -u -n 50` |
| Syntaxe konfigurace | `nginx -t` / `sshd -t` | `nginx -t; sshd -t` |
| Port obsazený | `ss` / `lsof` / `fuser` | `ss -tulpn \| grep :PORT` |
| Pomalý systém (CPU) | `uptime` / `vmstat` | `uptime; vmstat 1 5` |
| Pomalý systém (I/O) | `iostat` / `dmesg` | `iostat -x 1; dmesg --level=err,warn` |
| Pomalý systém (RAM) | `free` / `dmesg` | `free -h; dmesg \| grep -i oom` |
| Síť — connectivity | `ping` / `traceroute` | `ping -c 4 IP; traceroute IP` |
| Síť — naslouchání | `ss` / `curl` / `telnet` | `ss -tulpn; curl -v http://localhost:PORT` |
| Síť — DNS | `nslookup` / `dig` | `nslookup domena; dig domena` |
| Síť — packety | `tcpdump` | `sudo tcpdump -i any port 80` |
| Síť — firewall | `iptables` / `ufw` | `sudo iptables -L -n; sudo ufw status` |
| Oprávnění — práva | `ls` / `id` / `namei` | `ls -la; id; namei -l /cesta` |
| Oprávnění — ACL/capabilities | `getfacl` / `getcap` | `getfacl /cesta; getcap /usr/bin/binary` |
| Oprávnění — SELinux | `ls -Z` / `audit2why` | `ls -Z; audit2why < /var/log/audit/audit.log` |
| Oprávnění — AppArmor | `aa-status` / `syslog` | `sudo aa-status; grep DENIED /var/log/syslog` |
| Obecný postup | checklist / 5x proč | Viz sekce 8 |

> **POZNÁMKA:** Tato kapitola se zaměřuje na obecný troubleshooting. Speciální případy: [SELinux troubleshooting](23-selinux.md), [AppArmor troubleshooting](15-apparmor.md), [Netplan troubleshooting](16-netplan.md). Základní příkazy `ps`, `top`, `systemctl start/stop` najdete v [kapitole 6 – Správa procesů](06-sprava-procesu.md). Monitorování výkonu v [kapitole 21 – Monitoring systému](21-monitoring-systemu.md).

---

➡️ [Zpět na přehled](README.md)
