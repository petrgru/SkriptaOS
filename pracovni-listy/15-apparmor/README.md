# Pracovní list: AppArmor

> Procvičení Mandatory Access Control (MAC) pomocí AppArmor profilů.

---

## Zadání

### Úkol 1: Základní správa AppArmor

1. Zkontroluj, zda je AppArmor nainstalován a aktivní:
   ```bash
   sudo aa-status
   ```
2. Zjisti počet načtených profilů a kolik jich je v režimu enforce
3. Zjisti, zda je AppArmor modul načten v kernelu (`cat /sys/module/apparmor/parameters/enabled`)
4. Zobraz si seznam procesů, které mají profil (`aa-status` výše)
5. Jaké jsou 3 režimy AppArmor? Vysvětli každý.

### Úkol 2: Práce s profily

1. Najdi profil pro `ping` (`/etc/apparmor.d/bin.ping`)
2. Zobraz jeho obsah — jaká oprávnění má?
3. Dočasně přepni profil pingu do complain režimu:
   ```bash
   sudo aa-complain /etc/apparmor.d/bin.ping
   ```
4. Ověř změnu pomocí `sudo aa-status`
5. Přepni zpět do enforce
6. Co se stane, když program poruší profil v enforce režimu?

### Úkol 3: Vytvoření jednoduchého profilu

1. Vytvoř skript `/tmp/test_apparmor.sh`:
   ```bash
   #!/bin/bash
   echo "Hello from AppArmor test"
   cat /etc/shadow 2>&1
   ```
2. Udělej ho spustitelným: `chmod +x /tmp/test_apparmor.sh`
3. Vygeneruj profil pomocí:
   ```bash
   sudo aa-genprof /tmp/test_apparmor.sh
   ```
   (V interaktivním režimu zvol "Scan" a pak "Finish" pro základní profil)
4. Zobraz vygenerovaný profil v `/etc/apparmor.d/tmp.test_apparmor.sh`
5. Co bys musel do profilu přidat, aby mohl číst `/etc/shadow`?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1. Stav AppArmor
sudo aa-status
# apparmor module is loaded.
# 15 profiles are loaded.
# 12 profiles are in enforce mode.
# 3 profiles are in complain mode.
# 10 processes have profiles defined.

# 2. Kontrola kernel modulu
cat /sys/module/apparmor/parameters/enabled
# Y (pokud je aktivní)

# 3. AppArmor má 3 režimy:
# Enforcing — profil se vynucuje, porušení se blokuje a loguje
# Complain — porušení se pouze loguje, neblokuje (testování)
# Disabled — AppArmor je vypnutý
```

### Úkol 2 — Řešení

```bash
# 1-2. Profil pro ping
cat /etc/apparmor.d/bin.ping
# Obsahuje oprávnění: cap_net_raw, /bin/ping, /etc/ping.conf

# 3. Přepnutí do complain
sudo aa-complain /etc/apparmor.d/bin.ping

# 4. Ověření
sudo aa-status | grep ping
# /usr/bin/ping complain mode

# 5. Návrat do enforce
sudo aa-enforce /etc/apparmor.d/bin.ping

# 6. V enforce režimu: AppArmor zablokuje přístup
# a zapíše deny zprávu do /var/log/syslog
# Program může selhat s "Permission denied" i jako root
```

### Úkol 3 — Řešení

```bash
# 1-3. Vytvoření a generování profilu
cat > /tmp/test_apparmor.sh << 'EOF'
#!/bin/bash
echo "Hello from AppArmor test"
cat /etc/shadow
EOF
chmod +x /tmp/test_apparmor.sh
sudo aa-genprof /tmp/test_apparmor.sh

# 4. Zobrazení profilu
cat /etc/apparmor.d/tmp.test_apparmor.sh
# #include <tunables/global>
# /tmp/test_apparmor.sh {
#   #include <abstractions/base>
#   /tmp/test_apparmor.sh r,
# }

# 5. Pro čtení /etc/shadow by bylo potřeba přidat:
# /etc/shadow r,
# Nebo: /etc/** r,
# (záleží na míře omezení)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Víš, jak zkontrolovat stav AppArmor a počet profilů
- [ ] Rozumíš rozdílu mezi enforce a complain režimem
- [ ] Umíš přepínat režimy profilů
- [ ] Víš, jak profil funguje a co znamená path-based MAC

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../15-apparmor.md) · [Zpět na přehled](../../README.md)
