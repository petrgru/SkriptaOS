# Správa logů a prostředí

> **Log and Environment Management**
> Praktický průvodce správou logů, systémových proměnných a prostředí operačního systému.

---

## Úvod

Správa logů a systémových proměnných je klíčová pro diagnostiku problémů, auditování činnosti a zajištění správného běhu aplikací. Tato kapitola se věnuje nástrojům a postupům pro práci s logy, konfiguraci systémového prostředí a monitorování systémových zdrojů.

## 8.1 Souborový systém a práva

### 8.1.1 Výpis oprávnění a vlastnictví

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `ls -la` | Detailní výpis práv, vlastníka a skupiny | `ls -la ~/.ssh` | `-rw------- 1 user user 1234 May 17 10:00 id_rsa` |
| `stat` | Kompletní metadata včetně časů přístupu | `stat /etc/shadow` | `Access: 2026-05-17 08:00:00.000000000 +0200` |
| `find` s parametry | Hledání souborů s podezřelými právy | `find / -perm -4000 2>/dev/null` | `/usr/bin/su /usr/bin/sudo ...` |

**SUID/SGID skenování** (soubory s nebezpečnými právy):

```bash
# Najít všechny SUID binárky
find / -type f -perm -4000 -ls 2>/dev/null

# Najít soubory bez vlastníka (orphaned)
find / -nouser -o -nogroup -ls 2>/dev/null

# Najít world-writable soubory v systemových adresářích
find /etc -type f -perm -o+w -ls 2>/dev/null
```

### 8.1.2 Access Control Lists (ACL)

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------|
| `getfacl` | Zobrazení ACL na souboru | `getfacl ~/.gnupg/private-keys-v1.d` | `user:alice:rwx` |
| `setfacl` | Nastavení ACL | `setfacl -m u:alice:--- confidential.txt` | Odebere přístup uživateli |

```bash
# Zkontrolovat ACL na citlivých souborech
getfacl /etc/shadow /etc/gshadow /etc/ssl/private/

# Rekurzivní kontrola ACL v domovském adresáři
find ~ -type f -exec getfacl {} \; 2>/dev/null | grep -B1 "user:.*:rwx"
```

### 8.1.3 Šifrované souborové systémy

| Nástroj | Účel | Příklad |
|---------|------|---------|
| LUKS | Šifrování celého disku/oddílu | `cryptsetup luksOpen /dev/sda1 cryptroot` |
| eCryptfs | Šifrování adresáře (např. ~/Private) | `ecryptfs-mount-private` |
| Veracrypt | Přenosný šifrovaný kontejner | `veracrypt -t -c volume.tc` |
| fscrypt | Šifrování na úrovni souborů (ext4) | `fscrypt encrypt /home/user` |

```bash
# Kontrola šifrování disku
lsblk -o NAME,TYPE,FSTYPE,MOUNTPOINT | grep -E "crypt|luks"

# Ověření použití fscrypt
ls -la /home/*/.fscrypt 2>/dev/null
```

---

## 8.2 Uživatelská data

### 8.2.1 SSH klíče a konfigurace

| Kontrola | Příkaz | Účel |
|----------|--------|------|
| Oprávnění SSH | `ls -la ~/.ssh/` | Klíče musí mít `600`, `~/.ssh` `700` |
| Povolené klíče | `cat ~/.ssh/authorized_keys` | Kdo má přístup k účtu |
| Známé hostitele | `cat ~/.ssh/known_hosts` | Které servery jsou uloženy |

```bash
# Rychlý audit SSH adresáře
stat -c "%a %n" ~/.ssh ~/.ssh/* 2>/dev/null

# Kontrola, jestli nějaký klíč nemá heslo (passphrase)
for key in ~/.ssh/id_*; do
  ssh-keygen -y -f "$key" 2>/dev/null && echo "OK: $key" || echo "BEZ HESLA: $key"
done
```

### 8.2.2 GPG klíče

| Koncept | Příkaz | Popis |
|---------|--------|-------|
| Seznam klíčů | `gpg -K` | Soukromé klíče |
| Seznam veřejných | `gpg -k` | Veřejné klíče |
| Expirace | `gpg --list-keys --with-expiry` | Platnost klíčů |

```bash
# Audit GPG klíčů včetně podrobností
gpg -K --keyid-format LONG
gpg --list-keys --with-fingerprint --with-expiry

# Smazání nepotřebných klíčů
gpg --delete-key "email@example.com"
gpg --delete-secret-key "email@example.com"
```

### 8.2.3 Browser a aplikace

| Umístění | Popis |
|----------|-------|
| `~/.mozilla/firefox/*.default/` | Firefox profily (hesla, cookies, historie) |
| `~/.config/chromium/` | Chrome/Chromium data |
| `~/.config/BraveSoftware/` | Brave browser |
| `~/.local/share/keyrings/` | GNOME/KDE keyring |
| `~/.config/` | Aplikace třetích stran |

```bash
# Velikost browser cache
du -sh ~/.cache/{mozilla,chromium,brave} 2>/dev/null

# Seznam uložených hesel (keyring)
ls -la ~/.local/share/keyrings/
```

---

## 8.3 Síť a DNS

### 8.3.1 Monitorování síťového provozu

| Nástroj | Účel | Příklad |
|---------|------|---------|
| `tcpdump` | Zachytávání paketů | `tcpdump -i eth0 -nn port 80` |
| `tshark` | CLI verze Wiresharku | `tshark -i eth0 -T fields -e ip.src -e ip.dst` |
| `nethogs` | Sledování procesů dle síťové aktivity | `nethogs eth0` |
| `iftop` | Reálný přenos dle hostitele | `iftop -i eth0` |

```bash
# Zachytit DNS dotazy na rozhraní
sudo tcpdump -i any -nn port 53 2>/dev/null

# Sledovat spojení na konkrétní port
sudo tcpdump -i any -nn "tcp port 443 and host suspicious.example.com"

# Zjistit, který proces komunikuje (tshark + lsof)
sudo lsof -i :443
```

### 8.3.2 DNS Leak Test

```bash
# Zjistit aktuální DNS server
resolvectl status
# nebo
nmcli device show | grep DNS

# Otestovat DNS leak při použití VPN
# Porovnat veřejnou IP před a po připojení VPN
curl -s ifconfig.me
curl -s https://ipleak.net/json/

# Rychlý test DNS přes různé resolvery
dig +short whoami.akamai.net. @resolver1.opendns.com
dig +short myip.opendns.com @resolver1.opendns.com
```

### 8.3.3 VPN, Tor a Proxy

| Nástroj | Účel | Kontrola funkčnosti |
|---------|------|---------------------|
| WireGuard | Moderní VPN | `wg show` |
| OpenVPN | Tradiční VPN | `systemctl status openvpn@client` |
| Tor | Anonymizační síť | `curl --socks5 localhost:9050 ifconfig.me` |
| Proxy (HTTP/SOCKS) | Tunelování | `export http_proxy=http://proxy:8080` |

```bash
# Ověřit, že VPN skutečně mění IP
echo "Bez VPN:"
curl -s ifconfig.me
echo "S VPN:"
curl -s --interface wg0 ifconfig.me 2>/dev/null

# Kontrola Tor
curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org/api

# Zjistit únik IPv6 při použití VPN
curl -6 -s ifconfig.me 2>/dev/null || echo "IPv6 nedostupne (OK)"
```

### 8.3.4 Firewall

| Nástroj | Popis | Příkaz |
|---------|-------|--------|
| `ufw` | Jednoduchý firewall | `ufw status verbose` |
| `iptables` / `nftables` | Pokročilý firewall | `iptables -L -n -v` |
| `nft` | Moderní nftables | `nft list ruleset` |

```bash
# UFW: povolit jen nezbytné porty
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
sudo ufw status numbered

# iptables: zobrazit všechna pravidla
sudo iptables -L -n -v --line-numbers
sudo ip6tables -L -n -v

# Blokování nežádoucích IP
sudo iptables -A INPUT -s 10.0.0.0/8 -j DROP
```

---

## 8.4 Systém a logy

### 8.4.1 Bash historie

| Koncept | Příkaz | Popis |
|---------|--------|-------|
| Zobrazení historie | `history` | Celá historie příkazů |
| Velikost historie | `echo $HISTSIZE` | Počet uložených příkazů |
| Soubor historie | `ls -la ~/.bash_history` | Kde se historie fyzicky ukládá |

```bash
# Smazání historie (aktuální relace)
history -c
history -w

# Zabránění ukládání historie (dočasně)
set +o history
# Nebo smazáním proměnné
unset HISTFILE

# Trvalé vypnutí historie (přidat do ~/.bashrc)
echo "export HISTSIZE=0" >> ~/.bashrc
echo "export HISTFILE=/dev/null" >> ~/.bashrc
```

### 8.4.2 Systemd Journal

| Koncept | Příkaz | Popis |
|---------|--------|-------|
| Všechny logy | `journalctl` | Kompletní systémové logy |
| Logy od bootu | `journalctl -b` | Pouze aktuální relace |
| Uživatelské logy | `journalctl --user` | Logy uživatelských služeb |
| Sledování v reálu | `journalctl -f` | Live stream |

```bash
# Zjistit, kolik místa logy zabírají
journalctl --disk-usage

# Smazat logy starší než 7 dní
sudo journalctl --vacuum-time=7d

# Omezit velikost logů (v /etc/systemd/journald.conf)
# SystemMaxUse=500M

# Vyhledat citlivá data v logu (např. hesla)
sudo journalctl | grep -i "password\|passwd\|secret" 2>/dev/null
```

### 8.4.3 auditd

| Koncept | Příkaz | Popis |
|---------|--------|-------|
| Status | `sudo auditctl -s` | Stav audit subsystému |
| Pravidla | `sudo auditctl -l` | Aktivní audit pravidla |
| Sledování přístupu | `sudo auditctl -w /etc/passwd -p wa -k passwd_watch` | Sledování změn |
| Výpis záznamů | `sudo ausearch -k passwd_watch` | Hledání v audit logu |

```bash
# Naslouchání přístupů k shadow souboru
sudo auditctl -w /etc/shadow -p rwxa -k shadow_access

# Zobrazení výsledků
sudo ausearch -k shadow_access --interpret | tail -20

# Sledování přístupu k SSH klíčům
sudo auditctl -w /home/ -p r -k home_read
```

### 8.4.4 Uživatelské relace

| Nástroj | Popis | Příklad výstupu |
|---------|-------|-----------------|
| `w` | Kdo je přihlášen a co dělá | `user pts/0 10:00 0.10s 0.01s bash` |
| `who` | Seznam přihlášených | `user tty7 2026-05-17 09:00 (:0)` |
| `last` | Historie přihlášení | `user pts/0 10:00 - 11:30 (01:30)` |
| `lastlog` | Kdy se kdo naposled přihlásil | `username port 192.168.1.5 Sun May 17` |
| `finger` | Detail o uživateli | `Login: user Name: ... Shell: /bin/bash` |

```bash
# Kdo je aktuálně přihlášen
w
who -u

# Historie přihlášení konkrétního uživatele
last user

# Nikdy nepřihlášení uživatelé
lastlog | grep -E "Never logged|** Never logged in**"

# Neúspěšné pokusy o přihlášení
sudo lastb | head -20
```

---

## 8.5 Monitorování

### 8.5.1 Síťové sockety a procesy

| Nástroj | Popis | Příklad |
|---------|-------|---------|
| `lsof -i` | Který proces otevřel který port | `lsof -i -P -n` |
| `ss -tulnp` | Moderní náhrada netstat | `ss -tulnp state listening` |
| `netstat -tulnp` | Klasický výpis socketů | `netstat -tulnp 2>/dev/null` |
| `fuser` | Který proces používá port | `fuser 80/tcp` |

```bash
# Všechny naslouchající porty
echo "=== ss ==="
ss -tulnp | column -t
echo ""
echo "=== lsof ==="
sudo lsof -i -P -n | grep LISTEN

# Který proces poslouchá na portu 80
sudo lsof -i :80
sudo fuser 80/tcp

# Neobvyklé služby naslouchající na externích IP
ss -tulnp | grep -E "\*:[0-9]+" | grep -v "127.0.0.1\|::1"
```

### 8.5.2 Běžící procesy

| Nástroj | Popis | Příkaz |
|---------|-------|--------|
| `ps aux` | Všechny procesy | `ps auxf` (stromová struktura) |
| `top` / `htop` | Interaktivní monitor | `htop` |
| `pstree` | Strom procesů | `pstree -p user` |
| `systemd-cgls` | CGroup strom | `systemd-cgls` |

```bash
# Procesy s otevřenými síťovými sockety
sudo lsof -i | awk 'NR==1 || !/^COMMAND/ {print $1, $2, $9}' | sort -u

# Detekce podezřelých procesů
ps -eo pid,ppid,cmd,etime | grep -E "/(tmp|dev/shm)/"

# Skryté procesy (porovnání /proc s výstupem ps)
comm -23 <(ls /proc | grep -E '^[0-9]+$' | sort) <(ps -eo pid | tail -n +2 | sort)
```

### 8.5.3 Privátní data v paměti

```bash
# Zjistit, které procesy mají mapované citlivé soubory
sudo lsof | grep -E "\.gnupg|\.ssh|keyring"

# Dump procesu (nebezpečný, kontrolovat jen vlastní)
# sudo gcore <PID> 2>/dev/null
```

---

## 8.6 Nástroje pro mazání stop

### 8.6.1 Bezpečné mazání

| Nástroj | Popis | Příklad |
|---------|-------|---------|
| `shred` | Přepis souboru náhodnými daty | `shred -uzv secret.txt` |
| `wipe` | Vícenásobný přepis | `wipe -r sensitive_dir/` |
| `dd` | Přepis zařízení | `dd if=/dev/urandom of=/dev/sda bs=4M` |
| `srm` | Secure remove (z balíku secure-delete) | `srm -vz file` |

```bash
# Bezpečné smazání souboru (3 průchody + vynulování + smazání)
shred -n 3 -z -u -v confidential.pdf

# Bezpečné smazání celého adresáře
find sensitive_dir/ -type f -exec shred -n 1 -u {} \;

# Smazání volného místa na disku (velmi pomalé)
dd if=/dev/urandom of=delete.me bs=1M status=progress; rm delete.me
```

### 8.6.2 BleachBit (systémové čištění)

```bash
# Instalace
sudo apt install bleachbit

# CLI čištění (bez GUI)
bleachbit --clean system.cache system.tmp system.trash

# Seznam všech čističů
bleachbit --list-cleaners

# Simulace (dry run)
bleachbit --preview system.cache
```

### 8.6.3 Ruční čištění stop

```bash
# Smazání cache aplikací
rm -rf ~/.cache/*

# Smazání dočasných souborů
rm -rf /tmp/* 2>/dev/null

# Smazání historie prohlížeče (Firefox)
rm -rf ~/.mozilla/firefox/*.default/{cookies.sqlite,places.sqlite,formhistory.sqlite}

# Smazání clipboard historie
echo -n "" | xclip -selection clipboard

# Smazání nedávných dokumentů
rm -rf ~/.local/share/recently-used.xbel
```

---

## 8.7 Praktické scénáře

### Scénář 1: Kontrola otevřených portů a síťové bezpečnosti

```bash
#!/bin/bash
echo "=== NASLOUCHAJICI PORTY ==="
ss -tulnp | grep LISTEN

echo ""
echo "=== PROCESY NA SITI ==="
sudo lsof -i -P -n | grep -v "^COMMAND"

echo ""
echo "=== FIREWALL STATUS ==="
sudo ufw status verbose 2>/dev/null || sudo iptables -L -n --line-numbers

echo ""
echo "=== NEZNAMA SPOJENI ==="
ss -tnp state established | grep -v "127.0.0.1\|::1:" | grep -v "192.168."
```

### Scénář 2: Audit přístupových práv

```bash
#!/bin/bash
echo "=== SOUBORY S SUID/SGID ==="
find / -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null

echo ""
echo "=== WORLD-WRITABLE V /etc ==="
find /etc -type f -perm -o+w -ls 2>/dev/null

echo ""
echo "=== SOUBORY BEZ VLASTNIKA ==="
find / -nouser -o -nogroup -ls 2>/dev/null

echo ""
echo "=== OTEVRENE SSH KLICE ==="
find ~/.ssh -type f -perm /o+rwx -ls 2>/dev/null
```

### Scénář 3: Úplné smazání stop po práci

```bash
#!/bin/bash
echo "=== MAZANI HISTORIE ==="
history -c
history -w
rm -f ~/.bash_history

echo ""
echo "=== MAZANI CACHE ==="
rm -rf ~/.cache/*
rm -rf ~/.local/share/Trash/*

echo ""
echo "=== MAZANI LOGU (pouze uzivatelsky) ==="
rm -f ~/.xsession-errors
rm -f ~/.local/share/xorg/Xorg.0.log

echo ""
echo "=== MAZANI DOGASNYCH SOUBORU ==="
find /tmp -user "$USER" -delete 2>/dev/null

echo ""
echo "=== HOTOVO ==="
```

### Scénář 4: Test úniku dat přes síť

```bash
#!/bin/bash
echo "=== TEST 1: DNS Leak ==="
echo "Aktualni DNS:"
resolvectl status 2>/dev/null | grep "DNS Servers" || nmcli device show | grep DNS

echo ""
echo "=== TEST 2: Co vidi server ==="
curl -s https://ifconfig.me/all 2>/dev/null || curl -s ifconfig.me

echo ""
echo "=== TEST 3: IPv6 unik ==="
curl -6 -s https://ifconfig.me 2>/dev/null && echo "POZOR: IPv6 unika!" || echo "IPv6 OK (nedostupne)"

echo ""
echo "=== TEST 4: WebRTC (v prohlizeci) ==="
echo "WebRTC leak lze testovat na: https://browserleaks.com/webrtc"
```

---

## Shrnutí

| Oblast | Klíčové nástroje | Co kontrolovat |
|--------|------------------|----------------|
| Souborový systém | `ls -la`, `stat`, `find`, `getfacl` | Práva SUID/SGID, ACL, world-writable, orphaned soubory |
| Uživatelská data | `gpg -K`, `ssh-keygen`, `ls` | Oprávnění SSH, expirace GPG, browser cache |
| Síť | `tcpdump`, `ss`, `lsof`, `ufw` | Otevřené porty, DNS leak, firewall, Tor/VPN funkčnost |
| Logování | `journalctl`, `auditctl`, `ausearch` | History, logy, audit přístupů, přihlašovací záznamy |
| Monitorování | `ss -tulnp`, `lsof -i`, `ps aux` | Naslouchající služby, podezřelé procesy, skryté PID |
| Čištění | `shred`, `wipe`, `bleachbit` | Bezpečné mazání, cache, clipboard, temporary files |

**Zlatá pravidla soukromí:**

1. **Minimalizace** — neukládat citlivá data déle, než je nutné
2. **Šifrování** — všechna data v klidu (LUKS) i při přenosu (SSH, TLS, VPN)
3. **Audit** — pravidelně kontrolovat oprávnění, logy a otevřené porty
4. **Izolace** — používat Tor pro anonymitu, VPN pro soukromí, kontejnery pro aplikace
5. **Mazání** — mazat bezpečně (shred), ne jen `rm` (ten pouze odstraní odkaz)

---

> **Další čtení:**
> - `man 7 hier` — struktura souborového systému
> - `man 8 auditd` — audit subsystém
> - `man 8 tcpdump` — síťová analýza
> - Arch Wiki: [Security](https://wiki.archlinux.org/title/Security)
> - Privacy Guides: [https://www.privacyguides.org/](https://www.privacyguides.org/)

---

➡️ [Zpět na přehled](README.md)
