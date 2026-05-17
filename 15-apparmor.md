# AppArmor

> Mandatory Access Control (MAC) pro Linux

---

## Úvod

> **Platforma:** AppArmor je výchozí MAC framework na **Ubuntu** a **Debian**. Na Fedora/CentOS/RHEL se používá SELinux. Oba řeší stejný problém (omezení programu na nezbytné minimum), ale liší se přístupem — AppArmor je založen na cestách (path-based), SELinux na kontextech (label-based).

AppArmor (Application Armor) poskytuje **Mandatory Access Control (MAC)** — na rozdíl od standardních Linux oprávnění (DAC), kde uživatel rozhoduje o svých souborech, AppArmor omezuje programy podle předem definovaného profilu. I když je program spuštěn jako root, AppArmor mu může zakázat přístup k určitým souborům nebo operacím.

---

## 1. Základní správa AppArmor

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `aa-status` | Stav AppArmor (načtené profily, režimy) | `sudo aa-status` | `apparmor module is loaded.`<br>`12 profiles are loaded.`<br>`9 profiles are in enforce mode.` |
| `aa-enforce /cesta/k/profilu` | Přepne profil do režimu vynucení (enforce) | `sudo aa-enforce /etc/apparmor.d/usr.bin.firefox` | `Setting /etc/apparmor.d/usr.bin.firefox to enforce mode.` |
| `aa-complain /cesta/k/profilu` | Přepne profil do režimu nahlášení (complain — loguje, neblokuje) | `sudo aa-complain /etc/apparmor.d/usr.bin.firefox` | `Setting /etc/apparmor.d/usr.bin.firefox to complain mode.` |
| `aa-disable /cesta/k/profilu` | Zakáže profil | `sudo aa-disable /etc/apparmor.d/usr.bin.firefox` | `Disabling /etc/apparmor.d/usr.bin.firefox.` |
| `apparmor_parser -r` | Ručně načte profil | `sudo apparmor_parser -r /etc/apparmor.d/usr.bin.firefox` | (žádný výstup) |
| `apparmor_parser -R` | Odebere profil | `sudo apparmor_parser -R /etc/apparmor.d/usr.bin.firefox` | (žádný výstup) |

```bash
# Základní kontrola stavu
sudo aa-status
# apparmor module is loaded.
# 12 profiles are loaded.
#   9 profiles are in enforce mode.
#     - /usr/bin/man
#     - /usr/bin/firefox
#     - /sbin/dhclient
#   3 profiles are in complain mode.
# 10 processes have profiles defined.
#   10 processes are in enforce mode.
#     - /usr/bin/firefox (1234)

# Dočasné přepnutí do complain režimu (pro testování)
sudo aa-complain /etc/apparmor.d/usr.bin.firefox
# Po otestování zpět do enforce
sudo aa-enforce /etc/apparmor.d/usr.bin.firefox
```

---

## 2. Struktura profilů

Profily jsou uloženy v `/etc/apparmor.d/`. Název souboru odpovídá cestě k binárce s lomítky nahrazenými tečkami.

| Umístění | Popis |
|----------|-------|
| `/etc/apparmor.d/usr.bin.man` | Profil pro `/usr/bin/man` |
| `/etc/apparmor.d/usr.bin.firefox` | Profil pro Firefox |
| `/etc/apparmor.d/sbin.dhclient` | Profil pro DHCP klienta |
| `/etc/apparmor.d/abstractions/` | Sdílené abstrakce (base, X, networking) |
| `/etc/apparmor.d/tunables/` | Proměnné (home, global) |
| `/etc/apparmor.d/local/` | Lokální rozšíření profilů (nemění se při aktualizaci balíčku) |
| `/etc/apparmor.d/cache/` | Cache zkompilovaných profilů |

### Syntaxe profilu

```
# Soubor: /etc/apparmor.d/usr.bin.example
#include <tunables/global>

/profile/to/binary {
    #include <abstractions/base>
    #include <abstractions/bash>

    # Čtení a zápis do specifických souborů
    /etc/example.conf          r,
    /var/log/example.log       w,
    /var/lib/example/**        rw,
    /tmp/example-*             rw,

    # Spouštění podprocesů (s omezením)
    /bin/bash                  ix,        # inherit profile (stejná omezení)
    /usr/bin/grep              Cx -> grep, # přepnutí do profilu grep

    # Síťová oprávnění
    network inet stream,
    network inet dgram,

    # Schopnosti (capabilities)
    capability net_bind_service,

    # Odkaz na podprofil
    profile grep {
        #include <abstractions/base>
        /usr/bin/grep          r,
        /**                    r,
    }
}
```

### Jednotlivé direktivy

| Direktiva | Význam |
|-----------|--------|
| `r` | Read — čtení souboru |
| `w` | Write — zápis do souboru |
| `rw` | Read+Write — čtení i zápis |
| `m` | Memory map — mapování souboru do paměti |
| `k` | Lock — zamykání souboru |
| `l` | Link — vytváření symbolických linků |
| `ix` | inherit profile — program zdědí profil volajícího |
| `Px` | discrete profile — program běží s vlastním profilem |
| `Cx` | change profile — přepnutí do jiného profilu |
| `/** rw` | Rekurzivní přístup ke všem souborům |
| `/specific/file r,` | Přístup ke konkrétnímu souboru |
| `network *` | Povolení všech síťových operací |

---

## 3. Vytváření vlastních profilů

### aa-genprof — generování profilu

```bash
# Spuštění generování profilu pro program
sudo aa-genprof /usr/bin/myapp

# Průvodce se ptá na jednotlivé operace:
# (S)can system log pro nalezení přístupů
# (A)llow — povolit přístup
# (D)eny — zakázat přístup  
# (G)lob — povolit s wildcard (`*`)
# (N)ew — přidat novou pravidlo
# (F)inish — uložit profil

# Po dokončení: profil v /etc/apparmor.d/usr.bin.myapp
```

### aa-logprof — úprava profilu podle logů

```bash
# Po spuštění programu v complain režimu
sudo aa-complain /etc/apparmor.d/usr.bin.myapp
# Spustit program a provést všechny potřebné akce
./myapp --do-everything

# Z logů vygenerovat doplňující pravidla
sudo aa-logprof

# Pak přepnout do enforce režimu
sudo aa-enforce /etc/apparmor.d/usr.bin.myapp
```

---

## 4. Práce s profily — příklady

### Vytvoření profilu pro vlastní skript

```bash
# 1. Přepnutí do complain režimu pro sběr dat
sudo aa-complain /etc/apparmor.d/usr.bin.myscript

# 2. Spuštění skriptu
./myscript --all-operations

# 3. Analýza logů
sudo aa-logprof

# 4. Výsledný profil
sudo cat /etc/apparmor.d/usr.bin.myscript
# #include <tunables/global>
# /home/user/myscript {
#   #include <abstractions/base>
#   #include <abstractions/bash>
#   
#   /home/user/myscript          r,
#   /etc/config.ini              r,
#   /var/log/myscript.log        w,
#   /tmp/**                      rw,
#   /bin/bash                    ix,
# }

# 5. Aktivace
sudo aa-enforce /etc/apparmor.d/usr.bin.myscript
```

### Zakázání a znovupovolení

```bash
# Dočasné zakázání AppArmor (pro celý systém)
sudo systemctl stop apparmor
sudo systemctl start apparmor

# Zakázání jednoho profilu
sudo aa-disable /etc/apparmor.d/usr.bin.firefox

# Po restartu se profil znovu nenačte
# Pro trvalé odstranění smažte soubor profilu
sudo rm /etc/apparmor.d/usr.bin.firefox
```

---

## 5. Logování a troubleshooting

AppArmor loguje zamítnuté přístupy do systémového logu.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `aa-logprof` | Interaktivní analýza logů | `sudo aa-logprof` | (interaktivní průvodce) |
| `aa-notify` | Zobrazení AppArmor notifikací | `sudo aa-notify -p` | `1 event since 2025-01-15 10:00:00` |
| `journalctl -t apparmor` | AppArmor zprávy z journald | `journalctl -t apparmor --since today` | `DENIED ... profile=/usr/bin/firefox` |
| `dmesg | grep apparmor` | AppArmor zprávy z kernel logu | `dmesg | grep apparmor` | `apparmor: DENIED ... comm="firefox"` |
| `ausearch -m AVC` | Audit log (pokud je auditd aktivní) | `ausearch -m AVC -ts today` | `type=AVC msg=audit(1736900000.123:456): apparmor="DENIED"` |

### Typické chyby a řešení

```bash
# Program nedokáže otevřít soubor
# Log: apparmor="DENIED" operation="open" profile="/usr/bin/myapp"
#       name="/etc/myapp/config.ini" comm="myapp"
# Řešení: přidejte do profilu /etc/myapp/config.ini r,

# Program nedokáže vytvořit dočasný soubor
# Log: apparmor="DENIED" operation="create" profile="/usr/bin/myapp"
#       name="/tmp/myapp-abc123.tmp" comm="myapp"
# Řešení: přidejte /tmp/myapp-*.tmp rw,

# Rychlé řešení: dočasně přepnout do complain režimu
sudo aa-complain /etc/apparmor.d/usr.bin.myapp
# Po otestování spusťte aa-logprof a přepněte zpět
sudo aa-logprof
sudo aa-enforce /etc/apparmor.d/usr.bin.myapp
```

---

## 6. AppArmor vs SELinux

| Vlastnost | AppArmor | SELinux |
|-----------|----------|---------|
| **Princip** | Path-based (cesty k souborům) | Label-based (kontexty) |
| **Distribuce** | Ubuntu, Debian, OpenSUSE | Fedora, RHEL, CentOS |
| **Konfigurace** | Profily v textových souborech | Policy v modulárním jazyce |
| **Složitost** | Nižší — jednodušší syntaxe | Vyšší — strmější křivka učení |
| **Typické použití** | Omezení jednotlivých aplikací | Komplexní systémová politika |
| **Výchozí profily** | Jen pro vybrané aplikace | Vynuceno na celý systém |

```bash
# Kontrola, zda běží AppArmor nebo SELinux
aa-status 2>/dev/null && echo "AppArmor" || getenforce 2>/dev/null && echo "SELinux" || echo "MAC není aktivní"
```

---

## Shrnutí

- **AppArmor** poskytuje mandatorní řízení přístupu (MAC) na úrovni jednotlivých programů.
- **Profily** jsou uloženy v `/etc/apparmor.d/` a definují, k jakým souborům a operacím má program přístup.
- **Režimy:** `enforce` (blokuje) a `complain` (pouze loguje) — complain je vhodný pro testování nových profilů.
- **aa-genprof** a **aa-logprof** pomáhají vytvářet a ladit profily na základě skutečného chování programu.
- **Logování** — hledejte v `journalctl -t apparmor` nebo `dmesg | grep apparmor`.
- AppArmor je jednodušší než SELinux a vhodný pro běžné serverové i desktopové aplikace na Ubuntu/Debian.
- Pro základní přehled bezpečnostní architektury viz [07 - Bezpečná architektura](07-bezpecna-architektura.md).

---

➡️ [Zpět na přehled](README.md)
