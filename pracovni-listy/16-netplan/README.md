# Pracovní list: Netplan

> Procvičení konfigurace síťového rozhraní pomocí Netplan YAML syntaxe.

---

## Zadání

### Úkol 1: Analýza současné sítě

1. Zobraz aktuální síťová rozhraní: `ip a` nebo `ip addr`
2. Zjisti aktuální konfiguraci Netplan: `netplan get`
3. Zjisti, jaký backend Netplan používá: `netplan info`
4. Zobraz všechny konfigurační soubory v `/etc/netplan/`
5. Zobraz obsah prvního konfiguračního souboru
6. Jaký je rozdíl mezi backendy `networkd` a `NetworkManager`?

### Úkol 2: Napiš YAML konfiguraci

Napiš Netplan konfiguraci pro následující scénář:

- Rozhraní `eth0` se statickou IP `192.168.1.100/24`
- Brána `192.168.1.1`
- DNS servery: `8.8.8.8` a `1.1.1.1`
- DNS search doména: `example.com`
- Zakázané DHCP
- Renderer: `networkd`

Zapiš YAML do souboru `01-netcfg.yaml` (ve skutečnosti jen jako cvičení, neměň systémovou konfiguraci, pokud nevíš co děláš).

### Úkol 3: Testování a troubleshooting

1. Jak bys otestoval konfiguraci před tím, než ji natrvalo aplikuješ?
   - K čemu slouží `netplan try` a jaký má časový limit?
2. Co znamená výstup `netplan status`?
3. Jak bys přidal druhou IP adresu ke stejnému rozhraní?
4. Jak bys nastavil DHCP pro `eth0` a zároveň statickou IP pro `eth1`?
5. Co uděláš, když po `netplan apply` ztratíš spojení se serverem?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1. Síťová rozhraní
ip addr

# 2. Aktuální konfigurace
netplan get

# 3. Backend
netplan info
# backend: systemd-networkd nebo NetworkManager

# 4-5. Konfigurační soubory
ls /etc/netplan/
cat /etc/netplan/*.yaml 2>/dev/null

# 6. Rozdíl:
# networkd = systemd-networkd, lehký, vhodný pro servery
# NetworkManager = desktopový, podpora WiFi, VPN, dynamické změny
```

### Úkol 2 — Řešení

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      dhcp6: false
      addresses:
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
```

### Úkol 3 — Řešení

```bash
# 1. Testování: netplan try
sudo netplan try
# Aplikuje konfiguraci s 30s timeoutem
# Pokud nepotvrdíš (Enter), automatický rollback
# Bezpečné pro vzdálené servery

# 2. netplan status
sudo netplan status
# Zobrazí online state, DNS, stav rozhraní

# 3. Druhá IP adresa (IP alias)
# Přidat do addresses:
#   - 192.168.1.100/24
#   - 192.168.1.200/24

# 4. Kombinace DHCP a statické IP
# eth0: dhcp4: true
# eth1: dhcp4: false, addresses: [192.168.1.101/24], routes: ...

# 5. Při ztrátě spojení:
# a. Počkat na timeout netplan try (automatický rollback)
# b. Nebo připojit se přes iLO/iDRAC/KVM
# c. Nebo fyzicky restartovat (poslední známá konfigurace)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Znáš základní Netplan příkazy (get, apply, try, status, info)
- [ ] Umíš napsat YAML konfiguraci pro statickou IP
- [ ] Víš, k čemu slouží `netplan try` a jaký má timeout
- [ ] Rozumíš rozdílu mezi backendy networkd a NetworkManager

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../16-netplan.md) · [Zpět na přehled](../../README.md)
