# Pracovní list: Windows příkazy

> CMD a PowerShell příkazy pro Windows

---

## Zadání

### Úkol 1: CMD — základní příkazy

Otevři CMD (cmd.exe) a spusť:

```cmd
dir
cd %USERPROFILE%
mkdir cviceni
echo Hello > cviceni\hello.txt
type cviceni\hello.txt
```

1. Co vypíše `dir`?
2. Kam se přesuneš pomocí `cd %USERPROFILE%`?
3. Jaký obsah má `hello.txt`?

### Úkol 2: CMD — síť a systém

```cmd
ipconfig
netstat -n
systeminfo | find "OS Name"
```

1. Jakou IP adresu má tvůj počítač?
2. Kolik aktivních TCP spojení vidíš?
3. Jaká verze Windows běží?

### Úkol 3: PowerShell — objekty a pipeline

Otevři PowerShell a spusť:

```powershell
Get-Process | Sort-Object -Property CPU -Descending | Select-Object -First 5
Get-Service | Where-Object {$_.Status -eq "Running"} | Measure-Object
Get-ChildItem C:\ -Directory | Select-Object Name, LastWriteTime
```

1. Který proces spotřebovává nejvíce CPU?
2. Kolik služeb aktuálně běží?
3. Kolik adresářů je v kořeni C:\?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

1. `dir` vypíše obsah aktuálního adresáře (soubory + adresáře + velikost + datum).
2. `cd %USERPROFILE%` přesune do uživatelského profilu (např. `C:\Users\jmeno`).
3. `hello.txt` obsahuje "Hello" (resp. "Hello\n" podle verze CMD).

### Úkol 2 — Řešení

1. `ipconfig` ukáže IPv4 adresu, masku a výchozí bránu pro každou síťovou kartu.
2. Počet aktivních spojení závisí na systému. `netstat -n` ukáže všechny TCP spojení.
3. `systeminfo | find "OS Name"` vypíše název OS (např. "Microsoft Windows 11 Pro").

### Úkol 3 — Řešení

1. Proces s nejvyšším CPU bývá systémový proces nebo aktuální aplikace.
2. Počet běžících služeb závisí na konfiguraci (typicky 100-200).
3. Počet adresářů v `C:\` závisí na instalaci Windows.

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Všechny CMD příkazy proběhly bez chyby
- [ ] `hello.txt` byl vytvořen a obsahuje text
- [ ] `ipconfig` ukázal platnou IP adresu
- [ ] PowerShell příkazy vrátily očekávané objekty
- [ ] Rozumíš rozdílu mezi CMD a PowerShell (objekty vs text)

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../03-windows-prikazy.md) · [Zpět na přehled](../../README.md)
