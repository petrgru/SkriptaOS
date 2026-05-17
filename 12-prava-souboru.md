# Práva souborů a adresářů

> File and Directory Permissions in Linux

---

## Úvod

> **Související:** Základní přehled práv, chmod a ACL viz [07 - Bezpečná architektura](07-bezpecna-architektura.md)

Tato kapitola rozšiřuje téma oprávnění nad rámec základního přehledu v sekci 07. Pokrývá hlouběji standardní oprávnění, speciální bity, rozšířené atributy, ACL, capabilities a nástroje pro audit a diagnostiku.

---

## 1. Standardní oprávnění (rwx) — rozšířené

Základní oprávnění rwx (read/write/execute) pro tři kategorie: vlastník (u), skupina (g), ostatní (o).

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `chmod` | Symbolická změna (přidání/odebrání) | `chmod u+r,g-wx,o= file.txt` | (žádný výstup) |
| `chmod` | Číselná (oktálová) změna | `chmod 2755 script.sh` | (žádný výstup) |
| `chmod -R` | Rekurzivní změna práv | `chmod -R g+w /shared/project/` | (žádný výstup) |
| `chmod --reference` | Kopírování práv z referenčního souboru | `chmod --reference=ref.txt target.txt` | (žádný výstup) |
| `stat` | Zobrazení oprávnění včetně číselné hodnoty | `stat -c "%a %A %n" file.txt` | `644 -rw-r--r-- file.txt` |
| `ls -la` | Dlouhý výpis s oprávněními | `ls -la script.sh` | `-rwxr-xr-x 1 alice dev 1024 Jan 15 10:00 script.sh` |

### Oktálová reprezentace

| Právo | Symbol | Oktál | Binárně |
|-------|--------|-------|---------|
| --- | 0 | 000 |
| --x | x | 1 | 001 |
| -w- | w | 2 | 010 |
| -wx | wx | 3 | 011 |
| r-- | r | 4 | 100 |
| r-x | r-x | 5 | 101 |
| rw- | rw- | 6 | 110 |
| rwx | rwx | 7 | 111 |

```bash
# Speciální bity + standardní (4=SUID, 2=SGID, 1=Sticky)
chmod 4755 script.sh    # SUID + rwxr-xr-x
chmod 2770 /shared      # SGID + rwxrwx---
chmod 1777 /tmp         # Sticky + rwxrwxrwx

# Rekurzivní nastavení různých práv pro soubory a adresáře
find /shared -type f -exec chmod 644 {} \;
find /shared -type d -exec chmod 755 {} \;

# Zobrazení číselných oprávnění
stat -c "%a" file.txt
# Výstup: 644
```

---

## 2. Speciální oprávnění (SUID, SGID, Sticky bit)

| Speciální bit | Popis | Příkaz | Příklad výstupu `ls -la` |
|---------------|-------|--------|--------------------------|
| **SUID** (4xxx) | Soubor se spouští s právy vlastníka | `chmod u+s /usr/bin/prog` | `-rwsr-xr-x` |
| **SGID** (2xxx) | Soubor se spouští s právy skupiny; u adresářů nové soubory dědí skupinu | `chmod g+s /shared/dir` | `drwxrws---` |
| **Sticky bit** (1xxx) | Pouze vlastník nebo root může mazat soubory v adresáři | `chmod +t /shared/dir` | `drwxrwxrwt` |

### SUID — hledání a rizika

```bash
# Najít všechny SUID soubory v systému
find / -perm -4000 -type f 2>/dev/null
# Výstup (zkráceně):
# /usr/bin/su
# /usr/bin/sudo
# /usr/bin/passwd
# /usr/bin/pkexec

# Najít SGID soubory
find / -perm -2000 -type f 2>/dev/null

# Najít soubory s oběma (SUID+SGID)
find / -perm -6000 -type f 2>/dev/null

# Zkontrolovat Sticky bit adresářů
ls -ld /tmp /var/tmp
# drwxrwxrwt  ... /tmp
# drwxrwxrwt  ... /var/tmp
```

> **⚠️ BEZPEČNOST:** SUID binárky jsou častým cílem exploitů. Pravidelně auditujte `find / -perm -4000` a odstraňte SUID z nepotřebných programů: `chmod u-s /path/to/binary`.

---

## 3. Výchozí oprávnění (umask)

`umask` určuje, která oprávnění budou **odebrána** při vytváření nového souboru nebo adresáře.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `umask` | Zobrazení aktuální masky | `umask` | `0022` |
| `umask 0xxx` | Dočasné nastavení masky | `umask 0077` | (žádný výstup) |
| `umask -S` | Zobrazení masky symbolicky | `umask -S` | `u=rwx,g=rx,o=rx` |

### Jak umask funguje

```
Výchozí:  soubor 666 (rw-rw-rw-), adresář 777 (rwxrwxrwx)
umask 022: soubor → 644 (rw-r--r--), adresář → 755 (rwxr-xr-x)
umask 077: soubor → 600 (rw-------), adresář → 700 (rwx------)

Výpočet: 666 - umask = výsledná oprávnění (pro soubory)
          777 - umask = výsledná oprávnění (pro adresáře)
```

```bash
# Příklady různých umask hodnot
umask 002   # rw-rw-r-- (soubory), rwxrwxr-x (adresáře) — týmová spolupráce
umask 022   # rw-r--r-- (soubory), rwxr-xr-x (adresáře) — výchozí
umask 027   # rw-r----- (soubory), rwxr-x--- (adresáře) — bezpečné
umask 077   # rw------- (soubory), rwx------ (adresáře) — privátní

# Trvalé nastavení pro uživatele (v ~/.bashrc nebo ~/.profile)
echo "umask 027" >> ~/.bashrc

# Systémová výchozí maska v /etc/login.defs
grep UMASK /etc/login.defs
# UMASK 022
```

---

## 4. Rozšířené atributy souborů (chattr / lsattr)

Nástroje pro nastavení nízkoúrovňových atributů na souborových systémech ext2/3/4.

| Atribut | Popis | Příkaz | Příklad |
|---------|-------|--------|---------|
| `+i` (immutable) | Soubor nelze měnit, mazat, přejmenovávat ani na něj vytvářet linky | `chattr +i /etc/important.conf` | `lsattr /etc/important.conf` → `----i--------e--` |
| `+a` (append-only) | Pouze přidávání dat (ideální pro logy) | `chattr +a /var/log/app.log` | `-----a--------e--` |
| `+e` (extent) | Použití extentů pro ukládání (výchozí na ext4) | `chattr +e file` | `-------------e--` |
| `+c` (compressed) | Komprimovaný soubor (jádro transparentně dekomprimuje) | `chattr +c bigfile` | `-------c------e--` |
| `+s` (secure deletion) | Nulování při mazání (bezpečné smazání) | `chattr +s secret` | `------------s---` |
| `+u` (undeletable) | Umožňuje obnovu smazaného souboru | `chattr +u important` | `-------------u--` |

```bash
# Ochrana konfiguračních souborů před náhodnou změnou
sudo chattr +i /etc/passwd /etc/shadow /etc/sudoers
# Zrušení
sudo chattr -i /etc/passwd

# Log soubor pouze pro zápis
sudo chattr +a /var/log/auth.log

# Zobrazení atributů všech souborů v adresáři
lsattr /etc/
# ----i--------e-- /etc/passwd
# ----i--------e-- /etc/shadow
# --------------e-- /etc/hosts
```

---

## 5. Access Control Lists (ACL) — rozšířené

ACL umožňují jemněji nastavovat oprávnění pro konkrétní uživatele a skupiny nad rámec standardního rwx modelu.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `getfacl` | Zobrazení ACL | `getfacl file.txt` | `# file: file.txt`<br>`user::rw-`<br>`user:bob:r--`<br>`group::r--`<br>`mask::rw-`<br>`other::---` |
| `setfacl -m u:USER:PERMS` | Nastavení pro uživatele | `setfacl -m u:john:rwx /shared` | (žádný) |
| `setfacl -m g:GROUP:PERMS` | Nastavení pro skupinu | `setfacl -m g:dev:rx /shared` | (žádný) |
| `setfacl -m m::PERMS` | Nastavení masky | `setfacl -m m::rx /shared` | (žádný) |
| `setfacl -d -m` | Výchozí ACL (dědičnost pro nové soubory) | `setfacl -d -m g:dev:rwx /shared` | (žádný) |
| `setfacl -x u:USER` | Odebrání ACL položky | `setfacl -x u:john /shared` | (žádný) |
| `setfacl -b` | Odebrání všech ACL | `setfacl -b /shared` | (žádný) |

```bash
# Sdílený adresář pro tým s dědičností
mkdir /srv/team-project
sudo setfacl -m g:developers:rwx /srv/team-project
sudo setfacl -d -m g:developers:rwx /srv/team-project
sudo setfacl -m o::--- /srv/team-project

# Ověření
getfacl /srv/team-project
# # file: /srv/team-project
# # owner: root
# # group: root
# user::rwx
# group::r-x
# group:developers:rwx
# mask::rwx
# other::---
# default:user::rwx
# default:group::r-x
# default:group:developers:rwx
# default:mask::rwx
# default:other::---

# Přidání konkrétního uživatele
sudo setfacl -m u:external-consultant:r-x /srv/team-project

# Zjištění, které soubory mají ACL
ls -la /srv/team-project/
# drwxrwx---+   # + na konci oprávnění = ACL aktivní
```

---

## 6. Linux Capabilities

Capabilities rozdělují práva superuživatele (root) na jemné jednotky. Místo SUID lze programu přidělit jen konkrétní schopnosti.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `getcap` | Zobrazení capabilities souboru | `getcap /usr/bin/ping` | `/usr/bin/ping = cap_net_raw+ep` |
| `setcap` | Nastavení capabilities | `sudo setcap cap_net_raw+ep /usr/bin/custom-ping` | (žádný) |
| `capsh --print` | Zobrazení capabilities aktuálního procesu | `capsh --print` | `Current: = cap_chown,cap_dac_override,...` |
| `getpcaps PID` | Zobrazení capabilities procesu | `getpcaps $$` | `Capabilities for '1234': = cap_net_admin+ep` |

### Běžné capability

| Capability | Význam |
|------------|--------|
| `cap_net_raw` | Surový přístup k síti (ping, ARP) |
| `cap_net_bind_service` | Bind na privilegované porty (<1024) |
| `cap_net_admin` | Správa síťového rozhraní (firewall, routing) |
| `cap_sys_admin` | Různé administrační operace (mount, swapon) |
| `cap_dac_override` | Obejít kontrolu oprávnění (override rwx) |
| `cap_chown` | Měnit vlastníka souborů |
| `cap_kill` | Posílat signály libovolnému procesu |
| `cap_setuid` | Nastavit UID procesu |

```bash
# Místo SUID použít capabilities
sudo setcap cap_net_raw+ep /usr/bin/ping
# Ověření
getcap /usr/bin/ping
# /usr/bin/ping = cap_net_raw+ep

# Odebrání capabilities
sudo setcap -r /usr/bin/ping

# Zobrazení všech souborů s capabilities v systému
sudo getcap -r /usr/bin/
# /usr/bin/ping = cap_net_raw+ep
# /usr/bin/traceroute6.iputils = cap_net_raw+ep
```

> **📌 Poznámka:** Capabilities jsou modernější a bezpečnější alternativa k SUID. Místo binárek s SUID (které mají plný root přístup) lze udělit jen konkrétní schopnosti.

---

## 7. Audit a diagnostika oprávnění

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `namei -l` | Zobrazení oprávnění celé cesty | `namei -l /home/alice/secret/file.txt` | `drwxr-xr-x /`<br>`drwxr-xr-x home`<br>`drwx------ alice`<br>`-rw-r--r-- file.txt` |
| `find -perm` | Hledání souborů podle oprávnění | `find / -perm 6000 -type f` | `/usr/bin/su`<br>`/usr/bin/sudo` |
| `find -nouser` | Soubory bez vlastníka (orphan) | `find / -nouser -o -nogroup 2>/dev/null` | `/tmp/orphan_file.txt` |
| `ls -la` s kontrolou ACL | Detekce ACL pomocí `+` | `ls -la /shared` | `drwxrwx---+` |
| `auditctl` | Sledování přístupu k souboru (audit) | `auditctl -w /etc/passwd -p wa -k passwd_watch` | (žádný) |

```bash
# Auditování změn oprávnění v adresáři
sudo auditctl -w /etc/passwd -p wa -k passwd-change
sudo ausearch -k passwd-change
# time->Mon Jan 15 10:30:00 2025
# type=PROCTITLE ... proctitle=vim /etc/passwd

# Hledání světově zapisovatelných souborů
find / -perm -o+w -type f 2>/dev/null | head -10

# Hledání adresářů, kde kdokoliv může mazat (bez sticky bitu)
find / -perm -o+w -type d ! -perm -1000 2>/dev/null

# Kontrola, jestli má uživatel přístup k souboru
sudo -u alice test -r /etc/shadow && echo "Může číst" || echo "Nemůže číst"
```

---

## 8. Oprávnění ve Windows (NTFS)

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `icacls` | Zobrazení a změna NTFS oprávnění | `icacls C:\Shared\Project` | `C:\Shared\Project NT AUTHORITY\SYSTEM:(OI)(CI)F`<br>`BUILTIN\Administrators:(OI)(CI)F` |
| `icacls /grant` | Udělení oprávnění | `icacls C:\Shared /grant "Domain\Users:(OI)(CI)R"` | `processed file: C:\Shared` |
| `icacls /remove` | Odebrání oprávnění | `icacls C:\Shared /remove "Domain\Users"` | `processed file: C:\Shared` |
| `takeown` | Převzetí vlastnictví | `takeown /F C:\Protected\file.txt /A` | `SUCCESS: The file (or folder): ... now owned by Administrators` |

```bash
# NTFS permissions: (OI)=Object inherit, (CI)=Container inherit
# F=Full, M=Modify, RX=Read&Execute, R=Read, W=Write
icacls C:\Shared /grant "Domain\Users:(OI)(CI)RX"
icacls C:\Shared /grant "Domain\Developers:(OI)(CI)M"
icacls C:\Shared /grant "Creator Owner:(OI)(CI)F"

# Zobrazení efektivních práv
icacls C:\Shared /findsid "Domain\alice"

# Převzetí vlastnictví
takeown /F C:\Protected\* /R
```

---

## Shrnutí

- **Standardní oprávnění** rwx se nastavuje `chmod` (symbolicky nebo oktálově). `chown` mění vlastníka, `chgrp` skupinu.
- **Speciální bity** SUID (4), SGID (2), Sticky (1) se přidávají jako čtvrtá číslice k oktálovým právům.
- **umask** určuje výchozí oprávnění pro nové soubory. Vzorec: `666 - umask` (soubor), `777 - umask` (adresář).
- **chattr/lsattr** poskytují nízkoúrovňové atributy jako immutable (`+i`) a append-only (`+a`).
- **ACL** (`setfacl`) umožňují jemné oprávnění pro konkrétní uživatele/skupiny. Výchozí ACL (`-d`) řeší dědičnost.
- **Linux capabilities** nahrazují SUID jemnozrnnými právy. `setcap cap_net_raw+ep` místo SUID.
- **Audit** pomocí `find -perm`, `namei`, `auditctl` pomáhá odhalit bezpečnostní problémy.
- Pro základní přehled viz [07 - Bezpečná architektura](07-bezpecna-architektura.md).

---

➡️ [Zpět na přehled](README.md)
