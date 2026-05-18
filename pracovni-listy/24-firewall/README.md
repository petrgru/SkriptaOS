# Pracovní list: Firewall

> Procvičení iptables, nftables a ufw — pravidla, NAT, chainy a troubleshooting.

---

## Zadání

### Úkol 1: ufw (Uncomplicated Firewall)

1. Zkontroluj stav ufw: `sudo ufw status`
2. Povol SSH: `sudo ufw allow ssh`
3. Povol HTTP a HTTPS: `sudo ufw allow 80/tcp` a `sudo ufw allow 443/tcp`
4. Povol ufw: `sudo ufw enable`
5. Zkontroluj stav: `sudo ufw status verbose`
6. Zakáž port 23 (telnet): `sudo ufw deny 23`
7. Smaž pravidlo pro SSH: `sudo ufw delete allow ssh`
8. Zakaž ufw: `sudo ufw disable`

### Úkol 2: iptables — základní pravidla

1. Zobraz aktuální pravidla: `sudo iptables -L -n -v`
2. Zobraz pravidla v NAT tabulce: `sudo iptables -t nat -L -n`
3. Povol lokální loopback: `sudo iptables -A INPUT -i lo -j ACCEPT`
4. Povol navázaná spojení: `sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`
5. Povol SSH na portu 22: `sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT`
6. Povol HTTP a HTTPS: `sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT`
7. Zamítni vše ostatní: `sudo iptables -A INPUT -j DROP`
8. Zobraz výsledná pravidla

### Úkol 3: NAT a pokročilé koncepty

1. Vysvětli rozdíl mezi SNAT a DNAT
2. Jak bys nastavil masquerade (SNAT) pro odchozí traffic na eth0?
   ```bash
   sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   ```
3. Jak bys přesměroval port 8080 na port 80?
   ```bash
   sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
   ```
4. Jaký je rozdíl mezi iptables a nftables?
5. Ulož iptables pravidla: `sudo iptables-save > /etc/iptables/rules.v4`
6. Smaž všechna pravidla: `sudo iptables -F`

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-7. ufw pravidla
sudo ufw status
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
sudo ufw deny 23
sudo ufw delete allow ssh
sudo ufw disable

# ufw je wrapper nad iptables/nftables
# Zjednodušuje vytváření pravidel
# Výchozí politika: deny incoming, allow outgoing
```

### Úkol 2 — Řešení

```bash
# 1-8. iptables
sudo iptables -L -n -v

# Základní ochrana
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -j DROP

sudo iptables -L -n -v
```

### Úkol 3 — Řešení

```bash
# 1. SNAT = změna zdrojové adresy (odchozí)
# DNAT = změna cílové adresy (příchozí)

# 2. Masquerade (SNAT pro dynamické IP)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# 3. Přesměrování portu
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80

# 4. Rozdíl iptables vs nftables:
# iptables: starší, 5 tabulek, složité řetězení
# nftables: moderní, jedna unifikovaná syntaxe, rychlejší

# 5. Uložení pravidel
sudo iptables-save > /etc/iptables/rules.v4

# 6. Smazání
sudo iptables -F  # flush všech pravidel
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš nastavit základní pravidla v ufw
- [ ] Rozumíš iptables chainům (INPUT, OUTPUT, FORWARD)
- [ ] Víš, k čemu slouží NAT (SNAT, DNAT, MASQUERADE)
- [ ] Znáš rozdíl mezi iptables a nftables

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../24-firewall.md) · [Zpět na přehled](../../README.md)
