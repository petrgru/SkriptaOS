# Pracovní list: Optimalizace výkonu

> Procvičení monitorování a ladění výkonu CPU, paměti, disku a kernel parametrů.

---

## Zadání

### Úkol 1: CPU profiling

1. Zjisti aktuální scaling governor procesoru (`cpupower frequency-info`)
2. Zjisti počet jader a model CPU z `/proc/cpuinfo`
3. Spusť `stress-ng` (nebo `yes > /dev/null &` na pozadí) a sleduj vytížení CPU pomocí `top -o %CPU`
4. Změř load average z `/proc/loadavg` a interpretuj hodnoty vzhledem k počtu jader
5. Zastav všechny procesy na pozadí

### Úkol 2: Analýza paměti

1. Použij `free -h` a zaznamenej celkovou, použitou a dostupnou paměť
2. Vysvětli rozdíl mezi "used" a "available" v `free -h`
3. Najdi 5 procesů s nejvyšším využitím paměti (`ps aux --sort=-%mem`)
4. Zjisti velikost swapu a jestli se aktuálně používá
5. Ověř `/proc/meminfo` — najdi hodnoty MemTotal, MemFree, SwapTotal

### Úkol 3: Disková I/O a tuning

1. Zjisti seznam disků a oddílů pomocí `lsblk`
2. Změř aktuální využití disků (`df -h`) a najdi nejvíce zaplněný oddíl
3. Spusť `iostat -x 1 3` (pokud není, nainstaluj `sysstat`) a sleduj `%util`
4. Zkontroluj parametry kernelu pro VM: `sysctl vm.swappiness` — co tato hodnota znamená?
5. Jaká je doporučená hodnota `vm.swappiness` pro server a pro workstation?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1. Aktuální governor
cpupower frequency-info 2>/dev/null | head -10

# 2. Informace o CPU
grep -c ^processor /proc/cpuinfo  # počet jader
grep "model name" /proc/cpuinfo | head -1  # model CPU

# 3. Zátěžový test v pozadí
stress-ng --cpu 1 --timeout 10s &
# Nebo: yes > /dev/null &
top -o %CPU

# 4. Load average
cat /proc/loadavg
# 1.00 0.50 0.25 3/456 12345
# Interpretace: na 4-jádrovém CPU je load 3.5 v pohodě,
# na 2-jádrovém už znamená přetížení

# 5. Zastavení procesů
kill %1  # zastaví první úlohu na pozadí
```

### Úkol 2 — Řešení

```bash
# 1. Volná paměť
free -h
#                total  used  free  shared  buff/cache  available
# Mem:           15Gi  6.2Gi 2.1Gi   0.5Gi       7.0Gi     8.3Gi

# 2. "used" = opravdu používaná, "available" = k dispozici pro nové procesy
# available = free + část buff/cache (cache lze uvolnit)

# 3. Top 5 podle paměti
ps aux --sort=-%mem | head -6

# 4. Swap
swapon --show
free -h | grep Swap

# 5. /proc/meminfo
grep -E '^(MemTotal|MemFree|SwapTotal)' /proc/meminfo
```

### Úkol 3 — Řešení

```bash
# 1. Seznam disků
lsblk

# 2. Využití disků
df -h

# 3. Disková I/O
iostat -x 1 3
# %util blízko 100 % = disk je bottleneck

# 4. vm.swappiness
sysctl vm.swappiness
# Výchozí: 60
# Čím nižší, tím méně kernel swapuje (preferuje RAM)

# 5. Doporučené hodnoty
# Server (DB, výpočty): 10 (minimalizovat swap)
# Workstation: 30-60 (vyvážené)
# Pro SSD se doporučuje nižší hodnota (10-20)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš zjistit aktuální scaling governor a počet jader CPU
- [ ] Rozumíš výstupu `free -h` a rozdílu used/available
- [ ] Dokážeš identifikovat diskový bottleneck pomocí `iostat`
- [ ] Víš, co znamená `vm.swappiness` a jak ji nastavit

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../09-optimalizace-vykony.md) · [Zpět na přehled](../../README.md)
