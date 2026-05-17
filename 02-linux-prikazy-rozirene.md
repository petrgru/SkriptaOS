# Rozšířené Linux příkazy

## Úvod

Tato sekce pokrývá rozšířené příkazy pro správu souborů, procesů, paměti a sítě. Každý příkaz je popsán s příkladem a typickým výstupem. Příklady jsou v angličtině, popisy v češtině.

---

## Správa souborů

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `find` | Vyhledávání souborů podle jména, typu, velikosti, data | `find /home -name "*.log" -type f -size +1M` | `/home/user/app.log`<br>`/home/user/debug.log` |
| `grep` | Hledání textu v souborech pomocí regulárních příkazů | `grep -rn "error" /var/log/syslog` | `1024:error: SSL handshake failed` |
| `sort` | Seřazení řádků v souboru nebo ze vstupu | `sort -k2 -t',' data.csv` | `Alice,25,Prague`<br>`Bob,30,Berlin`<br>`Charlie,22,London` |
| `uniq` | Odstranění nebo počítání sousedících duplicitních řádků | `sort names.txt | uniq -c` | `3 Alice`<br>`2 Bob`<br>`1 Charlie` |
| `cut` | Vybraní sloupců nebo znaků z řádků | `cut -d',' -f1,3 data.csv` | `Alice,Prague`<br>`Bob,Berlin`<br>`Charlie,London` |
| `paste` | Spojení řádků ze dvou souborů vedle sebe | `paste -d',' col1.txt col2.txt` | `Alice,25`<br>`Bob,30` |
| `join` | Spojení dvou souborů podle společného sloupce | `join -t',' file1.csv file2.csv` | `Alice,25,Developer`<br>`Bob,30,Designer` |
| `xargs` | Předání výstupu jednoho příkazu jako argumenty dalšímu | `find . -name "*.tmp" | xargs rm -v` | `removed './app.tmp'`<br>`removed './cache.tmp'` |
| `wc` | Počítání řádků, slov, znaků | `wc -l -w -c report.txt` | `42 156 893 report.txt` |
| `head` / `tail` | Zobrazení začátku nebo konce souboru | `head -n 5 file.txt` | (prvních 5 řádků) |

---

## Správa procesů

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `ps` | Seznam běžících procesů | `ps aux` | `USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND`<br>`root 1 0.0 0.1 18432 4096 ? Ss Jan01 0:05 /sbin/init` |
| `ps` | Filtrování procesů podle jména | `ps -ef | grep nginx` | `www-data 1234 1 0 14:32 ? 00:00:01 nginx: worker process` |
| `top` | Interaktivní pohled na procesy a výkon | `top` | `Tasks: 142 total, 1 running, 141 sleeping`<br>`%Cpu: 12.3 us, 2.1 sy, 0.0 ni, 85.6 id` |
| `htop` | Rozšířený interaktivní pohled na procesy | `htop` | `PID USER PRI NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND`<br>`1234 root 20 0 2.1g 150M 45M S 5.0 3.2 12:34.56 firefox` |
| `kill` | Odeslání signálu procesu podle PID | `kill -9 1234` | (proces 1234 ukončen) |
| `killall` | Ukonečení procesů podle jména | `killall chrome` | (všechny procesy chrome ukončeny) |
| `nice` | Spuštění procesu s prioritou | `nice -n 5 ./long_task.sh` | (proces běží s prioritou 5) |
| `renice` | Změna priority běžícího procesu | `renice -n 3 -p 1234` | `1234 (process ID) old priority 10, new priority 3` |
| `pgrep` | Nalezení PID podle jména procesu | `pgrep -l sshd` | `892 sshd` |
| `pidof` | Zobrazení PID procesu podle jména | `pidof firefox` | `4521 4522 4523` |

---

## Správa paměti

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `free` | Zobrazení volné a zabrané paměti | `free -h` | `total used free shared buff/cache available`<br>`Mem: 15Gi 8.2Gi 3.1Gi 512Mi 4.2Gi 6.5Gi`<br>`Swap: 4.0Gi 1.2Gi 2.8Gi` |
| `vmstat` | Statistiky virtuální paměti a procesů | `vmstat 2 5` | `procs r b swpd free buff cache si so`<br>`0 2 0 12288 4096 8192 0 0` |
| `iostat` | Statistiky vstupu/výstupu disku a CPU | `iostat -m 2 3` | `avg-cpu: %user %nice %system %idle`<br>`0.50 0.10 1.20 98.20` |
| `sar` | Historické statistiky výkonu | `sar -r -f /var/log/sa/sar01` | `10:00:00 kbmemfree kbavail kbmemused %memused`<br>`10:00:00 823456 4117280 12582912 93.75` |
| `pmap` | Mapa paměti procesu | `pmap -x 1234` | `Address Kbytes RSS Dirty Mode Mapping`<br>`0000000000400000 1024 512 512 r-xp libpthread.so` |

---

## Správa sítě

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `ifconfig` | Konfigurace síťových rozhraní | `ifconfig` | `eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500`<br>`inet 192.168.1.10 netmask 255.255.255.0` |
| `netstat` | Statistiky síťových připojení | `netstat -tulnp` | `Proto Recv-Q Send-Q Local Address Foreign Address State PID/Program`<br>`tcp 0 0 0.0.0.0:80 0.0.0.0:* LISTEN 1234/nginx` |
| `ss` | Rychlejší alternativa k netstat | `ss -tulnp` | `Netid State Recv-Q Send-Q Local Address:Port Peer Address:Port`<br>`tcp LISTEN 0 128 0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=1234))` |
| `route` | Zobrazení a úprava tabulky tras | `route -n` | `Destination Gateway Genmask Flags Interface`<br>`0.0.0.0 192.168.1.1 0.0.0.0 UG eth0` |
| `ip` | Moderní nástroj pro správu sítě | `ip addr show` | `2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500`<br>`inet 192.168.1.10/24 scope global eth0` |
| `ping` | Testovat spojitost se serverem | `ping -c 4 google.com` | `64 bytes from google.com: icmp_seq=1 ttl=118 time=15.2ms` |
| `traceroute` | Stopka přes síťové uzly | `traceroute google.com` | `1 192.168.1.1 (192.168.1.1) 2.1 ms` |
| `curl` | Přídání dat přes URL | `curl -I https://example.com` | `HTTP/2 200`<br>`content-type: text/html` |
| `wget` | Stahování souborů z webu | `wget https://example.com/file.tar.gz` | `file.tar.gz [1.2M] 100% 2.3 MB/s` |

---

## Shrnutí

- **Správa souborů**: `find` a `grep` jsou nejpoužívanější pro vyhledávání. `sort`, `uniq`, `cut`, `paste`, `join` tvoří základ pro zpracování dat v pipeline.
- **Správa procesů**: `ps` pro rychlý přehled, `top`/`htop` pro interaktivní pohled, `kill` pro ukončení, `nice`/`renice` pro priority.
- **Správa paměti**: `free` pro rychlý přehled paměti, `vmstat` a `iostat` pro hlubší analýzu výkonu.
- **Správa sítě**: `ip` je moderní nástroj, `ss` je rychlejší než `netstat`, `ping` a `traceroute` pro diagnostiku spojitosti.

Každý příkaz má mnoho možností. Používejte `man <příkaz` pro podrobný přehled.

---

➡️ [Zpět na přehled](README.md)
