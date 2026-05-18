# Pracovní list: Základy OS

> Teoretický základ: kernel, syscalls, scheduling

---

## Zadání

### Úkol 1: Vysvětli pojmy

Vlastními slovy vysvětli:

1. Co je kernel a jaká je jeho hlavní role v operačním systému?
2. Jaký je rozdíl mezi monolitickým jádrem a mikrojádrem?
3. Co je systémové volání (syscall) a jak probíhá?

### Úkol 2: Kategorizace

Přiřaď následující pojmy do správné kategorie (kernel / scheduler / syscall):
- `fork()` — vytvoření nového procesu
- context switch — přepnutí mezi procesy
- `open()` — otevření souboru
- round-robin — algoritmus plánování
- monolithic vs microkernel — architektura jádra

### Úkol 3: Praktické ověření

Spusť v terminálu:

```bash
uname -r          # verze jádra
lscpu | grep "Model name"  # informace o CPU
cat /proc/cpuinfo | grep "processor" | wc -l  # počet jader
```

Zapiš, co jednotlivé příkazy vypíší.

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

1. **Kernel** je jádro OS — prostředník mezi hardwarem a aplikacemi. Spravuje paměť, procesy, zařízení a syscally. Běží v privilegovaném režimu (kernel space).
2. **Monolitické jádro** — celý kernel v jednom celku (Linux). Rychlé, ale při chybě páduje celý systém. **Mikrojádro** — pouze minimum v kernelu, zbytek jako uživatelské procesy (Minix). Stabilnější, ale pomalejší kvůli IPC.
3. **Syscall** je požadavek aplikace na jádro (např. čtení souboru, vytvoření procesu). Proces: aplikace → knihovna → syscall → kernel → odpověď.

### Úkol 2 — Řešení

- `fork()` → **syscall**
- context switch → **scheduler**
- `open()` → **syscall**
- round-robin → **scheduler**
- monolithic vs microkernel → **kernel**

### Úkol 3 — Řešení

Příkazy vypíší:
- `uname -r` — číslo verze jádra (např. `6.1.0-23-amd64`)
- `lscpu | grep "Model name"` — model CPU (např. `Intel(R) Core(TM) i7`)
- `cat /proc/cpuinfo | grep "processor" | wc -l` — počet jader (např. `8`)

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Vysvětlil/a jsi kernel vlastními slovy — ne opsal/a definici
- [ ] Rozumíš rozdílu monolithic vs microkernel
- [ ] Spuštěné commandy vrátily reálná data o tvém systému
- [ ] Víš, co je context switch a k čemu slouží

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../00-zaklady-os.md) · [Zpět na přehled](../../README.md)
