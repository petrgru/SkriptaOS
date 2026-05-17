# Windows příkazy

## Úvod

Tato sekce pokrývá základní příkazy pro příkazový řádek Windows (CMD) a základní systémové nástroje. Každý příkaz je popsán s příkladem použití a ukázkou výstupu.

---

## Základní příkazy

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `dir` | Seznam souborů a složek | `dir /s` | `01/01/2024 12:00 1,024 file.txt` |
| `cd` | Změna aktuální složky | `cd C:\Users\petr` | `C:\Users\petr>` |
| `md` | Vytvoření nové složky | `md Documents\Projects` | `Documents\Projects>` |
| `rd` | Smazání složky | `rd /s /q Temp` | `Temp>` |
| `copy` | Kopírování souboru | `copy file.txt backup.txt` | `1 file(s) copied.` |
| `move` | Přesunutí souboru | `move file.txt folder\` | `1 file(s) moved.` |
| `del` | Smazání souboru | `del /f file.txt` | `C:\Users\petr\file.txt` |
| `cls` | Vyčištění obrazovky | `cls` | *(prázdná obrazovka)* |
| `exit` | Ukončení příkazového řádku | `exit` | `C:\Users\petr>` |

---

## Textové soubory

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `type` | Zobrazení obsahu souboru | `type file.txt` | `obsah souboru` |
| `echo` | Vytisknutí textu nebo vytvoření souboru | `echo Hello > hello.txt` | *(vytvoří soubor s textem)* |
| `findstr` | Hledání textu v souboru | `findstr "text" file.txt` | `text nalezen v řádku 1` |
| `sort` | Seřazení obsahu souboru | `sort file.txt` | `řádek 1`<br>`řádek 2`<br>`řádek 3` |
| `more` | Zobrazení obsahu po stránkách | `more file.txt` | *(stránkování obsahu)* |
| `comp` | Porovnání dvou souborů | `comp file1.txt file2.txt` | `File1 and File2 are identical` |

---

## Síťová připojení

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `ping` | Kontrola síťového připojení | `ping google.com` | `Reply from 142.250.190.4: bytes=32 time=15ms TTL=58` |
| `ipconfig` | Zobrazení IP konfigurace | `ipconfig /all` | `IPv4 Address. . . . . . : 192.168.1.100` |
| `netstat` | Stav síťových připojení | `netstat -an` | `TCP 192.168.1.100:54321 142.250.190.4:443 ESTABLISHED` |
| `route` | Správa směrovací tabulky | `route print` | `Network Destination 0.0.0.0  Netmask 0.0.0.0  Gateway 192.168.1.1` |
| `tracert` | Stopa k cílovému hostiteli | `tracert google.com` | `1  15ms  192.168.1.1`<br>`2  20ms  142.250.190.4` |
| `nslookup` | Dotaz na DNS záznam | `nslookup google.com` | `Name: google.com`<br>`Address: 142.250.190.4` |

---

## Systémové informace

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `systeminfo` | Detailní informace o systému | `systeminfo` | `Host Name: DESKTOP-ABC123`<br>`OS Name: Microsoft Windows 11 Pro` |
| `tasklist` | Seznam běžících procesů | `tasklist` | `Image Name  PID  Mem Usage`<br>`explorer.exe  1234  150,000 K` |
| `taskkill` | Ukončení procesu | `taskkill /IM notepad.exe /F` | `SUCCESS: The process notepad.exe with PID 5678 has been terminated.` |
| `wmic` | Správa prostředků přes WMI | `wmic cpu get name, numberofcores` | `Name  NumberOfCores`<br>`Intel Core i7  8` |
| `ver` | Verze systému | `ver` | `Microsoft Windows [Version 10.0.22621]` |
| `hostname` | Název počítače | `hostname` | `DESKTOP-ABC123` |

---

## Shrnutí

- **Základní příkazy** (`dir`, `cd`, `md`, `rd`, `copy`, `move`, `del`) pokrývají většinu každodenních úkolů souborového systému.
- **Textové soubory** (`type`, `echo`, `findstr`, `sort`) umožňují rychlou práci s obsahem bez plného editoru.
- **Síťová připojení** (`ping`, `ipconfig`, `netstat`, `route`) jsou klíčová pro diagnostiku sítě.
- **Systémové informace** (`systeminfo`, `tasklist`, `wmic`) poskytují přehled o stavu systému a běžících procesech.
- Příkazy v CMD jsou citlivé na velká písmena (např. `dir` vs `DIR`), ale většina příkazů funguje oběma způsoby.
- Použijte `příkaz /?` pro rychlý přehled možností jakéhokoli příkazu.

---

➡️ [Zpět na přehled](README.md)
