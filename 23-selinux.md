# SELinux

> Security-Enhanced Linux

---

## Úvod

**SELinux** (Security-Enhanced Linux) je implementace mandatorního řízení přístupu (MAC) pro Linux. Projekt vznikl v roce 2000 v **NSA** (National Security Agency) jako sada patchů pro Linux kernel. V roce 2005 byl uvolněn jako open source a integrován do hlavní větve Linux kernelu (verze 2.6).

Zatímco klasické UNIXové **DAC** (Discretionary Access Control) umožňuje uživateli spravovat vlastní soubory (majitel nastavuje práva), **MAC** definuje centrální politiku, kterou nemůže uživatel obejít. SELinux vynucuje pravidla nezávisle na standardních právech rwx.

SELinux je postaven na frameworku **LSM** (Linux Security Modules), který umožňuje různým bezpečnostním modulům (SELinux, AppArmor, Smack) hookovat se do jádra. SELinux je jeden z LSM modulů, který je aktivní, pokud je povolen v kernelu a nakonfigurován v bootovacích parametrech.

```bash
# Kontrola, zda kernel podporuje SELinux
ls -l /sys/fs/selinux

# Pokud adresář existuje, SELinux je podporován a aktivní
# Výstup: dr-xr-xr-x. 1 root root 0 May 17 10:00 /sys/fs/selinux
```

SELinux je **výchozí** na distribucích RHEL, Fedora, CentOS a Rocky Linux. Na Debian a Ubuntu je dostupný, ale výchozím MAC frameworkem je AppArmor. Princip SELinuxu je jednoduchy: každý objekt (soubor, proces, socket, IPC) má bezpečnostní kontext a politika rozhoduje o kazdém přístupu na základě těchto kontextů.

```bash
# Instalace SELinuxu na Ubuntu (pozor, může kolidovat s AppArmor)
sudo apt install selinux-basics selinux-policy-default selinux-utils
# Po instalaci je nutný reboot a nastavení selinux=enforcing v /etc/default/grub
```

---

## 1. Režimy SELinuxu

SELinux pracuje ve trech rezimech, které urcují, jak se politika aplikuje:

### Enforcing

Politiky se vynucují. Přístup je blokován, pokud politika neuděluje výslovné povolení. Všechny deny se logují do audit logu. Toto je doporučený režim pro produkční prostředí.

### Permissive

Politiky se **ne**vynucují — porušení jsou pouze logována, ale neblokována. Tento režim je ideální pro ladění, migraci a identifikaci pravidel, která by bylo potřeba přidat.

### Disabled

SELinux je zcela vypnut. Pro přechod z disabled do enforcing (nebo naopak) je vyzadován reboot systému, protože se musí přenačíst bezpečnostní kontexty všech objektů v systému.

```bash
# Zjištění aktuálního režimu
getenforce
# Výstup: Enforcing  |  Permissive  |  Disabled

# Okamžitá zmena rezimu (do pristiho rebootu)
setenforce 0   # Permissive
setenforce 1   # Enforcing
# Poznámka: přepnutí z Disabled není možné za běhu
```

```bash
# Trvalá konfigurace v /etc/selinux/config
cat /etc/selinux/config
```

```ini
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing

# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

```bash
# Detailní výpis stavu SELinuxu
sestatus
# SELinux status:                 enabled
# Loaded policy name:             targeted
# Current mode:                   enforcing
# Mode from config file:          enforcing
# Max kernel policy version:      33
```

---

## 2. Kontexty (labels)

Kontext je základním stavebním kamenem SELinuxu. Každý objekt v systému (soubor, proces, uživatel, socket) má prirazen bezpecnostní kontext ve formátu:

```
user:role:type:level
```

Například: `unconfined_u:object_r:httpd_sys_content_t:s0`

### Složky kontextu

- **user** — SELinux uživatel (ne UNIX uživatel). Príklady: `unconfined_u`, `system_u`, `user_u`, `root`. Určuje, jaká omezení platí pro daného uživatele.
- **role** — role (RBAC — Role-Based Access Control). `object_r` pro soubory, `system_r` pro procesy, `unconfined_r` pro neomezené procesy.
- **type** — typ, nejdůležitější složka. Rozhoduje o přístupu na základě **Type Enforcement (TE)**. SELinux rozhoduje o přístupu porovnáním typu zdroje (proces) a typu cíle (soubor).
- **level** — úroveň pro MLS/MCS. `s0` je výchozí, vyšší hodnoty (s1, s2) jsou pro klasifikovaná data.

```bash
# Zobrazení kontextu souboru, procesu a uživatele
ls -Z /var/www/html/index.html
# unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
ps -Z | grep httpd
# system_u:system_r:httpd_t:s0   12345 ?  00:00:00 httpd
id -Z
# unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

```bash
# Zmena kontextu souboru (dočasně)
chcon -t httpd_sys_content_t /var/www/html/index.html
chcon -u system_u -r object_r -t httpd_sys_content_t /var/www/html/index.html
```

```bash
# Obnovení kontextu podle politiky
restorecon -v /var/www/html/index.html
# Výstup: Relabeled /var/www/html/index.html from unconfined_u:object_r:user_home_t:s0 to system_u:object_r:httpd_sys_content_t:s0
```

Kontexty jsou prirazovány automaticky podle pravidel v kontextových souborech. Například soubory v `/home/` dostanou typ `user_home_t`, webové soubory pod `/var/www/` dostanou `httpd_sys_content_t`. Pokud soubor zkopírujete z home do `/var/www/`, jeho kontext se nezmění — musíte spustit `restorecon`.

---

## 3. Politiky

SELinux politika definuje pravidla, která urcují, jaké operace mohou procesy vykonávat nad objekty. K dispozici jsou tri typy politik:

### Targeted (cílená)

Výchozí politika na RHEL/Fedora/CentOS. Chrání vybrané síťové služby (httpd, sshd, named, mysqld, postfix atd.) tím, že jim přiřazuje specifické typy (`httpd_t`, `sshd_t`). Ostatní procesy beží v doméně `unconfined_t` bez omezení. Tato politika poskytuje dobrý kompromis mezi bezpečností a použitelností.

### MLS (Multi-Level Security)

Striktní politika implementující víceúrovňovou bezpečnost. Každý objekt má úroveň utajení (Unclassified, Secret, Top Secret). Používá se hlavně v armádě, vládě a organizacích s přísnými bezpečnostními požadavky.

### Minimum

Minimalistická politika pro embedded zařízení a systémy s omezenými zdroji. Chrání pouze nejkritičtější subsystémy.

```bash
# Zjištění aktuálně načtené politiky
sestatus | grep "Loaded policy"
# Loaded policy name:             targeted

# Seznam načtených modulů politiky
semodule -l | head -5
# abrt       1.0.0
# accountsd  1.0.0
# ...
semodule -l | grep httpd
# httpd       2.0.0
semodule -l | wc -l
# (počet všech modulů)
```

```bash
# Adresářová struktura politiky
ls /etc/selinux/targeted/
# contexts/   modules/   policy/
```

---

## 4. Booleany

Booleany jsou přepínače v SELinux politice, které umožňují zapnout nebo vypnout specifická oprávnění bez nutnosti vytvářet novou politiku. Místo psaní vlastních pravidel stací nastavit boolean.

```bash
# Výpis všech booleanů
getsebool -a | head -5
# abrt_anon_write --> off
# antivirus_can_scan_system --> off
# auditadm_exec_content --> on
getsebool -a | wc -l  # celkový počet

# Zjištění konkrétního booleanu
getsebool httpd_enable_homedirs
# httpd_enable_homedirs --> off

# Dočasné / trvalé zapnutí
setsebool httpd_enable_homedirs on
setsebool -P httpd_can_network_connect on

# Výpis změněných booleanů
semanage boolean -l --extract
```

### Důležité booleany pro web server

| Boolean | Účel | Výchozí |
|---------|------|---------|
| `httpd_enable_homedirs` | Povolí httpd přístup k domovským adresářům | off |
| `httpd_can_network_connect` | Povolí httpd připojení k síti (např. k databázi) | off |
| `httpd_can_network_connect_db` | Povolí httpd připojení k databázovému portu | off |
| `httpd_can_sendmail` | Povolí httpd odesílat emaily | off |
| `httpd_use_nfs` | Povolí httpd přístup k NFS mountům | off |
| `httpd_read_user_content` | Povolí httpd čtení uživatelského obsahu | off |
| `ssh_chroot_rw` | Povolí zápis v chroot prostředí SSH | off |
| `ftp_home_dir` | Povolí FTP přístup k domovským adresářům | off |
| `samba_share_nfs` | Povolí Samba export NFS | off |

```bash
# Typické použití: web server potrebuje pristup k databázi
setsebool -P httpd_can_network_connect on
```

---

## 5. Audit a logování

SELinux loguje všechny deny (zamítnuté přístupy) do auditovacího subsystému. Pro ladění a troubleshooting je klíčové umět tyto zprávy najít a interpretovat.

```bash
# Hlavní zdroj SELinux zpráv
ls -l /var/log/audit/audit.log
# -rw-------. 1 root root 1234567 May 17 10:00 /var/log/audit/audit.log
```

Pokud `auditd` nebeží, zprávy padají do `/var/log/messages`:

```bash
# SELinux zprávy v messages
grep "SELinux" /var/log/messages | tail -5
```

```bash
# Hledání AVC denial v audit logu (posledních 5 minut)
ausearch -m avc -ts recent
# ----
# type=AVC msg=audit(1700000000.123:456): avc:  denied  { read } for  pid=12345
# comm="httpd" name="index.html" scontext=httpd_t tcontext=user_home_t tclass=file

# Hledání podle commandu nebo jen deny
ausearch -m avc -c httpd
ausearch -m avc --success no
```

```bash
# Vysvetlení, proc bylo blokováno
audit2why < /var/log/audit/audit.log
# Was caused by:
#     Missing type enforcement (TE) allow rule.
#     You can use audit2allow to generate a policy.

# Čitelnější výpis
audit2allow -w -a
# The file /var/www/html/index.html is not labeled with the httpd_sys_content_t type.
# You can use restorecon to fix the label.
```

---

## 6. audit2allow — vlastní politiky

Když SELinux blokuje legitimní operaci, je potřeba vytvořit vlastní politické pravidlo. K tomu slouží nástroj `audit2allow`, který analyzuje AVC denial zprávy a generuje odpovídající pravidla.

### Typický workflow

```bash
# 1. Prepnutí do permissive rezimu
setenforce 0

# 2. Reprodukce problému (např. spuštění webového skriptu)
curl http://localhost/myapp/cache

# 3. Vygenerování politiky ze všech denial záznamu
audit2allow -a -M myapp
# ************** IMPORTANT ***************
# To install the generated policy, run:
# semodule -i myapp.pp
```

Příkaz `audit2allow -a -M myapp` načte AVC denial z audit logu a vygeneruje `myapp.te` (textová politika) a `myapp.pp` (binární modul).

```bash
# Zobrazení vygenerované politiky
cat myapp.te
# module myapp 1.0;
# require {
#     type httpd_t;
#     type httpd_sys_content_t;
#     class file { write create unlink };
# }
# allow httpd_t httpd_sys_content_t:file { write create unlink };

# Instalace / odebrání modulu
semodule -i myapp.pp
semodule -r myapp
```

### Manuální kompilace (kdyz audit2allow neni k dispozici)

```bash
# Kompilace textové politiky do modulu
checkmodule -M -m myapp.te -o myapp.mod
semodule_package -m myapp.mod -o myapp.pp
semodule -i myapp.pp
```

### Praktický příklad: povolení zápisu pro httpd

```bash
# Web server potrebuje zapisovat do /var/www/html/data
mkdir /var/www/html/data
setenforce 0
echo "test" > /var/www/html/data/test.txt  # provedeno pres httpd
audit2allow -a -M httpd_write_data
semodule -i httpd_write_data.pp
setenforce 1
```

---

## 7. semanage — správa kontextů

Nástroj `semanage` slouží k trvalé správě SELinux konfigurace: definování kontextů pro soubory, porty, uživatele a permissivní domény.

### Správa kontextů souborů

```bash
# Výpis všech definovaných kontextu souboru
semanage fcontext -l | head -20
# Výstup (zkráceno):
# /var/www(/.*)?                          all files     system_u:object_r:httpd_sys_content_t:s0
# /var/www/html(/.*)?                     all files     system_u:object_r:httpd_sys_content_t:s0
# /var/www/cgi-bin(/.*)?                  all files     system_u:object_r:httpd_sys_script_exec_t:s0
```

```bash
# Přidání nového kontextu pro adresář
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/data(/.*)?'

# Aplikace kontextu (prepsání existujících)
restorecon -Rv /var/www/html/data
```

### Správa portů

```bash
# Výpis portů a jejich SELinux typu
semanage port -l | head -5
# http_port_t     tcp  80, 81, 443, 488, 8008, 8009, 8443, 9000
# ssh_port_t      tcp  22
# mysqld_port_t   tcp  1186, 3306, 63132-63164

# Povolení / odebrání / modifikace portu
semanage port -a -t http_port_t -p tcp 8080
semanage port -d -t http_port_t -p tcp 8080
semanage port -m -t http_port_t -p tcp 8080
```

### Další správa

```bash
# Dočasné povolení cele domény
semanage permissive -a httpd_t
semanage permissive -d httpd_t
```

```bash
# Mapování UNIX uživatelu na SELinux
semanage login -l
# __default__   unconfined_u   s0-s0:c0.c1023
# root          unconfined_u   s0-s0:c0.c1023

semanage user -l
# guest_u       user  s0  s0          guest_r
# unconfined_u  user  s0  s0-s0:c0.c1023  system_r unconfined_r
# user_u        user  s0  s0          user_r
```

---

## 8. Troubleshooting SELinuxu

Při řešení problému se SELinuxem postupujte systematicky. Mnoho problémů má jednoduché řešení, pokud víte, co hledat.

### Služba nebeží / selhává

```bash
# 1. Je to SELinux? Zkontrolujte AVC denial
ausearch -m avc -ts recent

# 2. Rychlý test — dočasné prepnutí do permissive
getenforce
setenforce 0
# Pokud problém zmizí, je to SELinux. Vratte do enforcing:
setenforce 1
```

### Postup pro identifikaci a řešení

```bash
# Krok 1: Rychlý prehled v messages
grep "SELinux" /var/log/messages | tail -10

# Krok 2: Detailní AVC denial
ausearch -m avc -ts recent

# Krok 3: Vysvetlení
audit2why < /var/log/audit/audit.log

# Krok 4: Automatické generování pravidla
audit2allow -a -M myapp_fix && semodule -i myapp_fix.pp
```

### Caste problémy a jejich řešení

| Problém | Pravděpodobná příčina | Řešení |
|---------|----------------------|--------|
| Web server hlásí 403 Forbidden | Špatný kontext souboru | `restorecon -Rv /var/www/html/` |
| Služba nejde spustit | Chybějící boolean | `setsebool -P ... on` |
| Aplikace nemůže zapisovat | Špatný typ kontextu | `semanage fcontext` + `restorecon` |
| Bind na port selže | Port není v politice | `semanage port -a -t ..._port_t -p tcp PORT` |
| Aplikace nemá síťový přístup | `httpd_can_network_connect` je off | `setsebool -P httpd_can_network_connect on` |
| NFS share nejde mountnout | Chybí boolean pro NFS | `setsebool -P httpd_use_nfs on` |

### Důležité pravidlo

**Nikdy nevypínejte SELinux** trvalým nastavením `SELINUX=disabled` v `/etc/selinux/config` jen proto, že vám neco nefunguje. Vždy problém identifikujte a vyreste budováním vlastní politiky nebo nastavením booleanu. Vypnutí SELinuxu vytváří falešný pocit bezpečí a oslabuje systém.

---

## 9. Porovnání s AppArmor

SELinux a AppArmor jsou dva nejrozšířenější MAC frameworky pro Linux. Oba řeší stejný problém — mandatorní řízení přístupu — ale liší se filozofií, složitostí a cílovou skupinou.

| Vlastnost | SELinux | AppArmor |
|-----------|---------|----------|
| Autor | NSA | Canonical (Cisco, Novell) |
| Mechanismus | Label-based (kontexty) | Path-based (cesty) |
| Politiky | Komplexní, TE + RBAC + MLS | Jednodušší profily |
| Konfigurace | Složitější (mnoho nástrojů) | Jednodušší (profily v textu) |
| Distribuce | RHEL/Fedora/CentOS | Ubuntu/Debian/SUSE |
| Default politika | Targeted (omezuje služby) | LXC profily pro základní služby |
| Logování | `/var/log/audit/audit.log` | `/var/log/syslog` |
| Vlastní pravidla | audit2allow + checkmodule | Ruční psaní profilů |
| Booleany | Ano (100+) | Ne (není koncept) |
| Kontexty | user:role:type:level | Pouze cesta k souboru |
| Výkon | Mírná režie (label checking) | Nižší režie |

```bash
# Zjištění, který MAC framework je aktivní
if [ -d /sys/fs/selinux ]; then
    echo "SELinux je aktivní ($(getenforce))"
elif [ -d /sys/kernel/security/apparmor ]; then
    echo "AppArmor je aktivní"
else
    echo "Žádný MAC framework není aktivní"
fi
```

Oba frameworky řeší stejný problém (MAC), ale liší se filozofií. SELinux je komplexnější a flexibilnější, AppArmor jednodušší na údržbu. Volba závisí na distribuci a požadavcích na bezpečnost. Detailní popis AppArmor viz [kapitolu 15](15-apparmor.md).

---

## Shrnutí

SELinux je výkonný nástroj pro mandatorní řízení přístupu v Linuxu. Poskytuje granularitu na úrovni jednotlivých typů, rolí a úrovní, což umožňuje vytvářet velmi detailní bezpečnostní politiky. Následující tabulka shrnuje nejdůležitější příkazy:

| Kategorie | Příkaz | Popis |
|-----------|--------|-------|
| Stav | `getenforce` | Aktuální režim (Enforcing/Permissive/Disabled) |
| Stav | `sestatus` | Detailní stav SELinuxu |
| Režim | `setenforce 1\|0` | Dočasná změna režimu |
| Režim | `/etc/selinux/config` | Trvalá konfigurace režimu a politiky |
| Kontext | `ls -Z` / `ps -Z` | Zobrazení bezpečnostního kontextu |
| Kontext | `chcon -t TYPE soubor` | Změna kontextu souboru |
| Kontext | `restorecon -v soubor` | Obnovení kontextu podle politiky |
| Politika | `semodule -l` | Seznam načtených modulů |
| Politika | `semodule -i modul.pp` | Instalace nového modulu |
| Boolean | `getsebool -a` | Výpis všech booleanů |
| Boolean | `setsebool -P nazev on` | Trvalé zapnutí booleanu |
| Audit | `ausearch -m avc -ts recent` | Hledání AVC denial |
| Audit | `audit2why < /var/log/audit/audit.log` | Vysvětlení denial |
| Vlastní | `audit2allow -a -M myapp` | Generování politiky z logů |
| Vlastní | `semodule -i myapp.pp` | Instalace vygenerované politiky |
| Kontext | `semanage fcontext -l` | Správa kontextů souborů |
| Kontext | `semanage port -a -t http_port_t -p tcp PORT` | Povolení portu |
| Uživatel | `semanage login -l` | Mapování UNIX -> SELinux uživatelů |
| Uživatel | `semanage user -l` | Seznam SELinux uživatelů |
| Ladění | `semanage permissive -a domena_t` | Dočasné povolení domény |

> **POZNÁMKA:** Základy MAC a porovnání s DAC jsou v [kapitole 7 — Bezpečná architektura](07-bezpecna-architektura.md). AppArmor, alternativní MAC framework, je popsán v [kapitole 15](15-apparmor.md).

---

➡️ [Zpět na přehled](README.md)
