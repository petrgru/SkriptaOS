# SkriptaOS - Operační systémy

Výuková dokumentace k předmětu **Operační systémy**. Přehled základních příkazů, skriptování, správy procesů, bezpečnosti a optimalizace výkonu v Linuxu i Windows.

Tento kurz je určen pro studenty informatiky a programátory, kteří chtějí pochopit, jak fungují operační systémy z praxe. Každá sekce kombinuje teorii s praktickými příklady.

---

## Obsah

| # | Sekce | Popis | Cvičení |
|---|-------|-------|---------|
| 00 | [Základy OS](00-zaklady-os.md) | Teoretický základ: kernel, syscalls, scheduling | [Cvičení](pracovni-listy/00-zaklady-os/) |
| 01 | [Základní Linux příkazy](01-linux-prikazy-zakladni.md) | Navigace, soubory, výstupní proudy, práva | [Cvičení](pracovni-listy/01-linux-prikazy-zakladni/) |
| 02 | [Rozšířené Linux příkazy](02-linux-prikazy-rozirene.md) | Pipe, grep, sed, awk, find, xargs | [Cvičení](pracovni-listy/02-linux-prikazy-rozirene/) |
| 03 | [Windows příkazy](03-windows-prikazy.md) | CMD a PowerShell příkazy pro Windows | [Cvičení](pracovni-listy/03-windows-prikazy/) |
| 04 | [Bash skriptování](04-bash-skriptovani.md) | Proměnné, cykly, funkce, argumenty | [Cvičení](pracovni-listy/04-bash-skriptovani/) |
| 05 | [PowerShell skriptování](05-powershell-skriptovani.md) | Objekty, pipeline, moduly, cmdlety | [Cvičení](pracovni-listy/05-powershell-skriptovani/) |
| 06 | [Správa procesů](06-sprava-procesu.md) | Procesy, vlákna, PID, priority, signály | [Cvičení](pracovni-listy/06-sprava-procesu/) |
| 07 | [Bezpečná architektura](07-bezpecna-architektura.md) | Práva, uživatele, skupiny, bezpečnost | [Cvičení](pracovni-listy/07-bezpecna-architektura/) |
| 08 | [Správa logů a prostředí](08-ovladani-soukromosti.md) | Souborové systémy, environment, logy | [Cvičení](pracovni-listy/08-ovladani-soukromosti/) |
| 09 | [Optimalizace výkonu](09-optimalizace-vykony.md) | CPU, paměť, disk, síť, profilování | [Cvičení](pracovni-listy/09-optimalizace-vykony/) |
| 10 | [Aktualizace systému](10-aktualizace-systemu.md) | Balíčky, jádro, patche, rollback | [Cvičení](pracovni-listy/10-aktualizace-systemu/) |
| 11 | [Správa uživatelů a skupin](11-sprava-uzivatelu.md) | Uživatelé, skupiny, práva, sudo, sudoers | [Cvičení](pracovni-listy/11-sprava-uzivatelu/) |
| 12 | [Práva souborů a adresářů](12-prava-souboru.md) | SUID, SGID, sticky bit, ACL, capabilities, umask | [Cvičení](pracovni-listy/12-prava-souboru/) |
| 13 | [Správa souborového systému](13-sprava-filesystemu.md) | Disky, oddíly, mkfs, mount, fsck, LVM, swap | [Cvičení](pracovni-listy/13-sprava-filesystemu/) |
| 14 | [Zálohování a obnova](14-zalohovani.md) | tar, gzip, rsync, dd, 3-2-1 strategie | [Cvičení](pracovni-listy/14-zalohovani/) |
| 15 | [AppArmor](15-apparmor.md) | AppArmor – Mandatory Access Control | [Cvičení](pracovni-listy/15-apparmor/) |
| 16 | [Netplan](16-netplan.md) | Netplan – konfigurace sítě | [Cvičení](pracovni-listy/16-netplan/) |
| 17 | [RAID a Redundance disků](17-raid-redundance-disku.md) | RAID, ZFS, BTRFS, LVM, SnapRAID | [Cvičení](pracovni-listy/17-raid-redundance-disku/) |
| 18 | [Proxmox RAID1 Mirror](18-proxmox-raid.md) | RAID1 na Proxmox VE – praktický návod | [Cvičení](pracovni-listy/18-proxmox-raid/) |
| 19 | [Práce s SSH](19-prace-s-ssh.md) | SSH klíče, sshd_config, tunelování, scp/rsync | [Cvičení](pracovni-listy/19-prace-s-ssh/) |
| 20 | [Systemd](20-systemd.md) | Správa služeb, unit soubory, journalctl, timery | [Cvičení](pracovni-listy/20-systemd/) |
| 21 | [Monitoring systému](21-monitoring-systemu.md) | Sledování CPU, paměti, disku, sítě | [Cvičení](pracovni-listy/21-monitoring-systemu/) |
| 22 | [Řešení problémů](22-reseni-problemu.md) | Diagnostika, nástroje, praktické scénáře | [Cvičení](pracovni-listy/22-reseni-problemu/) |
| 23 | [SELinux](23-selinux.md) | Mandatory Access Control, politiky, kontexty | [Cvičení](pracovni-listy/23-selinux/) |
| 24 | [Firewall](24-firewall.md) | iptables, nftables, ufw — správa a konfigurace Linux firewallu | [Cvičení](pracovni-listy/24-firewall/) |
| 25 | [Bezpečnostní audit](25-bezpecnostni-audit.md) | Rootkity, kontrolní součty, AIDE, debsums, detekce průniku | [Cvičení](pracovni-listy/25-bezpecnostni-audit/) |
| 26 | [Webový server](26-web-server.md) | Apache, Nginx, PHP-FPM, gunicorn, PM2, ACME, logování | [Cvičení](pracovni-listy/26-web-server/) |

---

## Cílová skupina

- **Studenti informatiky** – rychlá reference před zkouškami
- **Programátoři** – praktické příklady pro každodenní práci
- **Sysadmini** – přehled příkazů pro správu serverů

---

## Jak používat

Každá sekce obsahuje:
- Teoretický přehled s vysvětlením
- Praktické příklady příkazů (česky + anglické názvy příkazů)
- Shrnutí nejdůležitějších bodů

Doporučený postup je číst sekce po sobě – každá navazuje na předchozí.

---

## Struktura souborů

```
SkriptaOS/
├── README.md                      # Hlavní navigace
├── 00-zaklady-os.md               # Základy OS
├── 01-linux-prikazy-zakladni.md   # Základní Linux příkazy
├── 02-linux-prikazy-rozirene.md   # Rozšířené Linux příkazy
├── 03-windows-prikazy.md          # Windows příkazy
├── 04-bash-skriptovani.md         # Bash skriptování
├── 05-powershell-skriptovani.md   # PowerShell skriptování
├── 06-sprava-procesu.md           # Správa procesů
├── 07-bezpecna-architektura.md    # Bezpečná architektura
├── 08-ovladani-soukromosti.md     # Správa logů a prostředí
├── 09-optimalizace-vykony.md      # Optimalizace výkonu
├── 10-aktualizace-systemu.md    # Aktualizace systému
├── 11-sprava-uzivatelu.md         # Správa uživatelů a skupin
├── 12-prava-souboru.md            # Práva souborů a adresářů
├── 13-sprava-filesystemu.md       # Správa souborového systému
├── 14-zalohovani.md               # Zálohování a obnova
├── 15-apparmor.md                 # AppArmor (MAC)
├── 16-netplan.md                  # Netplan – konfigurace sítě
├── 17-raid-redundance-disku.md     # RAID a Redundance disků
├── 18-proxmox-raid.md              # Proxmox RAID1 Mirror
├── 19-prace-s-ssh.md               # Práce s SSH
├── 20-systemd.md                   # Systemd
├── 21-monitoring-systemu.md        # Monitoring systému
├── 22-reseni-problemu.md           # Řešení problémů
├── 23-selinux.md                   # SELinux
├── 24-firewall.md
├── 25-bezpecnostni-audit.md
└── 26-web-server.md
```
