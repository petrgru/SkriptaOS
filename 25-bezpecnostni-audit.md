# 25. Bezpečnostní audit a detekce průniku
> Řešení zabezpečení, kontrola integrity a detekce nelegálního přístupu v Linuxu

---

## Úvod

Bezpečnostní audit je systematická kontrola systému s cílem odhalit neoprávněné změny, skrytý malware, rootkity nebo jinou podezřelou aktivitu. Zatímco kapitola 7 se věnovala základní bezpečné architektuře a kapitola 24 firewallu, tato kapitola se zaměřuje na nástroje pro detekci již probíhajícího nebo proběhlého útoku.

Bezpečnostní audit dělíme na několik oblastí:

- **Kontrola integrity souborů** — ověření, zda se soubory od poslední známé verze nezměnily (checksumy).
- **Detekce rootkitu** — hledání skrytých modulů, procesů a změněných systémových binárek.
- **Sledování změn v kritických adresářích** — nástroje jako AIDE nebo Tripwire.
- **Ověření nainstalovaných balíčků** — kontrola, zda binární soubory balíčků nebyly pozměněny.
- **Automatizace pravidelného auditu** — skripty a cron úlohy.

```mermaid
graph TD
  A[Sběr dat] --> B[Kontrola checksumů]
  A --> C[Sken rootkitů]
  A --> D[Kontrola integrity souborů]
  A --> E[Ověření balíčků]
  B --> F[Porovnání s baseline]
  C --> G[Analýza podezřelých souborů]
  D --> H[Detekce změn]
  E --> I[Ověření systémových souborů]
  F --> J[Report]
  G --> J
  H --> J
  I --> J
```

> **Předpoklady:** Základní znalost bash příkazu (kapitoly 1, 2), správy balíčků (kapitola 10) a souborových práv (kapitola 12).

---

## 1. Kontrolní součty (checksumy)

Kontrolní součet (hash, fingerprint) je jednosměrný otisk souboru. Pokud se soubor byť jen nepatrně změní, výsledný hash je zcela jiný. To umožňuje detekovat i sebemenší změny.

### Použité nástroje

| Nástroj | Příkaz | Součást |
|---------|--------|---------|
| MD5 | `md5sum` | coreutils |
| SHA-1 | `sha1sum` | coreutils |
| SHA-256 | `sha256sum` | coreutils |
| BLAKE2 | `b2sum` | coreutils |

Všechny nástroje jsou součástí balíčku `coreutils`, tedy dostupné na každé Linuxové distribuci.

### Generování checksumu

```bash
# SHA-256 otisk jednoho souboru
sha256sum /etc/passwd
# a1b2c3d4e5f6...   /etc/passwd

# Uložení do souboru (baseline)
sha256sum /etc/passwd > passwd.sha256

# Více souborů najednou
sha256sum /etc/passwd /etc/shadow /etc/group > baseline.sha256
# a1b2c3...  /etc/passwd
# d4e5f6...  /etc/shadow
# g7h8i9...  /etc/group
```

### Ověření

```bash
# Ověření jednoho souboru
sha256sum -c passwd.sha256
# /etc/passwd: OK

# Ověření všech souborů v baseline
sha256sum -c baseline.sha256
# /etc/passwd: OK
# /etc/shadow: OK
# /etc/group: OK
```

Při neshodě vypadá výstup takto:

```bash
sha256sum -c baseline.sha256
# /etc/passwd: OK
# /etc/shadow: FAILED
# /etc/group: OK
# sha256sum: WARNING: 1 computed checksum did NOT match
```

### Ostatní algoritmy

```bash
# MD5 (jen pro kompatibilitu, ne pro bezpečnost)
md5sum /etc/passwd
# 5d41402abc4b2a76b9719d911017c592  /etc/passwd

# SHA-1 (oslabený, nedoporučuje se)
sha1sum /etc/passwd

# BLAKE2b (moderní, rychlý)
b2sum /etc/passwd
```

### Porovnání algoritmů

| Algoritmus | Délka hash | Rychlost | Bezpečnost | Použití |
|------------|-----------|----------|------------|---------|
| MD5 | 128 bitů / 32 znaků | Velmi rychlý | Není bezpečný | Kontrola integrity (ne kryptografie) |
| SHA-1 | 160 bitů / 40 znaků | Rychlý | Oslabený | Legacy systémy |
| SHA-256 | 256 bitů / 64 znaků | Střední | Bezpečný | Standard pro ověřování souboru |
| BLAKE2b | 512 bitů / 128 znaků | Velmi rychlý | Bezpečný | Moderní alternativa SHA-256 |

### Bezpečnostní poznámky

- **MD5** má známé kolize (dva různé soubory mohou mít stejný hash). Nepoužívejte ho pro bezpečnostní účely, jen pro kontrolu nezávazných dat.
- **SHA-1** je teoreticky oslabený (2017 publikován útok SHAttered na kolizi). Staré systémy jej mohou používat, pro nové projekty zvolte SHA-256.
- **SHA-256** je aktuální standard. Používá se pro podpisy balíčků, certifikáty a integritní kontrolu.
- **BLAKE2b** je moderní algoritmus, rychlejší než SHA-256 na 64bitových systémech. Součást coreutils (b2sum) od coreutils 8.26.

> **Poznámka:** Checksum pouze kontroluje integritu souboru, ne autenticitu (kdo soubor vytvořil). Pro ověření původu použijte GPG podpis (`gpg --verify`).

---

## 2. Detekce rootkitu

Rootkit je software, který se po průniku do systému snaží utajit svou přítomnost. Typicky modifikuje systémová volání, skryvá procesy, soubory a síťové spojení před standardními nástroji (ps, ls, netstat).

### Jak rootkity fungují

- **Kernel rootkity** — modifikují jádro nebo načítají vlastní modul. Skryjí procesy a soubory na úrovni jádra (např. `sys_call_table` hooking).
- **Userland rootkity** — nahrazují systémové binárky (ps, ls, netstat) vlastními verzemi, které neukazují skryté položky.
- **Bootkit** — infikuje bootloader nebo MBR, načítá se dříve než OS.
- **Firmware rootkit** — infikuje firmware zarízení (UEFI, síová karta, disk).

### chkrootkit

chkrootkit je jednoduchý skener, který kontroluje známé rootkity a podezřelé vzory v systému.

```bash
# Instalace
sudo apt install chkrootkit

# Spuštění základní kontroly
sudo chkrootkit
```

Výstup je členěný do sekcí:

```
ROOTDIR is '/'
Checking 'direct'... not infected
Checking 'bindshell'... not infected
Checking 'lkm'... not infected
Checking 'chkutmp'... not infected
...
Checking 'php_includes'... not infected
```

Důležité je hledat slovo `INFECTED`:

```bash
sudo chkrootkit | grep INFECTED
# (prázdný výstup = žádný nález)
```

Pokud chkrootkit neco najde:

```bash
sudo chkrootkit | grep INFECTED
# Checking 'scalper'... INFECTED
```

Varování: chkrootkit někdy hlásí falešné poplachy na některých bezproblémových systémech. Vždy ověřte ručně.

```bash
# Test "not tested" — moduly, které nebyly otestovány
sudo chkrootkit 2>&1 | grep "not tested"
```

### rkhunter (Rootkit Hunter)

rkhunter je komplexnější nástroj. Oproti chkrootkit kontroluje také systémové binárky (pomocí checksumu), skryté procesy a podezřelé stringy.

```bash
# Instalace
sudo apt install rkhunter

# Vytvoření databáze "known good" souborů (po instalaci)
sudo rkhunter --propupd

# Spuštění kontroly (bez cekání na klávesu)
sudo rkhunter --check --skip-keypress
```

Při prvním spuštění rkhunter vytvoří databázi `/var/lib/rkhunter/db` s otisky systémových souborů. Při dalších kontrolách porovnává aktuální stav.

Výstup:

```
[ ok ] Checking for rootkits [ None found ]
[ ok ] Checking for malware [ None found ]
[ ok ] Checking for suspicious files [ None found ]
[ ok ] Checking file properties [ Warning ]
```

Varování u file properties znamená, že se soubor od doby propupd změnil (např. po aktualizaci):

```
Warning: The file properties have changed:
  File: /bin/ls
  Current hash: a1b2c3...
  Stored hash: d4e5f6...
  Reason: Updated package
```

To je často falešný poplach po apt upgrade. Řešením je znovu spustit `rkhunter --propupd`.

### Automatizace přes cron

```bash
# Denní kontrola v 2:00, výsledky do logu
echo "0 2 * * * root /usr/bin/rkhunter --check --skip-keypress --report-warnings-only | mail -s 'RKhunter report' admin@example.com" | sudo tee /etc/cron.d/rkhunter
```

### Srovnání chkrootkit a rkhunter

| Vlastnost | chkrootkit | rkhunter |
|-----------|-----------|----------|
| Rychlost | Rychlý | Pomalejší (kontroluje binárky) |
| Detekce rootkitu | Znamé rootkity | Znamé rootkity + podezřelé vzory |
| Kontrola binárek | Ne | Ano (pomocí hashe a vlastností) |
| Databáze | Není | Ano (`/var/lib/rkhunter/db`) |
| Falesné poplachy | Někdy (závisí na systému) | Po aktualizacích často |
| Vhodné pro | Rychlá orientační kontrola | Pravidelný hloubkový audit |

> **Důležité:** Oba nástroje detekují pouze *známé* rootkity. Nový nebo modifikovaný rootkit může projít. Proto kombinujeme kontrolu rootkitu s ověřením integrity (AIDE, debsums).

---

## 3. Sledování integrity souborů

Pro kontrolu změn v adresářích jako `/etc`, `/bin`, `/sbin` nebo `/usr` slouží dedikované integritní nástroje. Umožňují vzít "snapshot" kritických souborů a pravidelně jej porovnávat.

### AIDE (Advanced Intrusion Detection Environment)

AIDE vytvoří databázi hashu (SHA-256, SHA-512, RIPEMD160 atd.) ze sledovaných souborů. Při každé kontrole porovná aktuální hashe s databází.

```bash
# Instalace
sudo apt install aide

# Inicializace databáze
sudo aideinit
# Výstup: 
# Start timestamp: 2025-01-17 10:00:00 +0000
# Processing: /etc/aide/aide.conf
# ...
# Done: 42 files, 12 directories, 0 errors
```

`aideinit` vytvoří `/var/lib/aide/aide.db.new`. Pro použití je třeba přejmenovat:

```bash
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

Kontrola:

```bash
sudo aide --check
# AIDE found differences between database and filesystem!!
# ...
# Changed files:
# changed: /etc/hosts
# changed: /etc/ssh/sshd_config
```

Konfigurace v `/etc/aide/aide.conf` určuje, co a jakým algoritmem se kontroluje:

```
# Sledovat /etc se SHA-256 a vsemi atributy
/etc p+i+n+u+g+s+m+c+sha256

# Nezahrnovat logy
!/var/log
```

Aktualizace databáze po legitimních změnách:

```bash
# Po aktualizaci systemu
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

### Tripwire

Tripwire funguje na stejném principu jako AIDE, ale je konfiguračně náročnější.

```bash
# Instalace
sudo apt install tripwire

# Konfigurace (policy)
sudo twadmin --create-polfile /etc/tripwire/twpol.txt

# Inicializace databáze
sudo tripwire --init

# Kontrola
sudo tripwire --check
```

Výstup:

```
Tripwire(R) 2.4.4 Integrity Check Report

Report generated: Mon Jan 17 10:00:00 2025

  Rule Name                       Severity Level    Added   Removed  Modified
  ----------                      --------------    -----   -------  --------
  Tripwire Data Files                 100             0        0        0
  Critical configuration files        100             0        0        0
  ...

Total files scanned:   425
Files added:           0
Files removed:         0
Files modified:        2    ← /etc/hosts, /etc/ssh/sshd_config

Modified objects:
"/etc/hosts"
"/etc/ssh/sshd_config"
```

Aktualizace po legitimní změně:

```bash
sudo tripwire --update --twrfile /var/lib/tripwire/report/*.twr
```

### Workflow integrity monitoringu

```mermaid
graph LR
  A[Inicializace databáze] --> B[Snapshot souborů]
  B --> C[Databáze hashů]
  C --> D[Pravidelná kontrola]
  D --> E{Změna detekována?}
  E -->|Ano| F[Autorizovaná?]
  F -->|Ano| G[Aktualizace databáze]
  F -->|Ne| H[Alert - možný průnik]
  E -->|Ne| D
```

### Srovnání AIDE vs Tripwire

| Vlastnost | AIDE | Tripwire |
|-----------|------|----------|
| Instalace | Jednoduchá | Složitější (nutný klíč) |
| Rychlost | Rychlá | Pomalejší |
| Konfigurace | `/etc/aide/aide.conf` | `/etc/tripwire/twpol.txt` + klíče |
| Databáze | Obyč. soubor | Šifrovaná + podepsaná |
| Hash algoritmy | SHA-256, SHA-512, RIPEMD160 | MD5, SHA-256, SHA-512 |
| Aktualizace | mv aide.db.new aide.db | `tripwire --update` |
| Vhodné pro | Ub/Deb/RHEL (univerzální) | Když je třeba podpis databáze |

> **Platformní poznámka:** Oba nástroje jsou dostupné na všech Linuxových distribucích. AIDE je jednodušší na údržbu a doporučujeme ho pro začínající uživatele.

---

## 4. Integrita nainstalovaných balíčků

Zatímco AIDE a checksumy kontrolují změny v libovolných souborech, debsums (Debian/Ubuntu) a `rpm --verify` (RHEL/Fedora) ověřují soubory přímo proti databázi balíčkového systému. To odhalí, zda byl binární soubor balíčků pozměněn od jeho instalace.

### debsums (Debian/Ubuntu)

debsums porovnává nainstalované soubory s otisky uloženými v balíčkové databázi (`/var/lib/dpkg/info/*.md5sums`).

```bash
# Instalace
sudo apt install debsums

# Kontrola všech balíčků (jen změněné)
sudo debsums -s
# (prázdný výstup = vse v pořádku)

# Kontrola konkrétního balíčku
debsums bash
# /bin/bash                                                  OK
# /bin/sh                                                    OK
# /usr/share/man/man1/bash.1.gz                              OK

# Pokud je soubor změněn:
debsums bash
# /bin/bash                                                  FAILED
```

Parametr `-a` kontroluje i konfigurační soubory:

```bash
sudo debsums -a
```

Kombinace s výpisem jen chyb:

```bash
sudo debsums -s
# /bin/ls                                                    FAILED
# /usr/bin/ps                                                FAILED
```

### rpm --verify (RHEL/Fedora/CentOS)

rpm vestavěné (není třeba instalovat):

```bash
# Kontrola všech balíčků
rpm -Va
# S.5....T.  c /etc/ssh/sshd_config
# .M......    /bin/ls
```

Výstupní kódování:

| Atribut | Význam |
|---------|--------|
| S | Velikost se lisí |
| M | Práva se lisí |
| 5 | MD5 checksum (obsah) se lisí |
| D | Major/minor císla se lisí |
| L | Symlink se lisí |
| T | Cas modifikace se lisí |
| c | Konfiguracní soubor (config) |
| d | Dokumentacní soubor (doc) |

Kontrola konkrétního balíčku:

```bash
rpm -V bash
# (prázdný výstup = vse OK)

rpm -V coreutils
# S.5....T.    /bin/ls
```

### Co dělat, když hash nesouhlasí

1. Zjistěte, jestli soubor nebyl změněn aktualizací (apt upgrade / dnf update).
2. Pokud ano, je to v pořádku — databáze se po aktualizaci sama obnoví.
3. Pokud ne, a změna není zaznamenaná v logu, je to vážný problém. Může jít o rootkit, který nahradil binární soubory.
4. Běžte okamžitě k bodu 7 (Praktický příklad).

> **Platformní poznámka:** `debsums` je dostupný na Debian/Ubuntu a odvozených distribucích. `rpm --verify` je vestavěný v RHEL/Fedora/CentOS a odvozených. Na systemd distribucích lze použít také `systemd-analyze verify` pro kontrolu unit souborů.

---

## 5. Přepočty checksumu po aktualizacích

Každá aktualizace systému (`apt upgrade`, `dnf update`) mění binární soubory. Pokud nepřepočteme integritní databáze, povedou další kontroly k falešným poplachům.

### AIDE automatizace

Po apt upgrade je třeba databázi pregenerovat. Lze to řídit automaticky přes `apt.conf.d`:

```bash
# /etc/apt/apt.conf.d/99aide-update
DPkg::Post-Invoke { "if [ -f /var/lib/aide/aide.db ]; then cp /var/lib/aide/aide.db /var/lib/aide/aide.db.backup; aideinit; mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db; fi"; };
```

Tento script se spustí po každém `apt install` nebo `apt upgrade`. Pokud se `aideinit` nepodaří, zůstává záložní kopie.

Alternativa přes cron (týdne):

```bash
# /etc/cron.weekly/aide-update
#!/bin/bash
if [ -f /var/lib/aide/aide.db ]; then
    cp /var/lib/aide/aide.db /var/lib/aide/aide.db.backup
    aideinit && mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
fi
```

### rkhunter automatizace

```bash
# Prepocet databáze po každém apt upgrade
# /etc/apt/apt.conf.d/99rkhunter-update
DPkg::Post-Invoke { "/usr/bin/rkhunter --propupd"; };
```

Nebo týdenní cron:

```bash
# /etc/cron.weekly/rkhunter-update
#!/bin/bash
/usr/bin/rkhunter --propupd
```

### integrity-check.sh a aktualizace

Pokud používáte vlastní baseline (z další sekce), je třeba ji po každé aktualizaci pregenerovat:

```bash
# Prepocet po apt upgrade
sudo integrity-check.sh -g
```

### Souhrn automatizace

| Nástroj | Trigger | Akce |
|---------|---------|------|
| AIDE | Post-Invoke apt | aideinit + mv |
| AIDE | Cron (nedele 3:00) | aideinit + mv |
| rkhunter | Post-Invoke apt | rkhunter --propupd |
| rkhunter | Cron (nedele 4:00) | rkhunter --propupd |
| integrity-check.sh | Manuálně po změnách | integrity-check.sh -g |

---

## 6. Pomocné skripty

### Skript 1: integrity-check.sh

Skript vytvoří baseline vybraných kritických souborů (SHA-256) a umožňuje jejich pravidelné ověřování.

```bash
#!/bin/bash
# integrity-check.sh - Kontrola integrity kritických systémových souborů
# Použití: sudo ./integrity-check.sh
#   -g (generate) : Vytvoří baseline
#   -c (check)    : Porovná s baseline (výchozí)

BASELINE="/var/log/checksum-baseline.txt"
FILES=("/etc/passwd" "/etc/shadow" "/etc/ssh/sshd_config"
       "/etc/sudoers" "/etc/hosts" "/etc/resolv.conf")

generate_baseline() {
    > "$BASELINE"
    for f in "${FILES[@]}"; do
        if [ -f "$f" ]; then
            sha256sum "$f" >> "$BASELINE"
        fi
    done
    echo "Baseline vytvořena: $BASELINE ($(wc -l < "$BASELINE") souborů)"
}

check_integrity() {
    if [ ! -f "$BASELINE" ]; then
        echo "CHYBA: Baseline neexistuje. Spust s -g."
        exit 1
    fi
    if sha256sum -c "$BASELINE" 2>/dev/null; then
        echo "Všechny soubory v pořádku."
        exit 0
    else
        echo "POZOR: Některé soubory byly změněny!"
        sha256sum -c "$BASELINE" 2>/dev/null | grep -v "OK$"
        exit 2
    fi
}

case "${1:--c}" in
    -g) generate_baseline ;;
    -c) check_integrity ;;
    *) echo "Použití: $0 [-g|-c]" ;;
esac
```

Použití:

```bash
# Vytvoření/obnovení baseline
sudo ./integrity-check.sh -g
# Baseline vytvořena: /var/log/checksum-baseline.txt (6 souborů)

# Kontrola
sudo ./integrity-check.sh -c
# /etc/passwd: OK
# /etc/shadow: OK
# /etc/ssh/sshd_config: OK
# /etc/sudoers: OK
# /etc/hosts: OK
# /etc/resolv.conf: OK
# Všechny soubory v pořádku.
```

### Skript 2: security-audit.sh

Tento skript byl představen již v kapitole 7, ale zde je jeho úplné znění v kontextu bezpečnostního auditu.

```bash
#!/bin/bash
# security-audit.sh - Základní bezpečnostní audit

echo "=== Neúspěšné SSH prihlasení ==="
journalctl -u sshd -p err --since "7 days ago" | grep "Failed password" | wc -l
# 127

echo "=== Posledních 5 neúspěšných pokusu ==="
journalctl -u sshd -p err --since "7 days ago" | grep "Failed password" | tail -5
# Jan 17 09:15:23 server sshd[4521]: Failed password for root from 10.0.0.99 port 54321 ssh2
# Jan 17 09:15:25 server sshd[4523]: Failed password for invalid user admin from 10.0.0.99 port 54322 ssh2

echo "=== Aktuálně prihlásení uživatelé ==="
who
# alice    pts/0        2025-01-17 10:00 (192.168.1.50)
# bob      pts/1        2025-01-17 09:45 (192.168.1.51)

echo "=== Sudo použití za poslední týden ==="
journalctl -t sudo --since "7 days ago" | grep "COMMAND"
# Jan 17 10:00:01 server sudo[5678]: alice : TTY=pts/0 ; PWD=/home/alice ; USER=root ; COMMAND=/usr/bin/apt update

echo "=== Otevřené porty ==="
ss -tlnp
# State   Recv-Q  Send-Q   Local Address:Port     Peer Address:Port  Process
# LISTEN  0       128          0.0.0.0:2222          0.0.0.0:*      users:(("sshd",pid=1234))
# LISTEN  0       128          0.0.0.0:80            0.0.0.0:*      users:(("nginx",pid=2345))
# LISTEN  0       128          0.0.0.0:443           0.0.0.0:*      users:(("nginx",pid=2345))

echo "=== Soubory s SUID/SGID (bez standardních) ==="
find /usr -type f \( -perm -4000 -o -perm -2000 \) | grep -v -E '/(bin|lib)/'
# /usr/local/bin/custom-app

echo "=== Účty bez hesla ==="
awk -F: '($2 == "" || $2 == "!") {print $1 " has no password set"}' /etc/shadow 2>/dev/null
# (prázdný výstup = všechny úcty mají heslo)

echo "=== Zmeny v /etc/passwd za poslední den ==="
ausearch -k passwd_changes --start today 2>/dev/null || echo "auditd rule not configured"
```

### Kombinace skriptu

Pro komplexní kontrolu můžeme skripty zkombinovat:

```bash
#!/bin/bash
# daily-audit.sh - Denní bezpečnostní kontrola

echo "=== 1. Integrity check (baseline) ==="
./integrity-check.sh -c || echo "INTEGRITY FAILURE"

echo "=== 2. Rootkit sken ==="
sudo chkrootkit | grep -E "INFECTED|not tested"

echo "=== 3. AIDE kontrola ==="
sudo aide --check | grep -E "changed:|added:|removed:"

echo "=== 4. Ověření balíčků ==="
sudo debsums -s

echo "=== 5. Security audit ==="
./security-audit.sh
```

---

## 7. Praktický příklad
### Scénář: Detekce podezřelé aktivity na serveru
**Situace:** Spravujete Linux server (Debian 12). Všimne si zvýseného síťového provozu na portu 443 a divné chování webového serveru.
**1. Zahájení auditu**
Administrátor spustí rychlé kontrolní nástroje:
```bash
# Kontrola, co běží na portech
ss -tlnp
# State   Recv-Q  Send-Q   Local Address:Port     Peer Address:Port  Process
# LISTEN  0       128          0.0.0.0:443           0.0.0.0:*      users:(("httpd",pid=9999))
# LISTEN  0       128          0.0.0.0:4444          0.0.0.0:*      users:(("httpd",pid=9999))
```
Nový port 4444 je podezřelý — httpd by tam nemel naslouchat.
**2. Rootkit sken**
```bash
sudo chkrootkit | grep INFECTED
# Checking 'scalper'... INFECTED
```
chkrootkit našel znamý rootkit. Pro potvrzení:
```bash
sudo rkhunter --check --skip-keypress | grep -E "Warning|INFECTED"
# Warning: The file properties have changed:
#   File: /bin/ls
#   File: /bin/ps
#   File: /usr/sbin/sshd
```
Rootkit pravděpodobně nahradil systémové binárky (ls, ps, sshd), aby skryl svou aktivitu.
**3. Integritní kontrola**
```bash
# Vlastní baseline
sudo ./integrity-check.sh -c
# /etc/passwd: OK
# /etc/shadow: OK
# /etc/ssh/sshd_config: FAILED
# /etc/sudoers: OK
# /etc/hosts: OK
# /etc/resolv.conf: OK
# AIDE
sudo aide --check | grep "changed:"
# changed: /bin/ls
# changed: /bin/ps
# changed: /usr/sbin/sshd
# changed: /etc/ssh/sshd_config
```
**4. Ověření balíčků**
```bash
# Na Debian/Ubuntu
sudo debsums -s | grep FAILED
# /bin/ls                                                    FAILED
# /bin/ps                                                    FAILED
# /usr/sbin/sshd                                             FAILED
```
Nebo na RHEL/Fedora:
```bash
rpm -Va | grep -v "c"
# S.5....T.    /bin/ls
# S.5....T.    /bin/ps
# S.5....T.    /usr/sbin/sshd
```
**5. Vyhodnocení**
Výsledky jsou jednoznačné:
| Nástroj | Nález |
|---------|-------|
| chkrootkit | INFECTED (scalper) |
| rkhunter | Warning: změnené binárky |
| integrity-check.sh | sshd_config se lisí |
| AIDE | 4 soubory změněny |
| debsums | 3 balíčky FAILED |
**6. Doporučené kroky**
1. **Okamžitě odpojit server ze sítě** — fyzicky nebo přes iptables:
   ```bash
   iptables -P INPUT DROP
   iptables -P OUTPUT DROP
   ```
2. **Forenzní analýza** (nedotýkat se serveru):
   - Vytvořit obraz disku (dd)
   - Uložit logy (journalctl, /var/log)
   - Zaznamenat procesy (ps aux) a síťové spojení (ss -tlnp)
3. **Preinstalace** — vážné narušení integrity binárek znamená, že server nelze bezpečně vyčistit. Jediným spolehlivým řešením je reinstalace z ověřeného média.
4. **Po reinstalaci**:
   - Aplikovat všechny aktualizace
   - Změnit všechna hesla a SSH klíče
   - Zavést denní integritní kontroly (cron + AIDE + debsums)
   - Zavést pravidelný rootkit sken (rkhunter v cronu)
   - Zpřísnit firewall (viz kapitola 24)
> **Poznámka:** Detekce je až poslední linie obrany. Prevence (firewall, aktualizace, SELinux/AppArmor, bezpečné konfigurace) je vždy lepší než detekce. Viz kapitoly 7, 15, 23, 24.
---
## Shrnutí
Bezpečnostní audit je nepostradatelnou součástí správy každého serveru. Žádný firewall nebo aktualizace není dokonalá — dříve nebo později se může útok podařit. Rozdíl mezi bezvýznamnou událostí a vážným bezpečnostním incidentem je včasná detekce.
### Přehled nástrojů
| Nástroj | Účel | Frekvence | Zdroj |
|---------|------|-----------|-------|
| sha256sum | Kontrola integrity jednotlivých souborů | Podle potřeby | coreutils |
| b2sum | Rychlejší alternativa SHA-256 | Podle potřeby | coreutils |
| chkrootkit | Rychlé skenování známých rootkitu | Denně | apt |
| rkhunter | Hloubková kontrola + ověření binárek | Denně (s --propupd po upgrade) | apt |
| AIDE | Integrity monitoring kritických adresářů | Denně nebo týdně | apt |
| Tripwire | AIDE s podepsanou databází | Denně nebo týdně | apt |
| debsums | Ověření balíčků proti databázi dpkg | Po každém podezření | apt |
| rpm -Va | Ověření balíčků na RHEL/Fedora | Po každém podezření | vestavěný |
| integrity-check.sh | Vlastní baseline kritických souborů | Podle potřeby | vlastní skript |
| security-audit.sh | Rychlá kontrola SSH logu, portu, SUID | Denně | vlastní skript |
### Doporučený harmonogram
| Frekvence | Akce |
|-----------|------|
| Denně | security-audit.sh (SSH logy + porty + uživatele) |
| Denně | chkrootkit (rychlý rootkit scan) |
| Denně | rkhunter --check (hloubková kontrola) |
| Týdně | AIDE --check (integrita /etc, /bin, /sbin, /usr) |
| Po každém apt upgrade | rkhunter --propupd + aideinit + integrity-check.sh -g |
| Při podezření | debsums -s + integrity-check.sh -c + manuální analýza |
### Související kapitoly
- [Kapitola 7: Bezpečná architektura](07-bezpecna-architektura.md) — audit logu, auditd, fail2ban, kernel hardening
- [Kapitola 10: Aktualizace systému](10-aktualizace-systemu.md) — správa balíčků, apt, dpkg
- [Kapitola 12: Práva souborů](12-prava-souboru.md) — SUID, SGID, ACL
- [Kapitola 14: Zálohování a obnova](14-zalohovani.md) — záloha dat před reinstalací
- [Kapitola 15: AppArmor](15-apparmor.md) — mandatory access control
- [Kapitola 19: Práce s SSH](19-prace-s-ssh.md) — bezpečná konfigurace SSH
- [Kapitola 23: SELinux](23-selinux.md) — mandatory access control na RHEL
- [Kapitola 24: Firewall](24-firewall.md) — iptables, nftables, ufw
---
➡️ [Zpět na přehled](README.md)
