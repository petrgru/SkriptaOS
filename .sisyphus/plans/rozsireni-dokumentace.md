# Rozšíření dokumentace SkriptaOS - 6 nových témat

## TL;DR

> **Cíl**: Rozšířit stávající sadu o 6 nových přehledových souborů (11-16) pro správu uživatelů, práva souborů, správu FS, zálohování, AppArmor a Netplan.
>
> **Formát**: Samostatné .md soubory, bilingvální (česky + anglické příklady s výstupy), stejná struktura jako stávající 01-10.
>
> **Rozsah**: 6 nových souborů (~150-250 řádků každý) + aktualizace README.
>
> **Parallel Execution**: ANO - 1 vlna (vše paralelně) + final wave

---

## Kontext

### Co chybí (požadavek uživatele)
- Správa uživatelů a skupin
- Zálohování a obnova
- Práce se souborovým systémem (vytváření, formátování, oblasti)
- Práva souborů a adresářů - základní i rozšířené
- AppArmor (mandatory access control)
- Netplan (konfigurace sítě)

### Překryv s existujícím obsahem
Soubory 11, 12 a 15 částečně překrývají `07-bezpecna-architektura.md`. Rozhodnuto: **hlubší rozšíření + křížový odkaz**. Nové soubory jdou do větší hloubky a na začátku odkazují na 07 pro základní přehled.

### Metis Review
**Critical gaps addressed**:
- 07-overlap: Vyřešeno jako "deeper coverage + cross-reference"
- Filenames: `15-apparmor.md` a `16-netplan.md` (produktové názvy, ponechány)
- Platform notes: Přidány do 15 a 16
- Dangerous commands: Varování u dd, visudo, LVM destruktivních příkazů

---

## Work Objectives

### Core Objective
Vytvořit 6 nových přehledových referenčních příruček + aktualizovat README.

### Concrete Deliverables
- `11-sprava-uzivatelu.md` — Správa uživatelů a skupin
- `12-prava-souboru.md` — Práva souborů a adresářů (základní + rozšířené)
- `13-sprava-filesystemu.md` — Správa souborového systému
- `14-zalohovani.md` — Zálohování a obnova
- `15-apparmor.md` — AppArmor - povinný přístup (MAC)
- `16-netplan.md` — Netplan - konfigurace sítě
- README.md — Aktualizovaný s odkazy na 11-16

### Definition of Done
- [ ] Všech 6 nových souborů existuje v `/home/petrg/Project/SkriptaOS/`
- [ ] README.md obsahuje odkazy na všech 6 nových souborů
- [ ] Každý soubor má strukturu: H1 → subtitle → `---` → `## Úvod` → sekce → `## Shrnutí` → nav link
- [ ] Každý soubor má tabulky se 4 sloupci a sloupcem Výstup

### Must Have
- Bilingvální formát (české vysvětlení + anglické příkazy)
- Konzistentní struktura se stávajícími soubory
- Křížové odkazy na 07 u souborů 11, 12, 15
- Platform-specificity notes u 15 (AppArmor) a 16 (Netplan)
- Varovné boxy u nebezpečných příkazů

### Must NOT Have
- Duplikace obsahu z 07 (jde se do větší hloubky)
- Cloud/DB/container backup tools v 14
- RAID, ZFS/Btrfs hluboký ponor, smartctl v 13
- LDAP, enterprise identity, NSS v 11
- SELinux srovnávací tabulka v 15
- NetworkManager konfigurace v 16
- Více než 250 řádků na soubor

---

## Verification Strategy

### Test Decision
- **Infrastructure exists**: NO (dokumentace)
- **Automated tests**: None
- **QA verifications**: Agent-executed grep/wc kontroly

### QA Scenarios (Agent-Executed)
Každý nový soubor bude automaticky ověřen:
- `wc -l <file>` — musí být 100-250 řádků
- `grep "^# " <file>` — musí mít H1 titulek
- `grep "^## Úvod" <file>` — musí mít úvodní sekci
- `grep "|.*|.*|.*|.*|" <file>` — musí mít tabulku se 4 sloupci
- `grep "^## Shrnutí" <file>` — musí mít shrnutí
- `grep "Zpět na přehled" <file>` — musí mít odkaz na README
- `grep -c "^| 1[1-6] |" README.md` — musí mít 6 nových řádků

---

## Execution Strategy

### Wave 1 — Vše paralelně (Start Immediately)

Všechny soubory jsou nezávislé, lze vytvářet souběžně:

```
Wave 1 (7 v jednom — vše lze paralelizovat):
├── T1: 11-sprava-uzivatelu.md
├── T2: 12-prava-souboru.md
├── T3: 13-sprava-filesystemu.md
├── T4: 14-zalohovani.md
├── T5: 15-apparmor.md
├── T6: 16-netplan.md
└── T7: Aktualizace README.md

Wave FINAL — Ověření:
├── F1: Existence + struktura všech 6 souborů
├── F2: README aktualizace
└── F3: Křížové kontroly (odkazy, varování)
```

---

## TODOs

- [ ] 1. Vytvořit 11-sprava-uzivatelu.md

  **What to do**:
  - Vytvoř soubor `/home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md`
  - Struktura: `# Správa uživatelů a skupin` → `> User and Group Management in Linux` → `---` → `## Úvod` → tematické sekce s tabulkami → `## Shrnutí` → nav link
  - Sekce: Vytváření a správa uživatelů (useradd, usermod, userdel), Správa skupin (groupadd, groupmod, groupdel, gpasswd), Hesla a autentizace (passwd, chage, /etc/shadow), Systémové soubory (/etc/passwd, /etc/shadow, /etc/group), Práva superuživatele (sudo, visudo, /etc/sudoers), Informace o uživatelích (who, w, id, last)
  - V `## Úvod` přidej: `> **Související:** Základní přehled uživatelů a skupin viz [07 - Bezpečná architektura](07-bezpecna-architektura.md)`
  - PŘED sudo/sudoers obsahem: `> **⚠️ VAROVÁNÍ:** `/etc/sudoers` vždy editujte příkazem `visudo`, který kontroluje syntaxi. Chyba v sudoers může znemožnit přístup k root účtu.`
  - Formát tabulek: `| Příkaz | Popis | Příklad | Výstup |`
  - Konec: `---` → `➡️ [Zpět na přehled](README.md)`

  **Must NOT do**:
  - Nezahrnovat SSH key management (již v 07)
  - Nezahrnovat PAM detail (již v 07)
  - Nezahrnovat LDAP, enterprise identity, NSS

  **Acceptance Criteria**:
  - [ ] Soubor existuje a má 100-250 řádků
  - [ ] Má H1 titulek, `## Úvod`, `## Shrnutí`
  - [ ] Má alespoň 2 tabulky se 4 sloupci
  - [ ] Obsahuje křížový odkaz na 07
  - [ ] Obsahuje visudo varování
  - [ ] Končí odkazem na README
  - [ ] Obsahuje české popisy + anglické příklady

  **QA Scenarios**:
  ```
  Scenario: Ověření struktury souboru
    Tool: Bash
    Preconditions: Soubor existuje
    Steps:
      1. `wc -l < /home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md` → mělo by být 100-250
      2. `grep -q "^# Správa uživatelů" /home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md` → exit 0
      3. `grep -q "^## Úvod" /home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md` → exit 0
      4. `grep -q "^## Shrnutí" /home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md` → exit 0
      5. `grep -q "Zpět na přehled" /home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md` → exit 0
      6. `grep -c "|.*|.*|.*|.*|" /home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md` → >= 2
      7. `grep -c "useradd\|usermod\|userdel\|groupadd\|passwd" /home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md` → >= 3
      8. `grep -q "visudo" /home/petrg/Project/SkriptaOS/11-sprava-uzivatelu.md` → exit 0 (warning present)
    Expected Result: Všechny grep příkazy vrací exit 0, wc je v rozmezí 100-250
    Evidence: .sisyphus/evidence/rozsireni-11-struktura.log
  ```

  **Commit**: YES
  - Message: `docs: add user management reference (11-sprava-uzivatelu)`
  - Files: `11-sprava-uzivatelu.md`

---

- [ ] 2. Vytvořit 12-prava-souboru.md

  **What to do**:
  - Vytvoř soubor `/home/petrg/Project/SkriptaOS/12-prava-souboru.md`
  - Struktura: `# Práva souborů a adresářů` → `> File and Directory Permissions in Linux` → `---` → `## Úvod` → tematické sekce → `## Shrnutí` → nav link
  - Sekce:
    - **Základní práva**: chmod (symbolicky i oktálově), chown, chgrp, umask (s výpočtem: 666 - umask pro soubory, 777 - umask pro adresáře)
    - **Speciální práva**: SUID (chmod u+s), SGID (chmod g+s), sticky bit (chmod +t), jejich význam a zjištění (find / -perm ...)
    - **Rozšířená práva - ACL**: getfacl, setfacl, mask, default ACL, praktické použití (sdílený adresář)
    - **Atributy souborů**: chattr (immutable +i, append-only +a), lsattr
  - V `## Úvod` přidej křížový odkaz: `> **Související:** Základní přehled práv a bezpečnosti viz [07 - Bezpečná architektura](07-bezpecna-architektura.md)`
  - Formát tabulek: `| Příkaz/Koncept | Popis | Příklad | Výstup |`

  **Must NOT do**:
  - Nezahrnovat SELinux kontext (chcon, restorecon)
  - Nebýt kopií 07 — jde do větší hloubky (např. výpočet umask, kombinace ACL+umask)

  **QA Scenarios**:
  ```
  Scenario: Ověření struktury 12
    Tool: Bash
    Steps:
      1. `wc -l < 12-prava-souboru.md` → 100-250
      2. `grep -q "^## Úvod" 12-prava-souboru.md` AND `grep -q "^## Shrnutí" 12-prava-souboru.md`
      3. `grep -c "|.*|.*|.*|.*|" 12-prava-souboru.md` → >= 2
      4. `grep -q "chmod\|chown\|umask\|getfacl\|setfacl\|chattr\|SUID\|SGID\|sticky\|ACL" 12-prava-souboru.md`
      5. `grep "Zpět na přehled" 12-prava-souboru.md`
    Expected Result: Všechny kontroly prochází
    Evidence: .sisyphus/evidence/rozsireni-12-struktura.log
  ```

  **Commit**: YES
  - Message: `docs: add file permissions reference (12-prava-souboru)`
  - Files: `12-prava-souboru.md`

---

- [ ] 3. Vytvořit 13-sprava-filesystemu.md

  **What to do**:
  - Vytvoř soubor `/home/petrg/Project/SkriptaOS/13-sprava-filesystemu.md`
  - Struktura: `# Správa souborového systému` → `> Filesystem Management in Linux` → `---` → `## Úvod` → tematické sekce → `## Shrnutí` → nav link
  - Sekce:
    - **Zobrazení disků a oddílů**: lsblk, blkid, fdisk -l, df -h
    - **Správa oddílů**: fdisk (MBR), gdisk (GPT), parted — vytvoření, smazání, změna velikosti
    - **Vytváření FS**: mkfs.ext4, mkfs.xfs, mkfs.btrfs, mkswap
    - **Připojování FS**: mount, umount, /etc/fstab (struktura, UUID), mount options (noatime, discard, defaults)
    - **Kontrola a oprava**: fsck, e2fsck, xfs_repair
    - **LVM základy**: pvcreate, vgcreate, lvcreate, lvextend, pvs, vgs, lvs
    - **Swap**: swapon, swapoff, swapfile, priority
  - Před LVM destruktivními příkazy: `> **⚠️ VAROVÁNÍ:** Příkazy `lvremove`, `pvremove`, `vgremove` nenávratně ničí data. Vždy zálohujte před destruktivními operacemi s LVM.`
  - Formát tabulek: `| Příkaz | Popis | Příklad | Výstup |`

  **Must NOT do**:
  - RAID, mdadm
  - ZFS/Btrfs hluboké funkce (snapshoty, komprese... kromě mkfs příkladu)
  - smartctl, iostat, monitoring
  - Thin provisioning, LVM snapshoty

  **QA Scenarios**:
  ```
  Scenario: Ověření struktury 13
    Tool: Bash
    Steps:
      1. `wc -l < 13-sprava-filesystemu.md` → 100-250
      2. grep kontroly struktury (Úvod, Shrnutí, tabulky, zpět na README)
      3. `grep "mkfs\|fdisk\|mount\|LVM\|fsck\|swap" 13-sprava-filesystemu.md`
      4. `grep -i "varovani\|warning\|⚠" 13-sprava-filesystemu.md` → exit 0 (varování existuje)
    Evidence: .sisyphus/evidence/rozsireni-13-struktura.log
  ```

  **Commit**: YES
  - Message: `docs: add filesystem management reference (13-sprava-filesystemu)`
  - Files: `13-sprava-filesystemu.md`

---

- [ ] 4. Vytvořit 14-zalohovani.md

  **What to do**:
  - Vytvoř soubor `/home/petrg/Project/SkriptaOS/14-zalohovani.md`
  - Struktura: `# Zálohování a obnova` → `> Backup and Restore in Linux` → `---` → `## Úvod` → tematické sekce → `## Shrnutí` → nav link
  - Sekce:
    - **Archivace**: tar (create, extract, list, compression flags -z, -j, -J)
    - **Komprese**: gzip, gunzip, bzip2, xz, zstd, porovnání kompresních poměrů
    - **Synchronizace**: rsync (lokálně, přes SSH, --archive, --delete, --dry-run)
    - **Bitová kopie**: dd (if, of, bs, status=progress), včetně prominentního varování
    - **Zálohovací strategie**: 3-2-1 pravidlo (3 kopie, 2 média, 1 mimo lokalitu)
  - Před dd sekcí: 
    ```
    > **⚠️ VAROVÁNÍ:** `dd` pracuje přímo s blokovými zařízeními. 
    > Přesměrování na špatný `of=` může nenávratně zničit data na disku.
    > **VŽDY** dvakrát zkontrolujte cílové zařízení (`lsblk`, `blkid`).
    ```
  - Základní cron příklad pro automatizaci: `0 2 * * * /usr/bin/rsync -a /home /backup/`
  - Formát tabulek: `| Nástroj | Popis | Příklad | Výstup |`

  **Must NOT do**:
  - Cloud backup (AWS S3, rsync.net, apod.)
  - Database backup (pg_dump, mysqldump)
  - Container backup
  - Backup automation beyond single cron line
  - restic, borg, duplicity

  **QA Scenarios**:
  ```
  Scenario: Ověření struktury 14
    Tool: Bash
    Steps:
      1. `wc -l < 14-zalohovani.md` → 100-250
      2. grep kontroly struktury
      3. `grep "tar\|rsync\|dd\|gzip\|3-2-1" 14-zalohovani.md`
      4. `grep -i "varovani\|warning\|⚠" 14-zalohovani.md | grep -i dd` → exit 0 (dd varování)
      5. `grep "cron\|0 2" 14-zalohovani.md` → exit 0 (automatizace zmíněna)
    Evidence: .sisyphus/evidence/rozsireni-14-struktura.log
  ```

  **Commit**: YES
  - Message: `docs: add backup and restore reference (14-zalohovani)`
  - Files: `14-zalohovani.md`

---

- [ ] 5. Vytvořit 15-apparmor.md

  **What to do**:
  - Vytvoř soubor `/home/petrg/Project/SkriptaOS/15-apparmor.md`
  - Struktura: `# AppArmor - povinný přístup (MAC)` → `> Mandatory Access Control with AppArmor` → `---` → `## Úvod` → tematické sekce → `## Shrnutí` → nav link
  - Hned na začátku - platformní poznámka: `> **Poznámka k platformě:** AppArmor je výchozí na Ubuntu/Debianu a derivátech. Na RHEL/CentOS/Fedora se používá SELinux — viz [07 - Bezpečná architektura](07-bezpecna-architektura.md).`
  - V `## Úvod` křížový odkaz: `> **Související:** Základní pojmy MAC a přehled bezpečnostních mechanismů viz [07 - Bezpečná architektura](07-bezpecna-architektura.md)`
  - Sekce:
    - **Základní pojmy**: co je AppArmor, profil, enforce/complain/disable mód
    - **Práce s profily**: aa-status, aa-enforce, aa-complain, aa-disable
    - **Tvorba profilů**: aa-genprof (generování), aa-logprof (úprava podle logů)
    - **Syntaxe profilů**: struktura (include, capability, file, network, deny)
    - **Řešení problémů**: aa-notify, audit.log, journalctl, `dmesg | grep apparmor`
  - Formát tabulek: `| Příkaz/Koncept | Popis | Příklad | Výstup |`

  **Must NOT do**:
  - SELinux srovnávací tabulka (pouze zmínka v platformní poznámce)
  - Nebýt kopií 07 — jde do větší hloubky (tvorba profilů, syntaxe)

  **QA Scenarios**:
  ```
  Scenario: Ověření struktury 15
    Tool: Bash
    Steps:
      1. `wc -l < 15-apparmor.md` → 100-250
      2. grep kontroly struktury
      3. `grep "aa-status\|aa-enforce\|aa-complain\|aa-genprof\|aa-logprof" 15-apparmor.md`
      4. `grep -i "Ubuntu\|Debian\|platform" 15-apparmor.md` → exit 0 (platform note)
      5. `grep "07-bezpecna" 15-apparmor.md` → exit 0 (cross-reference)
    Evidence: .sisyphus/evidence/rozsireni-15-struktura.log
  ```

  **Commit**: YES
  - Message: `docs: add AppArmor reference (15-apparmor)`
  - Files: `15-apparmor.md`

---

- [ ] 6. Vytvořit 16-netplan.md

  **What to do**:
  - Vytvoř soubor `/home/petrg/Project/SkriptaOS/16-netplan.md`
  - Struktura: `# Netplan - konfigurace sítě` → `> Network Configuration with Netplan` → `---` → `## Úvod` → tematické sekce → `## Shrnutí` → nav link
  - Hned na začátku - platformní poznámka: `> **Poznámka k platformě:** Netplan je výchozí nástroj pro konfiguraci sítě na Ubuntu 18.04 a novějších. Na RHEL/CentOS/Fedora se používá tradičně `nmcli` nebo ifcfg soubory.`
  - Sekce:
    - **Úvod do Netplan**: YAML syntaxe, /etc/netplan/ adresář, renderer (NetworkManager vs systemd-networkd)
    - **Základní konfigurace**: DHCP, statická IP (addresses, gateway4/6, nameservers)
    - **Pokročilé**: VLAN, bridge (základní příklad)
    - **Správa**: netplan apply, netplan try (s bezpečnostním timeoutem), netplan generate, netplan info
    - **Řešení problémů**: netplan --debug apply, `journalctl -u netplan`, `networkctl`, kontrola YAML syntaxe
  - Před netplan try: `> **Tip:** `netplan try` aplikuje konfiguraci s timeoutem (výchozí 120s). Pokud se spojení ztratí, změny se automaticky vrátí. VŽDY používejte `try` před `apply` na vzdálených systémech.`
  - Formát tabulek: `| Příkaz/Koncept | Popis | Příklad | Výstup |`

  **Must NOT do**:
  - NetworkManager konfigurace přímo (pouze zmínka jako renderer)
  - Bond/bonding advanced (pouze základní bridge příklad)
  - ifupdown (/etc/network/interfaces) — jen zmínka pro starší systémy

  **QA Scenarios**:
  ```
  Scenario: Ověření struktury 16
    Tool: Bash
    Steps:
      1. `wc -l < 16-netplan.md` → 100-250
      2. grep kontroly struktury
      3. `grep "netplan apply\|netplan try\|DHCP\|static\|YAML" 16-netplan.md`
      4. `grep -i "Ubuntu\|platform" 16-netplan.md` → exit 0 (platform note)
      5. `grep "netplan try" 16-netplan.md` → exit 0 (safety tip)
    Evidence: .sisyphus/evidence/rozsireni-16-struktura.log
  ```

  **Commit**: YES
  - Message: `docs: add Netplan network configuration reference (16-netplan)`
  - Files: `16-netplan.md`

---

- [ ] 7. Aktualizovat README.md

  **What to do**:
  - Přidej 6 nových řádků do tabulky `## Obsah` v `/home/petrg/Project/SkriptaOS/README.md`
  - Formát (přidat za stávající řádek 10):
  ```
  | 11 | [Správa uživatelů a skupin](11-sprava-uzivatelu.md) | useradd, groupadd, passwd, sudoers, /etc/shadow |
  | 12 | [Práva souborů a adresářů](12-prava-souboru.md) | chmod, chown, ACL, SUID/SGID, chattr, umask |
  | 13 | [Správa souborového systému](13-sprava-filesystemu.md) | fdisk, mkfs, mount, LVM, fsck, swap |
  | 14 | [Zálohování a obnova](14-zalohovani.md) | tar, rsync, dd, gzip, 3-2-1 strategie |
  | 15 | [AppArmor - MAC](15-apparmor.md) | Profily, aa-status, aa-enforce, aa-genprof |
  | 16 | [Netplan - konfigurace sítě](16-netplan.md) | YAML, DHCP, statická IP, netplan apply/try |
  ```
  - Aktualizuj strukturu souborů na konci README (přidej 11-16)

  **QA Scenarios**:
  ```
  Scenario: Ověření README aktualizace
    Tool: Bash
    Steps:
      1. `grep -c "^| 1[1-6] |" README.md` → mělo by být 6
      2. Pro každý grep: `grep "| 11 |" README.md && grep "| 12 |" README.md && ...`
      3. Všechny odkazy vedou na existující soubory:
         `for i in 11 12 13 14 15 16; do test -f "${i}-*.md" && echo "OK: $i" || echo "MISSING: $i"; done`
    Expected Result: 6 nových řádků, všechny soubory existují
    Evidence: .sisyphus/evidence/rozsireni-readme.log
  ```

  **Commit**: YES (sdružit s jedním z předchozích)
  - Message: `docs: add system management references to README`
  - Files: `README.md`

---

## Final Verification Wave

- [ ] F1. **Existence a struktura** — `oracle`
  Ověř že všech 6 souborů existuje a splňuje strukturální požadavky. Zkontroluj:
  - `wc -l` každého souboru: 100-250
  - grep pro Úvod, Shrnutí, tabulky, nav link
  - Viz QA Scenarios výše

- [ ] F2. **Konzistence a křížové odkazy** — `unspecified-high`
  Ověř:
  - README obsahuje 6 nových řádků
  - Soubory 11, 12, 15 obsahují křížový odkaz na 07
  - Soubory 15 a 16 obsahují platformní poznámku
  - Varování u dd (14), visudo (11), LVM (13)

---

## Commit Strategy

- **1-7**: `docs: add system management references (11-16)` — skupinový commit
  - Files: `11-sprava-uzivatelu.md 12-prava-souboru.md 13-sprava-filesystemu.md 14-zalohovani.md 15-apparmor.md 16-netplan.md README.md`

---

## Success Criteria

### Verification Commands
```bash
ls -la /home/petrg/Project/SkriptaOS/*.md | wc -l        # Expected: 17 (was 11)
grep -c "^| 1[1-6] |" README.md                           # Expected: 6
for f in 11 12 13 14 15 16; do echo -n "$f: "; wc -l < "${f}-"*.md; done  # Expected: 100-250 each
```

### Final Checklist
- [ ] All 6 new files exist
- [ ] README links to all 6 new files
- [ ] Each file has proper structure (Úvod, Shrnutí, tabulky, nav link)
- [ ] Cross-references to 07 present where needed
- [ ] Platform notes present in 15 and 16
- [ ] Warnings present for dangerous commands
