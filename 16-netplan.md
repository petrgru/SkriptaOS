# Netplan — Konfigurace síťového rozhraní

> Network Configuration with Netplan on Ubuntu

---

## Úvod

> **Platforma:** Netplan je výchozí nástroj pro konfiguraci sítě na **Ubuntu 18.04+**. Nahrazuje tradiční `/etc/network/interfaces`. Konfigurace se zapisuje do YAML souborů v `/etc/netplan/` a Netplan generuje konfiguraci pro **networkd** (systemd) nebo **NetworkManager**.

Netplan používá deklarativní YAML syntaxi — popíšete požadovaný stav sítě a Netplan zajistí jeho aplikaci. Podporuje DHCP, statické IP, VLAN, bridge, bonding a pokročilé routovací parametry.

---

## 1. Základní příkazy

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `netplan apply` | Aplikuje konfiguraci (generuje + restartuje služby) | `sudo netplan apply` | (žádný výstup, pokud OK) |
| `netplan try` | Otestuje konfiguraci s časovým limitem (automatický rollback) | `sudo netplan try` | `Do you want to keep these settings?↵Press ENTER before the timeout to accept` |
| `netplan generate` | Pouze vygeneruje konfigurační soubory pro backend | `sudo netplan generate` | (žádný výstup) |
| `netplan get` | Zobrazí aktuální konfiguraci | `netplan get` | `network:`<br>  `ethernets:`<br>    `eth0:`<br>      `dhcp4: true` |
| `netplan set` | Nastaví hodnotu v konfiguraci (jednoduché změny) | `sudo netplan set ethernets.eth0.dhcp4=false` | (žádný výstup) |
| `netplan status` | Zobrazí aktuální stav rozhraní | `sudo netplan status` | `     Online state: online`<br>    `DNS Addresses: 192.168.1.1`<br>     `eth0: UP`<br>       `Gateway: 192.168.1.1` |
| `netplan info` | Zobrazí informace o backendu | `netplan info` | `backend: systemd-networkd` |

```bash
# Nejčastější pracovní postup
sudo nano /etc/netplan/01-netcfg.yaml     # úprava konfigurace
sudo netplan try                           # otestování (30s timeout)
# Pokud OK: Enter pro potvrzení
# Pokud timeout: automatický rollback
sudo netplan apply                         # finální aplikace
```

---

## 2. Struktura konfiguračních souborů

Soubory v `/etc/netplan/` mají příponu `.yaml`. Netplan je zpracovává v abecedním pořadí (pozdější soubory přepisují dřívější).

| Soubor | Typické použití |
|--------|-----------------|
| `01-netcfg.yaml` | Základní síťová konfigurace (DHCP, statická IP) |
| `01-network-manager-all.yaml` | Výchozí po instalaci (vše přes NetworkManager) |
| `99-custom.yaml` | Vlastní přidaná konfigurace (vyšší priorita) |

### Základní struktura YAML

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2                  # Verze schématu (vždy 2)
  renderer: networkd          # Backend: networkd (doporučeno) nebo NetworkManager
  ethernets:                  # Sekce pro ethernetová rozhraní
    eth0:                     # Název rozhraní
      dhcp4: true             # DHCP pro IPv4
      dhcp6: false            # Zakázat DHCP pro IPv6
    eth1:
      dhcp4: no               # Vypnout DHCP
      addresses:              # Statická IP
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
        search:
          - example.com
  wifis:                      # Sekce pro WiFi rozhraní
    wlan0:
      access-points:
        "MyWiFi":
          password: "heslo123"
      dhcp4: true
```

---

## 3. Běžné konfigurace

### DHCP (automatická adresa)

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
      dhcp6: false
```

### Statická IP adresa

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 10.0.0.100/24
      routes:
        - to: default
          via: 10.0.0.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
        search: [example.local]
```

### Statická IP s více adresami

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 10.0.0.100/24
        - 10.0.0.101/24     # Druhá IP na stejném rozhraní
      routes:
        - to: default
          via: 10.0.0.1
        - to: 192.168.100.0/24
          via: 10.0.0.254    # Statická routa do jiné sítě
      nameservers:
        addresses: [8.8.8.8]
```

### WiFi připojení

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: NetworkManager     # Pro WiFi je potřeba NetworkManager
  wifis:
    wlan0:
      access-points:
        "MojeSit":
          password: "bezpecne-heslo"
        "Kavarna-WiFi":
          password: "heslo-kavarna"
      dhcp4: true
```

> **📌 Poznámka:** Pro WiFi se obvykle používá `renderer: NetworkManager`. Pokud používáte `networkd`, WiFi musí být nakonfigurováno přes `wpa_supplicant`.

---

## 4. Pokročilé konfigurace

### VLAN

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
  vlans:
    vlan.10:
      id: 10
      link: eth0
      addresses: [192.168.10.1/24]
    vlan.20:
      id: 20
      link: eth0
      addresses: [192.168.20.1/24]
```

### Bridge (pro virtuální stroje nebo kontejnery)

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
  bridges:
    br0:
      addresses: [192.168.1.100/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8]
      interfaces: [eth0]      # Členové bridge
```

### Bonding (Link Aggregation)

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
    eth1:
      dhcp4: false
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      addresses: [10.0.0.1/24]
      parameters:
        mode: 802.3ad         # LACP (aktivní)
        mii-monitor-interval: 100
        lacp-rate: fast
```

---

## 5. Řešení problémů

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `netplan --debug apply` | Aplikace s debug výstupem | `sudo netplan --debug apply` | `DEBUG: generating configuration...` |
| `netplan try --timeout 60` | Test s vlastním timeoutem (s) | `sudo netplan try --timeout 60` | `Settings will be reverted in 60 seconds` |
| `networkctl status` | Stav rozhraní (systemd-networkd) | `networkctl status eth0` | `● State: routable (configured)`<br>  `Link: 1000 Mbps`<br>  `Address: 10.0.0.100` |
| `networkctl list` | Seznam všech rozhraní | `networkctl list` | `IDX LINK  TYPE     OPERATIONAL`<br>  `1 lo    loopback carrier`<br>  `2 eth0  ether    routable` |
| `resolvectl status` | DNS resolver (systemd-resolved) | `resolvectl status` | `Global: DNS Servers: 8.8.8.8` |
| `sudo netplan status --diff` | Rozdíl mezi konfigurací a aktuálním stavem | `sudo netplan status --diff` | `-  eth0:`<br>  `+  eth0:`<br>  `-    dhcp4: true` |

### Typické chyby

```bash
# Chyba: YAML syntax error
# Řešení: Zkontrolujte odsazení (YAML používá mezery, ne taby!)
# Výstup: netplan: Cannot parse YAML config: mapping values are not allowed here

# Chyba: neplatný název rozhraní
# Výstup: netplan: unknown interface 'eth3'
# Řešení: Zkontrolujte existující rozhraní příkazem ip link show

# Chyba: konfigurace neplatná pro backend
# Výstup: netplan: networkd: option 'access-points' is not valid for networkd
# Řešení: WiFi vyžaduje renderer: NetworkManager

# Rollback po ztrátě spojení
# Pokud se po netplan apply ztratí SSH spojení:
# 1. Připojit se přes iLO/IPMI nebo fyzicky
# 2. Spustit netplan try s delším timeoutem
# 3. Pokud se nic nestane, počkat na automatický rollback
```

---

## 6. Renderer: networkd vs NetworkManager

| Vlastnost | networkd (systemd) | NetworkManager |
|-----------|-------------------|----------------|
| **Rychlost** | Rychlejší start | Pomalejší |
| **Závislosti** | Minimalistické (systemd) | Více balíčků |
| **WiFi** | Pouze přes wpa_supplicant | Nativní podpora |
| **Desktop** | Omezené | Plná podpora (tray icon, GUI) |
| **Server** | ✅ Doporučeno | Možné, ale zbytečné |
| **DHCP** | Vynikající | Vynikající |
| **VLAN/Bridge/Bond** | ✅ Plná podpora | ✅ Plná podpora |

```bash
# Změna rendereru
# networkd → NetworkManager
sudo sed -i 's/renderer: networkd/renderer: NetworkManager/' /etc/netplan/01-netcfg.yaml
sudo netplan apply

# NetworkManager → networkd
sudo sed -i 's/renderer: NetworkManager/renderer: networkd/' /etc/netplan/01-netcfg.yaml
sudo netplan apply
```

---

## 7. Obnova po chybné konfiguraci

```yaml
# /etc/netplan/01-recovery.yaml — záchranná konfigurace (abecedně první)
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
```

```bash
# Pokud jste ztratili přístup k serveru chybnou konfigurací:

# 1. Přes fyzickou konzoli nebo iLO/IPMI
# 2. Smažte nebo přejmenujte chybný soubor
sudo mv /etc/netplan/90-broken.yaml /etc/netplan/90-broken.yaml.bak

# 3. Aplikujte záchrannou konfiguraci
sudo netplan apply

# 4. Pokud ani to nepomůže — ruční restart networkd
sudo systemctl restart systemd-networkd

# 5. Poslední možnost — ruční nastavení IP
sudo ip addr add 10.0.0.100/24 dev eth0
sudo ip route add default via 10.0.0.1
```

---

## Shrnutí

- **Netplan** je výchozí nástroj pro konfiguraci sítě na Ubuntu 18.04+. Používá YAML v `/etc/netplan/`.
- **Klíčové příkazy:** `netplan apply` (aplikace), `netplan try` (test s rollbackem), `netplan generate` (pouze generování).
- **Backend:** `networkd` (doporučeno pro servery) nebo `NetworkManager` (pro desktop/WiFi).
- **YAML syntaxe:** Dbejte na správné odsazení (mezery, ne taby). `network: version: 2` je vždy povinný.
- **Testování:** Vždy používejte `netplan try` před `netplan apply` na vzdálených serverech — automatický rollback po timeoutu.
- **Běžné chyby:** YAML syntaxe, neexistující rozhraní, nesprávný renderer pro WiFi.
- **Záchranný plán:** Mějte připravenou DHCP konfiguraci pro případ chyby.

---

➡️ [Zpět na přehled](README.md)
