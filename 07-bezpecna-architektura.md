# Bezpečná architektura OS

## Úvod

Tato sekce pokrývá bezpečnostní prvky operačních systémů, od oprávnění souborů přes síťovou ochranu až po šifrování. Každý koncept je popsán s příkladem a typickým výstupem. Popisy jsou v češtině, příkazy a příklady v angličtině.

---

## Souborová bezpečnost

### Práva souborů (chmod, chown, chattr)

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `chmod` | Změna práv souboru (symbolicky) | `chmod u+x script.sh` | (žádný výstup) |
| `chmod` | Změna práv souboru (oktálově) | `chmod 755 script.sh` | (žádný výstup) |
| `chown` | Změna vlastníka a skupiny | `chown user:admin file.txt` | (žádný výstup) |
| `chattr` | Změna atributů souboru (immutable, append-only) | `chattr +i /etc/passwd` | (žádný výstup) |
| `lsattr` | Zobrazení atributů souboru | `lsattr /etc/passwd` | `----i--------e-- /etc/passwd` |

```bash
# Symbolická práva
chmod u=rwx,g=rx,o=r script.sh
# Výsledek: -rwxr-xr-x

# Oktálová práva
chmod 640 secret.txt
# rw- r-- ---  (vlastník: čtení+zápis, skupina: čtení, ostatní: nic)

# Změna vlastníka
chown alice:developers project/
# Rekurzivně
chown -R alice:developers project/

# Neodstranitelný soubor (ani root nemůže smazat bez -i)
chattr +i important.conf
rm important.conf
# rm: cannot remove 'important.conf': Operation not permitted

# Append-only log (lze jen přidávat)
chattr +a /var/log/app.log
```

### ACL (Access Control Lists)

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `getfacl` | Zobrazení ACL souboru | `getfacl file.txt` | `# file: file.txt`<br>`# owner: alice`<br>`# group: admin`<br>`user::rw-`<br>`user:bob:r--`<br>`group::r--`<br>`mask::rw-`<br>`other::---` |
| `setfacl` | Nastavení ACL souboru | `setfacl -m u:bob:rw secret.txt` | (žádný výstup) |
| `setfacl -x` | Odebrání ACL položky | `setfacl -x u:bob secret.txt` | (žádný výstup) |
| `setfacl -R` | Rekurzivní nastavení ACL | `setfacl -R -m g:dev:rx /opt/app` | (žádný výstup) |

```bash
# Přidání přístupu pro konkrétního uživatele
setfacl -m u:john:rwx /shared/project
getfacl /shared/project
# # file: /shared/project
# user::rwx
# user:john:rwx
# group::r-x
# mask::rwx
# other::---

# Výchozí ACL pro nové soubory v adresáři
setfacl -m d:g:dev:rwx /shared/project
# Nové soubory automaticky zdědí oprávnění pro skupinu dev

# Odebrání ACL
setfacl -b file.txt                          # smaže všechny ACL
setfacl -x g:dev /shared/project             # smaže jen jednu položku
```

### umask, SUID, SGID, Sticky bit

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `umask` | Výchozí maska oprávnění pro nové soubory | `umask 022` | `0022` |
| SUID | Spuštění souboru s právy vlastníka | `chmod u+s /usr/bin/passwd` | `-rwsr-xr-x` |
| SGID | Spuštění s právy skupiny (nebo dědičnost skupiny u adresářů) | `chmod g+s /shared` | `drwxrws---` |
| Sticky bit | Omezení mazání na vlastníka souboru (typicky /tmp) | `chmod +t /tmp` | `drwxrwxrwt` |

```bash
# umask - určuje, která práva budou ODMAZÁNA
umask 077   # nové soubory: 600, nové adresáře: 700 (pouze vlastník)
umask 022   # nové soubory: 644, nové adresáře: 755 (výchozí)
umask 002   # nové soubory: 664, nové adresáře: 775 (skupinová spolupráce)

# Výpočet: 666 - 022 = 644 pro soubory, 777 - 022 = 755 pro adresáře

# SUID - program běží s právy vlastníka (třeba passwd potřebuje root)
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root 59976 Jan 10  2024 /usr/bin/passwd
# 's' místo 'x' ve vlastníkovi = SUID

# SGID na adresáři - nové soubory dědí skupinu adresáře
mkdir /shared/lab
chown :students /shared/lab
chmod g+s /shared/lab
touch /shared/lab/newfile.txt
ls -l /shared/lab/newfile.txt
# -rw-r--r-- 1 alice students 0 May 17 10:00 newfile.txt
# (skupina je 'students', ne 'alice')

# Sticky bit - pouze vlastník může smazat svůj soubor
ls -ld /tmp
# drwxrwxrwt 20 root root 4096 May 17 10:00 /tmp
# 't' místo 'x' u ostatních = sticky bit
```

---

## Uživatelé a skupiny

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `useradd` | Vytvoření nového uživatele | `useradd -m -G sudo alice` | (žádný výstup) |
| `usermod` | Úprava existujícího uživatele | `usermod -aG docker alice` | (žádný výstup) |
| `userdel` | Smazání uživatele | `userdel -r alice` | (žádný výstup) |
| `groupadd` | Vytvoření nové skupiny | `groupadd developers` | (žádný výstup) |
| `passwd` | Změna hesla uživatele | `passwd alice` | `New password:`<br>`Retype new password:`<br>`passwd: password updated successfully` |
| `id` | Zobrazení UID, GID a skupin | `id alice` | `uid=1001(alice) gid=1001(alice) groups=1001(alice),4(adm),27(sudo)` |
| `groups` | Zobrazení skupin uživatele | `groups alice` | `alice : alice adm sudo docker` |

### Konfigurační soubory

| Soubor | Popis | Příklad obsahu |
|--------|-------|----------------|
| `/etc/passwd` | Databáze uživatelů (veřejná) | `alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash` |
| `/etc/shadow` | Hashovaná hesla (přístup pouze root) | `alice:$y$j9T$...hash...:19876:0:99999:7:::` |
| `/etc/group` | Databáze skupin | `developers:x:2001:bob,charlie` |
| `/etc/sudoers` | Pravidla pro sudo | `alice ALL=(ALL:ALL) ALL` |
| `/etc/login.defs` | Výchozí nastavení účtů | `PASS_MAX_DAYS 99999`<br>`UID_MIN 1000` |

```bash
# Vytvoření uživatele s domovským adresářem a specifikací shellu
useradd -m -s /bin/bash -c "Alice Smith" alice
passwd alice

# Přidání uživatele do skupin
usermod -aG sudo,developers,docker alice

# Vytvoření skupiny a nastavení GID
groupadd -g 2001 developers

# /etc/passwd - struktura polí
# username:password:UID:GID:comment:home:shell
grep alice /etc/passwd
# alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash

# /etc/shadow - struktura polí
# username:hash:lastchange:min:max:warn:inactive:expire
sudo grep alice /etc/shadow
# alice:$y$j9T$xyz...hash...abc:19876:0:99999:7:::

# SUDOERS - přidání uživatele s plnými právy
echo "alice ALL=(ALL:ALL) ALL" >> /etc/sudoers
# nebo bezpečněji:
echo "alice ALL=(ALL:ALL) ALL" | sudo tee -a /etc/sudoers

# SUDOERS - skupina s omezením na konkrétní příkazy
echo "%developers ALL=(ALL) /usr/bin/systemctl, /usr/bin/journalctl" >> /etc/sudoers

# Ověření sudo přístupu
sudo -l -U alice
# User alice may run the following commands on host:
#     (ALL : ALL) ALL
```

---

## Síťová bezpečnost

> **Firewall** — Podrobný popis najdete v samostatné [kapitole 24 (Firewall)](24-firewall.md). Tato část obsahuje jen stručný přehled nástrojů.

Linux nabízí několik nástrojů pro správu firewallu: **iptables** (tradiční), **nftables** (moderní nástupce), **UFW** (uživatelsky přívětivý obal pro Ubuntu) a **firewalld** (RHEL/Fedora). Každý nástroj pokrývá stejnou funkcionalitu — filtrování paketů, NAT, ochranu proti útokům — liší se syntaxí a filozofií.

### TCP wrappers, SELinux, AppArmor

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| TCP wrappers | Povolení/zakázání přístupu k službám podle IP | `/etc/hosts.allow`<br>`/etc/hosts.deny` | `sshd: 192.168.1.`<br>`ALL: ALL` |
| `getenforce` | Stav SELinuxu (Enforcing/Permissive/Disabled) | `getenforce` | `Enforcing` |
| `setenforce` | Dočasná změna SELinux módu | `setenforce 0` | (dočasně Permissive) |
| `sestatus` | Detailní informace o SELinuxu | `sestatus` | `SELinux status: enabled`<br>`Current mode: enforcing` |
| `aa-status` | Stav AppArmor profilů | `aa-status` | `apparmor module is loaded.`<br>`28 profiles are loaded.` |

```bash
# TCP wrappers - povolení SSH jen z lokální sítě
echo "sshd: 192.168.1." >> /etc/hosts.allow
echo "ALL: ALL" >> /etc/hosts.deny

# SELinux - kontext a troubleshooting
ls -Z /var/www/html/index.html
# system_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html

# Změna SELinux kontextu
chcon -t httpd_sys_content_t /var/www/html/newpage.html
# nebo permanentně:
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html

# SELinux boolean - povolení HTTPD skriptů
getsebool httpd_enable_cgi
setsebool httpd_enable_cgi on

# SELinux log - při denied přístupu
grep "AVC" /var/log/audit/audit.log
# type=AVC msg=audit(1712345678.123:456): avc:  denied  { read } for  pid=1234

# AppArmor (Debian/Ubuntu)
aa-status
# apparmor module is loaded.
# 28 profiles are loaded.
# 28 profiles are in enforce mode.
#    /usr/sbin/ntpd
#    /usr/sbin/sshd

# Dočasné vypnutí profilu pro debugging
aa-complain /usr/sbin/sshd   # loguje, ale neblokuje
aa-enforce /usr/sbin/sshd    # znovu blokuje
```

---

## Systémová bezpečnost

### fail2ban, auditd, syslog, journalctl

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `fail2ban` | Blokování IP po opakovaných selháních | `fail2ban-client status sshd` | `Status for the jail: sshd`<br>`|- Filter`<br>`|  |- Currently failed: 3`<br>`|  |- Total failed: 42`<br>`|- Actions`<br>`   |- Currently banned: 2`<br>`   |- Total banned: 15` |
| `auditd` | Sledování systémových volání a souborů | `auditctl -w /etc/passwd -p wa` | (pravidlo přidáno) |
| `ausearch` | Vyhledávání v audit logu | `ausearch -f /etc/passwd` | `type=SYSCALL msg=audit(...): ...` |
| `journalctl` | Prohlížení systemd logů | `journalctl -u sshd` | `Jan 17 10:00:01 server sshd[1234]: Accepted publickey for alice` |
| `syslog` | Tradiční logovací systém | `tail -f /var/log/syslog` | `Jan 17 10:00:01 server kernel: [12345.678] ...` |

```bash
# fail2ban - konfigurace SSH ochrany
cat /etc/fail2ban/jail.local
# [sshd]
# enabled = true
# port = ssh
# filter = sshd
# logpath = /var/log/auth.log
# maxretry = 5
# bantime = 3600
# findtime = 600

# Restart a kontrola
systemctl restart fail2ban
fail2ban-client status
# Status
# |- Number of jail: 1
# |- Jail list: sshd

# auditd - sledování změn v /etc/passwd
auditctl -w /etc/passwd -p wa -k passwd_changes
auditctl -w /etc/shadow -p wa -k shadow_changes

# Vyhledání událostí
ausearch -k passwd_changes --start today
ausearch -k passwd_changes -ui alice

# Trvalá pravidla auditd
echo "-w /etc/passwd -p wa -k passwd_changes" >> /etc/audit/rules.d/security.rules
echo "-w /etc/sudoers -p wa -k sudoers_changes" >> /etc/audit/rules.d/security.rules
augenrules --load

# journalctl - praktické příklady
journalctl -u sshd --since "1 hour ago"
journalctl -p err -b                         # chyby od bootu
journalctl -k -f                             # kernel log v reálném čase
journalctl --disk-usage                      # velikost logů
# Archived and active journals take up 128.0M in the filesystem.

# syslog - klasické log soubory
tail -f /var/log/syslog      # systémové zprávy
tail -f /var/log/auth.log    # autentizace (Debian/Ubuntu)
tail -f /var/log/secure      # autentizace (RHEL/CentOS)
tail -f /var/log/kern.log    # kernel zprávy
```

### PAM (Pluggable Authentication Modules)

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `/etc/pam.d/` | Konfigurace PAM pro jednotlivé služby | `ls /etc/pam.d/` | `sshd login sudo passwd su common-auth` |
| `pam_tally2` | Omezení počtu pokusů o přihlášení | `pam_tally2 --user alice` | `Login Failures Latest failure From`<br>`alice 5 01/17/25 10:00:00 192.168.1.100` |
| `pam_unix.so` | Standardní unixová autentizace | (součást common-auth) | (heslo + shadow) |
| `pam_limits.so` | Omezení zdrojů pro uživatele | `/etc/security/limits.conf` | `alice hard nproc 100` |

```bash
# Příklad PAM konfigurace pro SSH (/etc/pam.d/sshd)
cat /etc/pam.d/sshd
# auth     required    pam_nologin.so
# auth     required    pam_securetty.so
# auth     required    pam_env.so
# auth     required    pam_unix.so
# account  required    pam_nologin.so
# account  required    pam_unix.so
# password required    pam_unix.so
# session  required    pam_unix.so
# session  required    pam_limits.so

# Omezení opakovaných pokusů (prevence brute force)
echo "auth required pam_tally2.so deny=5 unlock_time=600" >> /etc/pam.d/sshd

# Omezení zdrojů v /etc/security/limits.conf
# <domain> <type> <item> <value>
alice       hard    nproc   100        # max 100 procesu pro alice
@developers hard    nofile   2048      # max 2048 otevrenych souboru
*           soft    core     0         # zakazat core dump vsem
```

### Kernel hardening (sysctl)

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `sysctl` | Čtení/nastavení kernel parametrů za běhu | `sysctl net.ipv4.tcp_syncookies` | `net.ipv4.tcp_syncookies = 1` |
| `sysctl -w` | Dočasné nastavení parametru | `sysctl -w net.ipv4.ip_forward=0` | `net.ipv4.ip_forward = 0` |
| `sysctl -p` | Načtení parametrů ze souboru | `sysctl -p /etc/sysctl.d/99-hardening.conf` | (načtena všechna nastavení) |

```bash
# Základní kernel hardening - /etc/sysctl.d/99-hardening.conf
cat /etc/sysctl.d/99-hardening.conf
# === Síťová bezpečnost ===
# Zakázat IP forwarding (prevence route mezi sítěmi)
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# Ochrana proti SYN flood útokům
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_syn_retries = 5

# Ochrana proti spoofingu
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Neodpovídat na ICMP redirect
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

# Zakázat source routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# === Obecná bezpečnost ===
# Ochrana proti symlink a hardlink útokům
fs.protected_symlinks = 1
fs.protected_hardlinks = 1

# Ochrana proti útokům na /proc
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
kernel.printk = 3 3 3 3

# Omezit core dump
fs.suid_dumpable = 0

# === Ostatní ===
# Zvýšit limit pro připojení
net.core.somaxconn = 1024

# Aplikace nastavení
sysctl -p /etc/sysctl.d/99-hardening.conf
```

---

## Šifrování

### GPG (GNU Privacy Guard)

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `gpg --full-gen` | Vytvoření nového GPG klíče | `gpg --full-generate-key` | (interaktivní průvodce) |
| `gpg --encrypt` | Zašifrování souboru | `gpg --encrypt --recipient alice@example.com secret.txt` | (vytvoří `secret.txt.gpg`) |
| `gpg --decrypt` | Dešifrování souboru | `gpg --decrypt secret.txt.gpg` | `Tajny obsah souboru` |
| `gpg --sign` | Podepsání souboru | `gpg --sign document.txt` | (vytvoří `document.txt.gpg`) |
| `gpg --verify` | Ověření podpisu | `gpg --verify document.txt.gpg` | `gpg: Good signature from "Alice Smith <alice@example.com>"` |
| `gpg --list-keys` | Seznam veřejných klíčů | `gpg --list-keys` | `pub rsa4096 2025-01-17 [SC]`<br>`      A1B2C3D4E5F6...`<br>`uid [ultimate] Alice Smith <alice@example.com>` |

```bash
# Export veřejného klíče pro sdílení
gpg --armor --export alice@example.com > alice-public.key
cat alice-public.key
# -----BEGIN PGP PUBLIC KEY BLOCK-----
# mQINBGS3hDkBEAC...
# -----END PGP PUBLIC KEY BLOCK-----

# Import cizího veřejného klíče
gpg --import bob-public.key

# Šifrování symetrickým heslem (bez klíčů)
gpg --symmetric --cipher-algo AES256 secret.txt
# (zeptá se na heslo)

# Podepsání a šifrování zároveň
gpg --encrypt --sign --recipient bob@example.com report.pdf

# Dešifrování
gpg --output report-decrypted.pdf --decrypt report.pdf.gpg
```

### OpenSSL

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `openssl genrsa` | Generování RSA klíče | `openssl genrsa -out key.pem 2048` | `Generating RSA private key, 2048 bit long modulus` |
| `openssl req` | Vytvoření CSR (Certificate Signing Request) | `openssl req -new -key key.pem -out cert.csr` | (interaktivní nebo -subj) |
| `openssl x509` | Vytvoření self-signed certifikátu | `openssl req -x509 -days 365 -key key.pem -out cert.pem` | (vytvoří cert.pem) |
| `openssl enc` | Šifrování souboru | `openssl enc -aes-256-cbc -salt -in file.txt -out file.enc` | (vyžádá heslo) |
| `openssl s_client` | Testování SSL/TLS spojení | `openssl s_client -connect example.com:443` | `CONNECTED(00000003)`<br>`Certificate chain...` |

```bash
# Vytvoření self-signed certifikátu (jedním příkazem)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem \
  -days 365 -nodes -subj "/CN=server.example.com"

# Šifrování souboru AES-256-CBC
openssl enc -aes-256-cbc -salt -in confidential.pdf -out confidential.enc
# enter aes-256-cbc encryption password: ****
# Verifying - enter aes-256-cbc encryption password: ****

# Dešifrování
openssl enc -d -aes-256-cbc -in confidential.enc -out confidential.pdf
# enter aes-256-cbc decryption password: ****

# Zobrazení informací o certifikátu
openssl x509 -in cert.pem -text -noout | head -20
# Certificate:
#     Data:
#         Version: 3 (0x2)
#         Serial Number: ...
#         Issuer: CN = server.example.com
#         Validity
#             Not Before: Jan 17 10:00:00 2025 GMT
#             Not After : Jan 17 10:00:00 2026 GMT

# Otestování SSL na serveru
openssl s_client -connect google.com:443 -tls1_3
```

### LUKS (dm-crypt) - šifrování disku

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `cryptsetup luksFormat` | Inicializace LUKS oddílu | `cryptsetup luksFormat /dev/sdb1` | `WARNING! ... Are you sure? Type YES:` |
| `cryptsetup open` | Otevření (odemknutí) LUKS zařízení | `cryptsetup open /dev/sdb1 encrypted_volume` | `Enter passphrase for /dev/sdb1:` |
| `cryptsetup close` | Zavření (zamknutí) LUKS zařízení | `cryptsetup close encrypted_volume` | (žádný výstup) |
| `cryptsetup luksDump` | Zobrazení hlavičky LUKS | `cryptsetup luksDump /dev/sdb1` | `LUKS header information`<br>`Version: 2`<br>`Cipher: aes-xts-plain64` |

```bash
# Vytvoření šifrovaného oddílu
cryptsetup luksFormat /dev/sdb1
# WARNING!
# This will overwrite data on /dev/sdb1 irrevocably.
# Are you sure? Type uppercase YES: YES
# Enter passphrase for /dev/sdb1:
# Verify passphrase:

# Otevření a vytvoření filesystému
cryptsetup open /dev/sdb1 secret_data
mkfs.ext4 /dev/mapper/secret_data
mount /dev/mapper/secret_data /mnt/secret

# Práce se zařízením
df -h /mnt/secret
# /dev/mapper/secret_data  976M  2.5M  907M   1% /mnt/secret

# Zavření
umount /mnt/secret
cryptsetup close secret_data

# Zobrazení informací o LUKS
cryptsetup luksDump /dev/sdb1
# LUKS header information
# Version:        2
# Epoch:          3
# Metadata area:  16384 [bytes]
# UUID:           a1b2c3d4-e5f6-7890-abcd-ef1234567890
# Label:          (no label)
# Subsystem:      (no subsystem)
# Flags:          (no flags)
#
# Data segments:
#   0: crypt
#     offset: 16777216 [bytes]
#     length: (whole device)
#     cipher: aes-xts-plain64
#     sector: 512 [bytes]
#
# Keyslots:
#   0: luks2
#     Key:        512 bits
#     Priority:   normal
#     Cipher:     aes-xts-plain64

# Přidání dalšího klíče (např. pro backup)
cryptsetup luksAddKey /dev/sdb1
# Enter any existing passphrase:
# Enter new passphrase for key slot:
```

### SSH klíče

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `ssh-keygen` | Generování SSH klíčového páru | `ssh-keygen -t ed25519` | `Generating public/private ed25519 key pair.`<br>`Your identification has been saved in ~/.ssh/id_ed25519` |
| `ssh-copy-id` | Kopírování veřejného klíče na server | `ssh-copy-id user@server` | `Number of key(s) added: 1`<br>`Now try logging into the machine` |
| `ssh-add` | Přidání klíče do SSH agenta | `ssh-add ~/.ssh/id_ed25519` | `Identity added: id_ed25519 (alice@laptop)` |
| `ssh -i` | Připojení s konkrétním klíčem | `ssh -i ~/.ssh/project-key user@server` | (přihlášení na server) |

```bash
# Generování moderního Ed25519 klíče (doporučeno)
ssh-keygen -t ed25519 -C "alice@laptop"
# Generating public/private ed25519 key pair.
# Enter file in which to save the key (/home/alice/.ssh/id_ed25519):
# Enter passphrase (empty for no passphrase): ****
# Enter same passphrase again: ****
# Your identification has been saved in /home/alice/.ssh/id_ed25519
# Your public key has been saved in /home/alice/.ssh/id_ed25519.pub
# The key fingerprint is:
# SHA256:abc123... alice@laptop

# Nebo RSA (pro starší systémy)
ssh-keygen -t rsa -b 4096 -C "alice@laptop"

# Kopírování na server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@192.168.1.100
# /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "id_ed25519.pub"
# /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s)
# Number of key(s) added: 1
# Now try logging into the machine, with: "ssh 'user@192.168.1.100'"

# Kontrola otisku klíče
ssh-keygen -lf ~/.ssh/id_ed25519.pub
# 256 SHA256:abc123... alice@laptop (ED25519)

# Konfigurace SSH (zjednodušení připojení)
cat ~/.ssh/config
# Host myserver
#     HostName 192.168.1.100
#     User alice
#     Port 2222
#     IdentityFile ~/.ssh/project-key
#     ServerAliveInterval 60

# Poté stačí: ssh myserver
```

---

## Praktické příklady

### 1. SSH hardening

```bash
# /etc/ssh/sshd_config - bezpečná konfigurace
cat /etc/ssh/sshd_config

# Zakázat přihlášení root
PermitRootLogin no

# Pouze klíče (žádná hesla)
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no

# Pouze protokol 2
Protocol 2

# Omezit uživatele
AllowUsers alice bob
DenyUsers root guest

# Omezit skupiny
AllowGroups ssh-users

# Změnit port (proti automatizovaným skenům)
Port 2222

# Další hardening
MaxAuthTries 3
MaxSessions 5
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 60
PermitEmptyPasswords no
X11Forwarding no
AllowAgentForwarding no
TCPKeepAlive no

# Restart SSH služby
systemctl restart sshd

# Ověření, že SSH naslouchá na novém portu
ss -tlnp | grep 2222
# LISTEN 0 128 0.0.0.0:2222 0.0.0.0:* users:(("sshd",pid=1234,fd=3))
```

> Podrobný bezpečnostní audit včetně detekce rootkitů, kontroly integrity souborů
> a ověřování systémových balíčků je popsán v [kapitole 25](25-bezpecnostni-audit.md).

---

## Shrnutí

- **Souborová bezpečnost**: `chmod` pro základní práva, `setfacl` pro jemnější řízení přístupu, `chattr +i` pro ochranu proti smazání, SUID/SGID/sticky bit pro speciální případy.
- **Uživatelé a skupiny**: `useradd`/`usermod` pro správu uživatelů, `/etc/shadow` pro bezpečné ukládání hesel, `/etc/sudoers` pro delegování práv.
- **Síťová bezpečnost**: Firewall — podrobnosti v [kapitole 24](24-firewall.md), SELinux/AppArmor pro mandatory access control.
- **Systémová bezpečnost**: `fail2ban` blokuje brute force útoky, `auditd` sleduje změny, `sysctl` hardening chrání jádro, PAM řídí autentizaci.
- **Šifrování**: GPG pro šifrování souborů a emailů, OpenSSL pro TLS certifikáty, LUKS pro šifrování celého disku, SSH klíče pro bezpečné přihlašování.
- **Praktické tipy**: Vypněte root SSH přístup, používejte klíče místo hesel, pravidelně kontrolujte logy, aplikujte kernel hardening, zapněte auditování důležitých souborů.

---

➡️ [Zpět na přehled](README.md)
