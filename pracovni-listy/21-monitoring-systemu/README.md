# Pracovní list: Monitoring systému

> Procvičení monitorování CPU, paměti, disku a sítě pomocí CLI nástrojů.

---

## Zadání

### Úkol 1: CPU monitoring

1. Zobraz aktuální load average z `/proc/loadavg` a interpretuj hodnoty
2. Spusť `top -o %CPU` a zaznamenej 3 nejvytíženější procesy
3. Zobraz vytížení na každé jádro zvlášť: `mpstat -P ALL 1 3` (nainstaluj `sysstat`)
4. Sleduj sloupec `%iowait` — co znamená, když je nad 10 %?
5. Zobraz procesy setříděné podle CPU spotřeby: `ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -10`

### Úkol 2: Paměť a disk

1. Zobraz využití paměti: `free -h`
   - Zapiš: total, used, available
   - Vysvětli rozdíl mezi "used" a "available"
2. Zobraz využití swapu: `swapon --show`
3. Zobraz využití disků: `df -h`
   - Který oddíl je nejvíce zaplněný?
4. Zobraz diskovou aktivitu: `iostat -x 1 3`
   - Sleduj `%util` — co znamená hodnota blízká 100 %?
5. Zobraz největší adresáře v `/`: `du -sh /* 2>/dev/null | sort -rh | head -10`

### Úkol 3: Síťový monitoring

1. Zobraz síťová rozhraní a IP adresy: `ip addr`
2. Zobraz síťové statistiky: `ip -s link`
3. Zobraz aktuální síťové spojení: `ss -tlnp` (naslouchající porty)
4. Zobraz aktivní TCP spojení: `ss -tup`
5. Sleduj propustnost sítě: `nload` nebo `iftop` (pokud je nainstalováno)
6. Jaký nástroj použiješ pro dlouhodobé sledování výkonu a ukládání metrik?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1. Load average
cat /proc/loadavg
# 1.50 1.20 0.80 3/456 12345
# 1 min, 5 min, 15 min | running/total | last PID
# Na 4-jádrovém CPU: load 3 = v pohodě
# Na 2-jádrovém CPU: load 3 = přetížení

# 2. top
top -o %CPU -b -n 1 | head -17

# 3. mpstat
sudo apt install sysstat -y
mpstat -P ALL 1 3

# 4. %iowait > 10% = systém čeká na disk (I/O bottleneck)

# 5. Procesy podle CPU
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -10
```

### Úkol 2 — Řešení

```bash
# 1-2. Paměť a swap
free -h
# available = free + cache (kterou lze uvolnit pro aplikace)
# used = skutečně používaná paměť
swapon --show

# 3. Disky
df -h

# 4. Disková aktivita
iostat -x 1 3
# %util blízko 100% = disk je bottleneck (plně vytížen)

# 5. Největší adresáře
du -sh /* 2>/dev/null | sort -rh | head -10
```

### Úkol 3 — Řešení

```bash
# 1-4. Síť
ip addr
ip -s link
ss -tlnp  # naslouchající TCP porty
ss -tup   # aktivní TCP spojení

# 5. Sledování propustnosti
nload       # nebo iftop, iptraf-ng
nload eth0  # sledovat konkrétní rozhraní

# 6. Pro dlouhodobý monitoring:
# - Prometheus + Grafana (profesionální)
# - Nagios / Zabbix (tradiční)
# - netdata (jednoduchý)
# - collectd + grafana (lehké)
# - sysstat (sar) pro historická data
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš interpretovat load average vzhledem k počtu jader
- [ ] Rozumíš výstupu `free -h` a rozdílu used/available
- [ ] Víš, co znamená `%iowait` a `%util` u disku
- [ ] Dokážeš zjistit naslouchající porty a aktivní spojení

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../21-monitoring-systemu.md) · [Zpět na přehled](../../README.md)
