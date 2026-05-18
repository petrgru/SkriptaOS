# Pracovní list: Správa procesů

> Procvičení monitorování a řízení procesů — ps, top, kill, nice, signály.

---

## Zadání

### Úkol 1: Analýza procesů

Pomocí příkazů v terminálu:
1. Vypiš všechny běžící procesy uživatele `root` ve formátu: PID, příkaz, %CPU
2. Najdi 5 procesů s nejvyšším využitím paměti
3. Zjisti PID svého aktuálního shellu a jeho rodičovský proces
4. Vytvoř strom procesů (pomocí `ps --forest`) pro svého uživatele

Zapiš příkazy, které jsi použil, a okomentuj výsledky.

### Úkol 2: Práce se signály

1. Spusť v pozadí proces, který každou sekundu vypíše datum (`while true; do date; sleep 1; done`)
2. Pošli mu signál SIGSTOP a sleduj, co se stane
3. Pošli SIGCONT a obnov ho
4. Pošli SIGTERM a ukonči ho korektně
5. Zkusil jsi poslat SIGKILL místo SIGTERM — jaký je rozdíl?

### Úkol 3: Priorita procesů

1. Spusť náročný příkaz s nízkou prioritou: `nice -n 19 find / -name "*.txt" > /dev/null`
2. V jiném terminálu sleduj jeho prioritu pomocí `ps -eo pid,ni,cmd | grep find`
3. Zapiš výchozí nice hodnotu a změněnou hodnotu
4. Zkus změnit prioritu běžícího procesu pomocí `renice`
5. Proč bys používal `nice -n 19` místo výchozí hodnoty?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1. Procesy uživatele root
ps -U root -o pid,cmd,%cpu --no-headers

# 2. Top 5 podle paměti
ps aux --sort=-%mem | head -6

# 3. PID shellu a rodič
echo "Můj PID: $$, Rodič: $PPID"
# Nebo: ps -p $$ -o pid,ppid,cmd

# 4. Strom procesů
ps -u $(whoami) -o pid,ppid,stat,cmd --forest
```

### Úkol 2 — Řešení

```bash
# Spuštění procesu na pozadí
while true; do date; sleep 1; done &
# Výstup: [1] 12345

# Zastavení SIGSTOP
kill -19 12345
# Proces se zastaví (přestane vypisovat datum)

# Obnovení SIGCONT
kill -18 12345
# Proces pokračuje ve výpisu

# Ukončení SIGTERM
kill 12345  # výchozí je SIGTERM (15)
# Proces skončí

# Alternativa: SIGKILL (9) - násilné, proces nemůže zachytit
# kill -9 12345 - okamžité ukončení bez možnosti cleanupu
```

SIGTERM je **žádost** o ukončení — proces může provést cleanup. SIGKILL je **příkaz** — kernel proces okamžitě ukončí bez varování.

### Úkol 3 — Řešení

```bash
# Spuštění s nízkou prioritou
nice -n 19 find / -name "*.txt" > /dev/null &

# Sledování priority
ps -eo pid,ni,cmd | grep find
#      PID  NI COMMAND
#    12345  19 find / -name "*.txt"

# Změna priority běžícího procesu
renice -n 10 -p 12345

# Výchozí nice je 0, '-n 19' je nejnižší priorita
# Používá se pro úlohy, které nemají být na úkor jiných procesů
# (např. noční záloha, logrotate)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Dokážeš vypsat procesy podle uživatele, CPU a paměti
- [ ] Víš, jak zastavit a obnovit proces (SIGSTOP/SIGCONT)
- [ ] Rozumíš rozdílu mezi SIGTERM a SIGKILL
- [ ] Umíš nastavit prioritu procesu pomocí `nice` a `renice`

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../06-sprava-procesu.md) · [Zpět na přehled](../../README.md)
