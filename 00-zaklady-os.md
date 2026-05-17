# Základy OS

> Operating System Fundamentals

---

## Úvod

Tato kapitola je **teoretický základ** pro všechny následující kapitoly (01-19). Než se pustíte do konkrétních příkazů, skriptování nebo správy systému, musíte pochopit, co se pod kapotou vlastně děje.

Cílem je, abyste věděli **PROČ** děláte věci z kapitol 01-19, ne jen **JAK** je dělat. Když budete psát bash skript nebo spravovat procesy, měli byste tušit, jak do toho mluví kernel, scheduler a syscally.

Co získáte:
- Pochopení kernelu a jeho role
- Co jsou systémová volání (syscalls) a jak fungují
- Typy jader (monolitické, mikrojádro, hybridní)
- Základní přehled plánovače procesů
- Kontext pro všechna praktická cvičení v celém kurzu

Platforma: **Linux** (Ubuntu/Debian), vše je ale většinou přenositelné na jiné UNIXové systémy.

---

## Co je operační systém

**Definice**: Operační systém je vrstva mezi hardwarem a aplikacemi. Stará se o to, aby programy mohly používat hardware, aniž by musely znát jeho detaily, a aby mezi sebou navzájem nezasahovaly.

### Abstrakce hardware

| Hardware | Abstrakce v OS | Správce |
|----------|---------------|---------|
| CPU + RAM | Procesy | Plánovač (scheduler) |
| Disk | Soubory | Souborový systém (VFS) |
| RAM | Virtuální paměť | MMU + kernel |
| Síťová karta | Sockets | TCP/IP stack |
| Periferie | Zařízení (/dev) | Device drivers |

Každý fyzický zdroj má v OS svou logickou reprezentaci. Programátor neposílá data přímo na disk, ale zapisuje do souboru. Nepřiděluje si CPU cykly, ale vytváří proces. Tato abstrakce je to, co dělá OS použitelným.

### Role operačního systému

- **Správce prostředků (resource manager)** — kernel rozhoduje, kdo dostane CPU, paměť, I/O.
- **Poskytovatel abstrakcí** — skrývá detaily hardware za uniformní rozhraní (soubory, sockety, procesy).
- **Ochrana a izolace** — jeden proces nemůže číst paměť druhého (dokud to explicitně nepovolíte).

### Praktické commandy

```bash
# Zjisti, jaký kernel běží
uname -a

# Zjisti, jaká distribuce
cat /etc/os-release

# Kolik procesů běží
ps aux | wc -l
```

> **ℹ️ POZNÁMKA:** Tyto příkazy slouží jen jako ochutnávka. Podrobnému výkladu příkazů se věnují kapitoly 01 a 02.

---

## Kernel a userspace

### Dvě úrovně běhu

Operační systém Linux (a všechny moderní OS) rozděluje běh programu na **dvě úrovně**:

| Úroveň | Oprávnění | Označení | Co tu běží |
|--------|-----------|----------|------------|
| Kernel (jádro) | Plný přístup k hardware | Ring 0 | Jádro, ovladače, scheduler |
| Userspace (uživatelský prostor) | Omezený přístup | Ring 3 | Aplikace, služby, shell |

- **Kernel (Ring 0)** — má neomezený přístup k paměti, disku, síti, periferiím. Běží v privilegovaném režimu. Pokud kernel spadne, spadne celý systém.
- **Userspace (Ring 3)** — aplikace běží v neprivilegovaném režimu. Nemohou přímo přistupovat k hardware, nemohou číst cizí paměť. Pokud aplikace spadne, kernel to zachytí a ukončí jen tu jednu aplikaci.

### Proč oddělovat?

1. **Stabilita** — chybná aplikace nemůže shodit celý systém (na rozdíl od MS-DOS, kde ano).
2. **Bezpečnost** — aplikace nemůže číst paměť jiného procesu ani kernelu.
3. **Izolace** — každý proces má vlastní adresní prostor.

```
Textový diagram: Pro přesnější představu si to nakreslíme:

┌──────────────────────────────────┐
│         Userspace (Ring 3)       │
│  ┌────────┐  ┌────────┐         │
│  │  bash  │  │  vim   │  ...     │
│  └────────┘  └────────┘         │
│  ┌──────────────────────────┐    │
│  │      glibc (libc)        │ ←  │ Sjednocuje přístup k syscallům
│  └──────────────────────────┘    │
├──────────────────────────────────┤ ← syscall instrukce (brána do kernelu)
│         Kernel (Ring 0)          │
│  ┌──────────────────────────┐    │
│  │   Syscall handler        │    │
│  ├──────────────────────────┤    │
│  │ Správa procesů (scheduler)│    │
│  │ Správa paměti (MMU)      │    │
│  │ Souborový systém (VFS)   │    │
│  │ Síť (TCP/IP stack)       │    │
│  │ Device drivers           │    │
│  └──────────────────────────┘    │
└──────────────────────────────────┘
```

### Praktické commandy

```bash
# Verze kernelu
cat /proc/version

# Načtené moduly kernelu (monolitické jádro může být modulární)
lsmod | head -20

# Kernel log (co kernel dělal při bootu)
dmesg | head -10

# Hardware informace z kernelu
ls /sys/class/
```

> **ℹ️ POZNÁMKA:** Tato izolace mezi Ring 0 a Ring 3 je základem bezpečnostní architektury. Detaily o uživatelích, skupinách a právech najdete v [kapitole 07 – Bezpečná architektura](07-bezpecna-architektura.md). Správa souborového systému pak navazuje v [kapitole 13 – Správa souborového systému](13-sprava-filesystemu.md).


---

## Systémová volání (syscalls)

### Jak aplikace mluví s kernelem

Program v userspace nemůže přímo přistupovat k hardware ani k paměti kernelu. Když potřebuje otevřít soubor, poslat data po síti nebo vytvořit nový proces, musí požádat kernel — a to pomocí **systémového volání (syscall)**.

### Mechanismus syscallu

Volání printf("hello") v C → glibc zavolá write(1, "hello", 5) → write() je libc wrapper → provede `syscall` instrukci → CPU přepne do Ring 0 → kernel zpracuje požadavek → vrátí výsledek → CPU přepne zpět do Ring 3 → aplikace pokračuje.

Tento přechod není zadarmo — každý syscall stojí ~100–1000 ns (context switch, uložení registrů, kontrola oprávnění).

| Krok | Co se děje |
|------|-----------|
| 1. | Aplikace zavolá funkci z knihovny (např. `fwrite()`) |
| 2. | knihovna (glibc) připraví argumenty a číslo syscallu |
| 3. | `syscall` instrukce — CPU přepne do Ring 0 |
| 4. | Kernel zpracuje požadavek (např. zapíše na disk) |
| 5. | `sysret` instrukce — CPU přepne zpět do Ring 3 |
| 6. | Aplikace dostane výsledek |

### Důležité syscally

| Syscall | Účel | C wrapper | Podrobněji |
|---------|------|-----------|------------|
| `open` | Otevřít soubor | `fopen()` | [Kapitola 13](13-sprava-filesystemu.md) |
| `read` | Číst ze souboru | `fread()` | [Kapitola 13](13-sprava-filesystemu.md) |
| `write` | Zapsat do souboru | `fwrite()` | [Kapitola 13](13-sprava-filesystemu.md) |
| `fork` | Vytvořit nový proces | `fork()` | [Kapitola 06](06-sprava-procesu.md) |
| `execve` | Spustit program | `exec()` | [Kapitola 06](06-sprava-procesu.md) |
| `exit` | Ukončit proces | `exit()` | [Kapitola 06](06-sprava-procesu.md) |
| `socket` | Vytvořit síťové spojení | `socket()` | [Kapitola 16](16-netplan.md) |

> **ℹ️ POZNÁMKA:** Každý syscall má své číslo (např. `write` = 1 na x86_64). Seznam všech najdete v `/usr/include/asm/unistd_64.h` nebo příkazem `man syscalls`.

### Sledování syscallů v praxi — strace

Nejlepší způsob, jak syscally skutečně vidět, je nástroj `strace`:

```bash
# Nainstaluj strace (pokud není)
sudo apt install strace

# Sleduj všechny syscally při spuštění ls
strace ls

# Spočítej syscally — kolik jich ls udělá?
strace -c ls

# Sleduj jen souborové operace
strace -e trace=open,openat,read cat /etc/os-release

# Zjisti, jaký syscall dělá echo
strace echo "hello world" 2>&1 | grep write
```

> **⚠️ VAROVÁNÍ:** `strace` zpomaluje sledovaný program (každý syscall stojí čas). Nepoužívejte na produkčních serverech bez rozmyslu.

### Proč je to důležité

Každá operace, kterou děláte v bashi, Pythonu nebo C, končí jako jeden nebo více syscallů. Když napíšete `cat file.txt`, shell spustí `cat`, ten zavolá `open()` → `read()` → `write()` → `close()`. **Všechno končí u kernelu.**

Podrobněji se správou procesů (fork, exec, signály) zabývá [kapitola 06 – Správa procesů](06-sprava-procesu.md), souborovým systémem [kapitola 13 – Správa souborového systému](13-sprava-filesystemu.md).

---

## Typy operačních systémů podle jádra

Ne všechny operační systémy jsou postaveny stejně. Způsob, jak je kernel navržen, zásadně ovlivňuje stabilitu, výkon a bezpečnost systému.

### Monolitické jádro (Linux)

Celý operační systém — správa procesů, paměti, souborových systémů, ovladače, síťový stack — běží v jediném velkém kernelu v Ring 0.

**Výhody:**
- ✅ Vysoký výkon (vše je v jednom adresním prostoru, žádné zbytečné předávání zpráv)
- ✅ Modulární (loadable kernel modules — `lsmod`)
- ✅ Dobře odladěné za desítky let vývoje

**Nevýhody:**
- ❌ Chybný ovladač může shodit celý systém
- ❌ Obrovská kódová základna (~30M řádků)

Linux je monolitický, ale díky **loadable kernel modules (LKM)** umožňuje přidávat a odebírat ovladače za běhu — není to tedy "monolit" v pejorativním smyslu.

### Mikrojádro (Minix, QNX)

Jádro je co nejmenší — obsahuje jen naprosté minimum (správa procesů, správa paměti, IPC). Ovladače, souborové systémy a síť běží jako samostatné procesy v userspace.

**Výhody:**
- ✅ Stabilita — pád ovladače neshodí celý systém (ovladač se prostě restartuje)
- ✅ Bezpečnost — menší kódová základna = méně chyb
- ✅ Přenositelnost

**Nevýhody:**
- ❌ Pomalejší — komunikace mezi komponentami vyžaduje IPC (message passing)
- ❌ Složitější návrh

### Hybridní jádro (Windows NT)

Kompromis mezi monolitickým a mikrojádrem. Část ovladačů běží v kernelu (kvůli výkonu), část v userspace.

**Výhody:**
- ⚠️ Částečná stabilita mikrojádra + výkon monolitického

**Nevýhody:**
- ⚠️ Složitější architektura než čistě monolitická

### Porovnání

| Vlastnost | Monolitický (Linux) | Mikrojádro (Minix, QNX) | Hybridní (Windows NT) |
|-----------|-------------------|--------------------|--------------------|
| Velikost jádra | Velká | Malá (mikro) | Střední |
| Rychlost | ✅ Vysoká | ❌ Nižší (IPC režie) | ✅ Vysoká |
| Stabilita | ❌ Driver → pád systém | ✅ Driver v userspace | ⚠️ Částečná |
| Modularita | ✅ LKM moduly | ✅ Ano, mimo jádro | ⚠️ Omezená |
| Použití | Servery, desktop, vestavěné | Vestavěné, automotive (QNX) | Desktop, enterprise |

### Praktické commandy

```bash
# Zjisti verzi a typ jádra
uname -a

# Podívej se na načtené moduly (monolithic but modular)
lsmod | head -15

# Informace o jádře
cat /proc/version
```

> **ℹ️ POZNÁMKA:** Volba typu jádra má přímý dopad na bezpečnostní architekturu ([kapitola 07](07-bezpecna-architektura.md)). Například systémy s AppArmorem ([kapitola 15](15-apparmor.md)) využívají Linux Security Modules (LSM) — bezpečnostní framework vestavěný přímo do monolitického jádra Linuxu.

---

## Multitasking a plánování (scheduling)

### Co je multitasking

Na jednojádrovém CPU může v jeden okamžik běžet **pouze jeden proces**. multitasking je iluze — kernel rychle přepíná mezi procesy a vytváří dojem, že běží současně.

### Preemptivní vs kooperativní multitasking

| Typ | Kdo rozhoduje | Použití |
|-----|--------------|---------|
| **Preemptivní** | OS (scheduler) | Linux, Windows, macOS |
| **Kooperativní** | Proces sám | Starší Windows (3.x), `yield()` |

- **Preemptivní** — kernel má kontrolu. Proces dostane časové kvantum, po jehož vypršení mu scheduler sebere CPU a dá ho jinému procesu. Proces to nemůže ovlivnit (kromě snižování priority).
- **Kooperativní** — proces se musí sám vzdát CPU (`yield()`). Pokud jeden proces zacykli, zamrzne celý systém.

### Context switch

Přepnutí z jednoho procesu na druhý není zadarmo. Scheduler musí:

1. Uložit registry právě běžícího procesu do jeho PCB (Process Control Block)
2. Přepnout page table (MMU)
3. Vybrat další proces
4. Obnovit jeho registry
5. Předat mu CPU

Celé to trvá řádově **jednotky mikrosekund** (~1–10 µs). Při tisíci přepnutích za sekundu to není zanedbatelné.

### Linux CFS (Completely Fair Scheduler)

Linux používá od verze 2.6.23 **CFS** — jeho cílem je spravedlivě rozdělit CPU čas mezi všechny běžící procesy.

Klíčové koncepty:
- **Time slice (časové kvantum)** — jak dlouho smí proces běžet, než scheduler zváží přepnutí. V Linuxu typicky ~6–12 ms.
- **Priorita** — procesy s vyšší prioritou (nižší nice hodnota) dostanou víc CPU času.
- **Nice hodnota** — rozsah −20 (nejvyšší priorita) až +19 (nejnižší). Výchozí je 0.

### Praktické commandy

```bash
# Sleduj počet context switchů za sekundu
vmstat 1 5
# Sleduj sloupec "cs" (context switch)

# Zobraz prioritu a nice hodnoty procesů
ps -eo pid,pri,nice,cmd | head -20

# Zjisti scheduling politiku procesu (CFS = SCHED_OTHER)
chrt -p 1

# Zátěžový test — uvidíš multitasking v akci
sudo apt install stress 2>/dev/null
stress --cpu 4 --timeout 5

# Spusť proces s nižší prioritou
nice -n 15 stress --cpu 1 --timeout 5 &

# Zobraz časová kvanta scheduleru
cat /proc/sys/kernel/sched_latency_ns
```

> **⚠️ VAROVÁNÍ:** `stress` vytíží CPU na 100 % — nespouštějte na důležitém serveru bez rozmyslu.

Podrobněji se správou procesů (vytváření, stavy, signály, IPC) zabývá [kapitola 06 – Správa procesů](06-sprava-procesu.md). Dopad plánovače na výkon řeší [kapitola 09 – Optimalizace výkonu](09-optimalizace-vykony.md).

---

## Shrnutí

| Koncept | Klíčová myšlenka | Vyzkoušej |
|---------|------------------|-----------|
| Operační systém | Abstrakce hardware, správce prostředků | `uname -a`, `cat /etc/os-release` |
| Kernel vs userspace | Ring 0 vs Ring 3, izolace a bezpečnost | `cat /proc/version`, `lsmod`, `dmesg` |
| Systémová volání | Brána do kernelu — libc → syscall → kernel | `strace ls`, `strace -c ls` |
| Typy jader | Monolitický (Linux) vs mikrojádro (QNX) vs hybridní (Windows) | `uname -a`, `lsmod` |
| Multitasking | Preemptivní scheduling, časová kvanta, context switch | `vmstat 1`, `ps -eo pri,nice`, `chrt -p 1` |

### Klíčové body k zapamatování

1. **Všechno končí u kernelu** — každá operace (soubor, síť, proces) je syscall.
2. **Userspace je izolovaný** — aplikace nemůže shodit systém (díky Ring 0/3 oddělení).
3. **Linux je monolitický, ale modulární** — ovladače lze načítat za běhu (`lsmod`).
4. **CPU není nikdy skutečně "idle"** — idle proces je prostě proces s nejnižší prioritou.
5. **Sledujte syscally s `strace`** — je to nejlepší nástroj pro pochopení toho, co programy doopravdy dělají.

## Přehled kapitol

Tato kapitola je základem pro všechny ostatní. Zde je mapa, kde jednotlivá témata najdete v praxi:

| Téma | Kapitola | Co se naučíte |
|------|----------|---------------|
| Základní příkazy | [01 – Linux příkazy](01-linux-prikazy-zakladni.md) | Práce v userspace (ls, cd, cp, mv, grep) |
| Procesy | [06 – Správa procesů](06-sprava-procesu.md) | fork, exec, stavy procesů, signály, IPC |
| Bezpečnost | [07 – Bezpečná architektura](07-bezpecna-architektura.md) | Uživatelé, skupiny, firewall, PAM |
| Výkon | [09 – Optimalizace výkonu](09-optimalizace-vykony.md) | CPU, paměť, profilování, tuning |
| Souborový systém | [13 – Správa FS](13-sprava-filesystemu.md) | Disky, oddíly, mount, VFS, syscally |
| MAC | [15 – AppArmor](15-apparmor.md) | Mandatory Access Control v Linuxu |
| Síť | [16 – Netplan](16-netplan.md) | Konfigurace sítě, síťový stack kernelu |

---

➡️ [Zpět na přehled](README.md)
