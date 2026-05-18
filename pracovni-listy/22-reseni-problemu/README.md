# Pracovní list: Řešení problémů

> Procvičení diagnostiky a řešení problémů — disk, CPU, síť, logy.

---

## Zadání

### Úkol 1: Disk je plný

Simuluj situaci: `/var/log/` zabírá příliš mnoho místa.

1. Zjisti využití disku: `df -h`
2. Najdi největší adresáře v `/var`: `du -sh /var/* | sort -rh | head -10`
3. Najdi soubory v `/var/log/` větší než 100 MB: `find /var/log -type f -size +100M`
4. Zkontroluj velikost journal logů: `journalctl --disk-usage`
5. Jak bys vyčistil staré logy?
   - Pomocí `journalctl --vacuum-size=200M`
   - Pomocí `logrotate`
6. Proč `df` a `du` můžou ukazovat rozdílné hodnoty?

### Úkol 2: Vysoké CPU

Server reaguje pomalu. Postupuj podle diagnostického checklistu:

1. Zjisti load average: `cat /proc/loadavg` a interpretuj
2. Najdi proces s nejvyšším CPU: `ps aux --sort=-%cpu | head -5`
3. Zkontroluj detaily procesu: `top -p PID`
4. Zjisti, co proces dělá: `strace -p PID -e trace=write 2>&1 | head`
5. Kolik vláken proces má: `ps -o nlwp -p PID`
6. Jaké jsou možnosti řešení? (nice, kill, restart, optimalizace)

### Úkol 3: Síťový problém

Aplikace na portu 3000 není dostupná. Diagnostikuj:

1. Zjisti, jestli port 3000 naslouchá: `ss -tlnp | grep 3000`
2. Pokud ne, zkus spustit jednoduchý test: `python3 -m http.server 3000 &`
3. Ověř, že server odpovídá: `curl http://localhost:3000`
4. Zkontroluj firewall: `sudo ufw status` nebo `sudo iptables -L INPUT -n`
5. Zkontroluj, jestli aplikace není v cgroup s omezením: `systemd-cgls`
6. Co bys dělal, kdyby port naslouchá na localhostu, ale není dostupný zvenčí?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-3. Diagnostika disku
df -h
du -sh /var/* | sort -rh | head -10
find /var/log -type f -size +100M

# 4. Journal
journalctl --disk-usage

# 5. Čištění
sudo journalctl --vacuum-size=200M
# Nebo: sudo journalctl --vacuum-time=2weeks
# Logrotate: /etc/logrotate.conf a /etc/logrotate.d/

# 6. df čte metadata filesystému, du počítá soubory
# Proces může mít otevřený smazaný soubor (df vidí, du ne)
# Řešení: lsof | grep deleted → restart procesu
```

### Úkol 2 — Řešení

```bash
# 1-4. Diagnostika CPU
cat /proc/loadavg
ps aux --sort=-%cpu | head -5
top -p $(ps aux --sort=-%cpu | awk 'NR==2{print $2}')
strace -p $(ps aux --sort=-%cpu | awk 'NR==2{print $2}') -e trace=write 2>&1 | head

# 5. Počet vláken
ps -o nlwp -p PID

# 6. Řešení:
# - nice -n 19 (snížit prioritu)
# - kill -9 (nouzové ukončení)
# - Restart služby
# - Optimalizace kódu / konfigurace
# - Přidání CPU / horizontální škálování
```

### Úkol 3 — Řešení

```bash
# 1-3. Diagnostika sítě
ss -tlnp | grep 3000
python3 -m http.server 3000 &
curl http://localhost:3000

# 4. Firewall
sudo ufw status        # pokud je ufw
sudo iptables -L INPUT -n | grep 3000

# 5. Cgroups
systemd-cgls

# 6. Pokud port naslouchá na 127.0.0.1, není dostupný zvenčí
# Řešení: změnit bind adresu na 0.0.0.0
# Nebo použít SSH tunel: ssh -L 3000:localhost:3000 server
# Nebo reverse proxy (nginx) na veřejném portu
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Víš, jak diagnostikovat plný disk a vyčistit ho
- [ ] Umíš najít proces s vysokým CPU a analyzovat ho
- [ ] Dokážeš diagnostikovat síťový problém (port, firewall, bind adresa)
- [ ] Znáš 5-krokovou metodologii troubleshootingu

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../22-reseni-problemu.md) · [Zpět na přehled](../../README.md)
