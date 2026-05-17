# Práce s SSH

> Working with SSH – Secure Shell

---

## Úvod

> **Platforma:** SSH je standardní součástí všech Linuxových distribucí. Na Ubuntu/Debian je klient předinstalovaný, server se instaluje balíkem `openssh-server`.

**SSH (Secure Shell)** je protokol pro šifrovanou vzdálenou komunikaci mezi počítači. Umožňuje bezpečné přihlášení ke vzdálenému serveru, spouštění příkazů a přenos souborů přes nebezpečnou síť (např. internet).

SSH používá **klient-server model**:
- **SSH klient** (`ssh`) – program, který spouštíte na svém počítači pro připojení ke vzdálenému serveru.
- **SSH server** (`sshd`) – démon běžící na serveru, který naslouchá na portu 22 (výchozí) a zpracovává příchozí spojení.

Proč používat SSH:
- **Šifrovaná komunikace** – veškerý provoz je šifrovaný (nikdo nemůže odposlouchávat hesla ani data).
- **Autentizace veřejným klíčem** – místo hesla lze použít pár klíčů (soukromý + veřejný), což je bezpečnější a pohodlnější.
- **Tunelování** – přes SSH lze bezpečně přesměrovávat porty a protokoly, které samy o sobě šifrování nepodporují.

---

## .ssh/ adresář

Všechna uživatelská konfigurace SSH se ukládá do adresáře `~/.ssh/`. Tento adresář musí mít nastavena **restriktivní oprávnění**, jinak SSH klient odmítne pracovat.

| Soubor | Účel | Oprávnění |
|--------|------|-----------|
| `id_ed25519` | Soukromý klíč (Ed25519) – nikdy nesdílet! | `600` |
| `id_ed25519.pub` | Veřejný klíč (Ed25519) – lze sdílet | `644` |
| `id_rsa` | Soukromý klíč (RSA) | `600` |
| `id_rsa.pub` | Veřejný klíč (RSA) | `644` |
| `id_ecdsa` | Soukromý klíč (ECDSA) | `600` |
| `authorized_keys` | Veřejné klíče povolené pro přihlášení na tomto serveru | `600` |
| `known_hosts` | Fingerprinty známých serverů (automaticky plněno) | `644` |
| `config` | Uživatelská konfigurace pro jednotlivé hostitele | `600` |

> **ℹ️ POZNÁMKA:** Adresář `~/.ssh/` musí mít oprávnění `700` a soubory uvnitř `600`/`644`. Pokud mají soubory nebo adresář volnější oprávnění, SSH klient odmítne pracovat s hláškou `Bad permissions`.

```bash
# Kontrola a oprava oprávnění
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519 ~/.ssh/authorized_keys ~/.ssh/config
chmod 644 ~/.ssh/id_ed25519.pub ~/.ssh/known_hosts
```

---

## Generování SSH klíčů

Pro autentizaci pomocí klíčů je nejdříve potřeba vygenerovat pár: **soukromý klíč** (zůstává u vás, chrání se jako heslo) a **veřejný klíč** (nahrajete na server).

### Ed25519 (doporučeno)

Nejmodernější a nejrychlejší typ klíče. Kratší klíč při stejné bezpečnosti jako RSA 4096. Podporován na všech moderních systémech.

```bash
# Interaktivní generování (s dotazem na cestu a heslo)
ssh-keygen -t ed25519 -C "popis"

# Neinteraktivní generování (cesta + prázdné heslo)
ssh-keygen -t ed25519 -C "popis" -f ~/.ssh/klic -N ""
```

### RSA (kompatibilní, starší)

Univerzální typ podporovaný i na velmi starých systémech. Vyžaduje delší klíč (min. 4096 bitů) pro srovnatelnou bezpečnost.

```bash
ssh-keygen -t rsa -b 4096 -C "popis"
```

### ECDSA (alternativa)

Varianta založená na eliptických křivkách. Kratší klíč než RSA, ale ne všechny starší implementace jej podporují.

```bash
ssh-keygen -t ecdsa -b 256 -C "popis"
```

### Přepínače ssh-keygen

| Přepínač | Význam | Příklad |
|----------|--------|---------|
| `-t` | Typ klíče (ed25519, rsa, ecdsa) | `-t ed25519` |
| `-b` | Délka klíče v bitech | `-b 4096` |
| `-C` | Komentář (obvykle email) | `-C "alice@example.com"` |
| `-f` | Cesta k souboru | `-f ~/.ssh/server.key` |
| `-N` | Passphrase (prázdné = bez hesla) | `-N ""` |

### Význam passphrase

Passphrase je **heslo k soukromému klíči**. Pokud passphrase nastavíte, je soukromý klíč na disku zašifrovaný – bez znalosti hesla ho nelze použít, ani když někdo získá soubor. Bez passphrase stačí soubor ukrást a klíč lze použít okamžitě.

> **⚠️ VAROVÁNÍ:** Soukromý klíč je ekvivalentem hesla – nikdy ho nesdílejte, nepřeposílejte e-mailem a neukládejte do veřejných repozitářů. Vždy nastavte oprávnění `600`.

---

## Přenos veřejného klíče na server

Po vygenerování klíčů je potřeba nahrát **veřejný klíč** na server. Ten se uloží do souboru `~/.ssh/authorized_keys` na serveru.

### Pomocí ssh-copy-id (jednoduše)

```bash
ssh-copy-id uzivatel@server
```

Příkaz automaticky zkopíruje výchozí veřejný klíč (`~/.ssh/id_ed25519.pub` nebo `~/.ssh/id_rsa.pub`) na server a přidá ho do `~/.ssh/authorized_keys`. Vyžaduje heslo k účtu na serveru (při prvním přihlášení).

Pro jiný než výchozí klíč:
```bash
ssh-copy-id -i ~/.ssh/server.key.pub uzivatel@server
```

> **ℹ️ POZNÁMKA:** `ssh-copy-id` automaticky vytvoří adresář `~/.ssh/` a nastaví správná oprávnění (`700` pro adresář, `600` pro `authorized_keys`).

### Manuální přenos (když ssh-copy-id není k dispozici)

```bash
cat ~/.ssh/id_ed25519.pub | ssh uzivatel@server "cat >> ~/.ssh/authorized_keys"
```

Po manuálním přenosu je nutné nastavit oprávnění na serveru:
```bash
ssh uzivatel@server "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

---

## Zabezpečení SSH serveru (sshd_config)

Hlavní konfigurační soubor SSH serveru je `/etc/ssh/sshd_config`. Zde se nastavuje, kdo se smí připojit, jakým způsobem a na jakém portu.

### Záloha před úpravou

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

### Nejdůležitější direktivy

| Direktiva | Doporučená hodnota | Význam |
|-----------|-------------------|--------|
| `Port` | `2222` (nebo jiný) | Změna portu proti automatickým skenům |
| `PermitRootLogin` | `prohibit-password` | Root jen s klíčem (nebo `no`) |
| `PubkeyAuthentication` | `yes` | Povolení přihlášení klíčem |
| `PasswordAuthentication` | `no` | Zakázání hesla (pouze klíče!) |
| `AllowUsers` | `alice` `bob` | Omezení na konkrétní uživatele |
| `MaxAuthTries` | `3` | Maximální počet pokusů o přihlášení |
| `ClientAliveInterval` | `300` | Kontrola spojení každých 300s |
| `LogLevel` | `VERBOSE` | Detailní logování přihlášení |

### Aplikace změn

```bash
sudo nano /etc/ssh/sshd_config      # úprava konfigurace
sudo systemctl restart sshd         # restart služby
```

> **⚠️ VAROVÁNÍ:** Před vypnutím `PasswordAuthentication` se **vždy nejdříve ujistěte, že přihlášení klíčem funguje**. Otevřete druhý terminál a vyzkoušejte přihlášení: `ssh -v uzivatel@server`. Pokud přihlášení nefunguje, budete mít zamčený přístup!

> **⚠️ VAROVÁNÍ:** Změna `Port` vyžaduje také úpravu firewallu: `sudo ufw allow 2222/tcp`. Nezapomeňte také aktualizovat config soubor na klientském počítači (viz Config soubor).

### Ověření nastavení

```bash
# Detailní debug výpis průběhu připojení
ssh -v uzivatel@server

# Kontrola logu SSH serveru
sudo journalctl -u sshd --since "1 hour ago"
```

`ssh -v` ukáže, které klíče se zkouší, zda proběhla autentizace klíčem nebo heslovou a případné chyby. Pro více detailů použijte `ssh -vvv`.

---

## Config soubor (~/.ssh/config)

Config soubor `~/.ssh/config` umožňuje definovat zkratky a nastavení pro jednotlivé servery. Místo psaní `ssh -p 2222 alice@192.168.1.100 -i ~/.ssh/server.key` pak stačí `ssh home`.

### Direktivy config souboru

| Direktiva | Popis | Příklad |
|-----------|-------|---------|
| `Host` | Zástupný název (alias) – používá se při volání `ssh home` | `Host home` |
| `HostName` | Skutečná adresa nebo IP serveru | `HostName 192.168.1.100` |
| `User` | Uživatelské jméno pro přihlášení | `User alice` |
| `IdentityFile` | Cesta k soukromému klíči pro tento host | `IdentityFile ~/.ssh/server.key` |
| `Port` | Port (jiný než výchozí 22) | `Port 2222` |

### Ukázka config souboru

```
# Osobní server
Host home
    HostName 192.168.1.100
    User alice
    Port 2222
    IdentityFile ~/.ssh/home.key

# GitHub (osobní přístup)
Host github.com
    User git
    IdentityFile ~/.ssh/github.key

# Firemní server
Host work
    HostName work.example.com
    User petr
    Port 2222
    IdentityFile ~/.ssh/work.key
```

S takto nastaveným configem stačí psát:
```bash
ssh home          # alias pro ssh -p 2222 alice@192.168.1.100
ssh work          # alias pro ssh -p 2222 petr@work.example.com
ssh github.com    # alias pro ssh git@github.com
```

### Wildcard (zástupný znak)

Pro skupinu podobných serverů lze použít `*`:
```
Host *.local
    User admin
    IdentityFile ~/.ssh/local.key
```

---

## Klíče pro GitHub

GitHub podporuje autentizaci přes SSH – místo zadávání hesla a tokenu se připojíte pomocí SSH klíče.

### Nastavení osobního SSH klíče

```bash
# 1. Generování samostatného klíče pro GitHub
ssh-keygen -t ed25519 -f ~/.ssh/github.key -C "github"

# 2. Zkopírování veřejného klíče
cat ~/.ssh/github.key.pub
```

3. Veřejný klíč přidejte na **GitHub → Settings → SSH and GPG keys → New SSH key**
4. Ověření funkčnosti:

```bash
ssh -T git@github.com
# Výstup: Hi username! You've successfully authenticated...
```

### Config pro GitHub (viz výše)

Do `~/.ssh/config` přidejte:
```
Host github.com
    User git
    IdentityFile ~/.ssh/github.key
```

> **ℹ️ POZNÁMKA:** GitHub používá pro SSH uživatele `git` bez ohledu na váš účet. Autentizace probíhá pomocí klíče, ne podle jména uživatele.

### Deploy klíče pro repozitáře

Deploy klíč umožňuje konkrétnímu repozitáři přistupovat k jinému repozitáři (např. pro CI/CD, automatické deploye).

1. Generování deploy klíče: `ssh-keygen -t ed25519 -f ~/.ssh/deploy.key -C "deploy"`
2. Přidání do repozitáře: **GitHub → Repozitář → Settings → Deploy keys → Add deploy key**
3. Povolení zápisu (Allow write access) pokud je potřeba pushovat

### Samostatné klíče pro různé servery

Pro každý server nebo službu používejte **samostatný pár klíčů**. Pokud dojde ke kompromitaci jednoho klíče, ostatní servery zůstanou v bezpečí.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/server1.key -C "server1"
ssh-keygen -t ed25519 -f ~/.ssh/github.key -C "github"
ssh-keygen -t ed25519 -f ~/.ssh/gitlab.key -C "gitlab"
```

Každý klíč pak přiřaďte v `~/.ssh/config` pomocí `IdentityFile`.

---

## Kopírování souborů přes SSH

SSH umožňuje bezpečný přenos souborů mezi počítači pomocí nástrojů `scp` a `rsync`. Oba nástroje využívají stejné šifrování jako SSH.

### SCP (Secure Copy)

Základní nástroj pro kopírování jednotlivých souborů.

```bash
# Lokál → server
scp soubor.txt uzivatel@server:/cesta/na/serveru/

# Server → lokál
scp uzivatel@server:/cesta/soubor.txt .

# Rekurzivně celý adresář
scp -r adresar/ uzivatel@server:/cesta/

# Jiný port (pokud jste změnili Port v sshd_config)
scp -P 2222 soubor.txt uzivatel@server:/cesta/
```

### RSYNC (pokročilá synchronizace)

Rsync je výkonnější než scp – přenáší pouze změny (diff), umí mazat soubory a zachovává oprávnění.

```bash
# Základní kopie s kompresí (-z) a podrobným výpisem (-v)
rsync -avz soubor.txt uzivatel@server:/cesta/

# Zrcadlení (včetně mazání souborů, které na zdroji neexistují)
rsync -avz --delete uzivatel@server:/cesta/ ./zaloha/

# Jiný port SSH
rsync -avz -e "ssh -p 2222" soubor.txt uzivatel@server:/cesta/

# Dry-run (co by se stalo, bez skutečného přenosu)
rsync -avz -n soubor.txt uzivatel@server:/cesta/
```

> **ℹ️ POZNÁMKA:** Rsync po prvním přenosu přenáší pouze změněné bloky souborů, což výrazně šetří přenosové pásmo. Pro časté zálohování je rsync vhodnější než scp.

---

## SSH tunelování

SSH umí přesměrovávat porty – vytváří **šifrovaný tunel** mezi lokálním a vzdáleným počítačem. To se hodí pro zabezpečení jinak nešifrovaných protokolů nebo pro obcházení firewallů.

### Local port forwarding (-L)

Přesměruje **lokální port** na vzdálený počítač přes SSH server.

```bash
# Vzdálená webová služba na localhost:8080 (přes SSH server)
ssh -L 8080:localhost:80 uzivatel@server

# Přesměrování na jiný stroj v síti (např. databáze)
ssh -L 3306:db-server:3306 uzivatel@jump-server
```

Po spuštění otevřete v prohlížeči `http://localhost:8080` a uvidíte obsah vzdáleného serveru. Veškerá komunikace je šifrovaná přes SSH.

### Remote port forwarding (-R)

Zpřístupní **lokální službu** na vzdáleném serveru.

```bash
# Lokální port 3000 bude dostupný na serveru na portu 9000
ssh -R 9000:localhost:3000 uzivatel@server
```

> **⚠️ VAROVÁNÍ:** Remote forwarding vyžaduje na serveru v `/etc/ssh/sshd_config` povolené `GatewayPorts no` (výchozí) – pak je port dostupný pouze na localhostu serveru. Pro přístup z venku je třeba `GatewayPorts yes`.

### SOCKS proxy (-D)

Vytvoří **dynamický tunel** – SOCKS proxy server na lokálním počítači.

```bash
ssh -D 1080 uzivatel@server
```

Poté v prohlížeči nastavte SOCKS proxy na `localhost:1080`. Veškerý webový provoz pak půjde přes SSH server.

### Tunel bez shellu (-N)

Přepínač `-N` zajistí, že se nespustí shell – tunel je aktivní, ale nevidíte příkazovou řádku.

```bash
ssh -N -L 8080:localhost:80 uzivatel@server
```

> **⚠️ VAROVÁNÍ:** SSH tunel šifruje data pouze mezi vaším počítačem a SSH serverem. Data mezi SSH serverem a cílovým počítačem (např. `db-server:3306`) jdou nešifrovaná, pokud samotný protokol nepodporuje šifrování.

---

## Shrnutí a bezpečnostní doporučení

SSH je nepostradatelný nástroj pro bezpečnou správu vzdálených serverů. Tato kapitola pokryla všechny základní dovednosti – od generování klíčů přes konfiguraci serveru až po tunelování.

### Bezpečnostní desatero SSH

1. **Vždy používejte klíče, nikdy hesla** – vypněte `PasswordAuthentication no` v sshd_config
2. **Soukromý klíč chraňte jako heslo** – oprávnění `600`, ideálně s passphrase
3. **Každý server/účel má vlastní pár klíčů** – kompromitace jednoho neovlivní ostatní
4. **Zazálohujte si veřejné klíče a config** – při ztrátě soukromého klíče se snadno obnovíte
5. **Měňte výchozí Port 22** – snížíte počet automatických útoků v logu
6. **Root přihlášení jen s klíčem** – `PermitRootLogin prohibit-password`
7. **Omezte uživatele a počet pokusů** – `AllowUsers` a `MaxAuthTries 3`
8. **Pravidelně kontrolujte `~/.ssh/authorized_keys`** – a `journalctl -u sshd`
9. **Používejte `ssh -v` pro diagnostiku** – verbose mód odhalí problémy s autentizací
10. **GitHub klíče spravujte přes Settings → SSH and GPG keys** – pravidelně kontrolujte, které klíče máte nahrané

### Tabulka: Kdy použít kterou techniku

| Účel | Nástroj | Příkaz |
|------|---------|--------|
| Kopie souboru | `scp` | `scp soubor server:/cesta/` |
| Synchronizace adresáře | `rsync` | `rsync -avz adresar/ server:/cesta/` |
| Lokální služba přes SSH | Local forward | `ssh -L 8080:localhost:80 server` |
| Sdílení lokálního portu | Remote forward | `ssh -R 9000:localhost:3000 server` |
| Bezpečný proxy prohlížeč | SOCKS | `ssh -D 1080 server` |
| Diagnostika připojení | verbose | `ssh -v user@server` |

> **ℹ️ POZNÁMKA:** Pro pokročilou bezpečnostní ochranu SSH serveru (fail2ban, PAM, AppArmor) viz [kapitolu 7 – Bezpečná architektura](07-bezpecna-architektura.md) a [kapitolu 15 – AppArmor](15-apparmor.md).

---

➡️ [Zpět na přehled](README.md)
