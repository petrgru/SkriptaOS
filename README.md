# SkriptaOS - Operační systémy

Výuková dokumentace k předmětu **Operační systémy**. Přehled základních příkazů, skriptování, správy procesů, bezpečnosti a optimalizace výkonu v Linuxu i Windows.

Tento kurz je určen pro studenty informatiky a programátory, kteří chtějí pochopit, jak fungují operační systémy z praxe. Každá sekce kombinuje teorii s praktickými příklady.

---

## Obsah

| # | Sekce | Popis |
|---|-------|-------|
| 01 | [Základní Linux příkazy](01-linux-prikazy-zakladni.md) | Navigace, soubory, výstupní proudy, práva |
| 02 | [Rozšířené Linux příkazy](02-linux-prikazy-rozirene.md) | Pipe, grep, sed, awk, find, xargs |
| 03 | [Windows příkazy](03-windows-prikazy.md) | CMD a PowerShell příkazy pro Windows |
| 04 | [Bash skriptování](04-bash-skriptovani.md) | Proměnné, cykly, funkce, argumenty |
| 05 | [PowerShell skriptování](05-powershell-skriptovani.md) | Objekty, pipeline, moduly, cmdlety |
| 06 | [Správa procesů](06-sprava-procesu.md) | Procesy, vlákna, PID, priority, signály |
| 07 | [Bezpečná architektura](07-bezpecna-architektura.md) | Práva, uživatele, skupiny, firewall |
| 08 | [Ovládání soukromosti](08-ovladani-soukromosti.md) | Souborové systémy, environment, logy |
| 09 | [Optimalizace výkonu](09-optimalizace-vykony.md) | CPU, paměť, disk, síť, profilování |
| 10 | [Aktualizace systému](10-aktualizace-a-aktualizace.md) | Balíčky, jádro, patche, rollback |
| 11 | [Správa uživatelů a skupin](11-sprava-uzivatelu.md) | Uživatelé, skupiny, práva, sudo, sudoers |
| 12 | [Práva souborů a adresářů](12-prava-souboru.md) | SUID, SGID, sticky bit, ACL, capabilities, umask |
| 13 | [Správa souborového systému](13-sprava-filesystemu.md) | Disky, oddíly, mkfs, mount, fsck, LVM, swap |
| 14 | [Zálohování a obnova](14-zalohovani.md) | tar, gzip, rsync, dd, 3-2-1 strategie |
| 15 | [AppArmor](15-apparmor.md) | AppArmor – Mandatory Access Control |
| 16 | [Netplan](16-netplan.md) | Netplan – konfigurace sítě |

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
├── 01-linux-prikazy-zakladni.md   # Základní Linux příkazy
├── 02-linux-prikazy-rozirene.md   # Rozšířené Linux příkazy
├── 03-windows-prikazy.md          # Windows příkazy
├── 04-bash-skriptovani.md         # Bash skriptování
├── 05-powershell-skriptovani.md   # PowerShell skriptování
├── 06-sprava-procesu.md           # Správa procesů
├── 07-bezpecna-architektura.md    # Bezpečná architektura
├── 08-ovladani-soukromosti.md     # Ovládání soukromosti
├── 09-optimalizace-vykony.md      # Optimalizace výkonu
├── 10-aktualizace-a-aktualizace.md # Aktualizace systému
├── 11-sprava-uzivatelu.md         # Správa uživatelů a skupin
├── 12-prava-souboru.md            # Práva souborů a adresářů
├── 13-sprava-filesystemu.md       # Správa souborového systému
├── 14-zalohovani.md               # Zálohování a obnova
├── 15-apparmor.md                 # AppArmor (MAC)
└── 16-netplan.md                  # Netplan – konfigurace sítě
```
