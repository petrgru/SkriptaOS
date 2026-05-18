# Pracovní list: PowerShell skriptování

> Procvičení PowerShell skriptování — objekty, pipeline, funkce a chybová kontrola.

---

## Zadání

### Úkol 1: Report procesů

Napiš PowerShell skript `process-report.ps1`, který:
1. Získá všechny běžící procesy
2. Vyfiltruje ty s využitím paměti (WorkingSet64) větším než 50 MB
3. Seřadí je sestupně podle paměti
4. Vybere prvních 5
5. Vypíše je jako tabulku se sloupci: `ProcessName`, `PID`, `MemoryMB` (zaokrouhleno na 2 desetinná místa)

### Úkol 2: Organizátor souborů

Vytvoř skript `organizer.ps1`, který:
1. V aktuálním adresáři vytvoří podsložky `Images`, `Documents`, `Archives`, `Other`
2. Přesune soubory podle přípon:
   - `.jpg`, `.png`, `.gif` → `Images/`
   - `.txt`, `.pdf`, `.docx` → `Documents/`
   - `.zip`, `.tar`, `.gz` → `Archives/`
   - vše ostatní → `Other/`
3. Vypíše počet přesunutých souborů v každé kategorii
4. Ošetří případ, kdy složky už existují (nehlásit chybu)

### Úkol 3: Testování dostupnosti serverů

Vytvoř skript `ping-test.ps1`, který:
1. Definuje pole serverů: `google.com`, `github.com`, `localhost`, `neexistujici-domena.xyz`
2. Otestuje každý server pomocí `Test-Connection` (1 ping, quiet)
3. Vytvoří pro každý výsledek objekt s vlastnostmi `Server`, `Reachable` (bool), `Timestamp`
4. Výsledky vypíše jako tabulku
5. Chyby (např. při nedostupnosti) ošetřené přes `try/catch`

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```powershell
Get-Process |
    Where-Object { $_.WorkingSet64 -gt 50MB } |
    Sort-Object WorkingSet64 -Descending |
    Select-Object -First 5 |
    Select-Object ProcessName, Id,
        @{Name="MemoryMB"; Expression={[math]::Round($_.WorkingSet64 / 1MB, 2)}}
```

### Úkol 2 — Řešení

```powershell
$dirs = @{
    Images    = @('.jpg', '.png', '.gif')
    Documents = @('.txt', '.pdf', '.docx')
    Archives  = @('.zip', '.tar', '.gz')
}

foreach ($name in $dirs.Keys) {
    New-Item -Path $name -ItemType Directory -Force | Out-Null
}

$counts = @{}
Get-ChildItem -File | ForEach-Object {
    $ext = $_.Extension.ToLower()
    $target = "Other"
    foreach ($name in $dirs.Keys) {
        if ($ext -in $dirs[$name]) { $target = $name; break }
    }
    Move-Item -Path $_.FullName -Destination $target -Force
    $counts[$target] = $counts[$target] + 1
}

foreach ($cat in $counts.Keys | Sort-Object) {
    Write-Host "$cat`: $($counts[$cat]) souborů"
}
```

### Úkol 3 — Řešení

```powershell
$servers = @("google.com", "github.com", "localhost", "neexistujici-domena.xyz")
$results = @()

foreach ($server in $servers) {
    try {
        $reachable = Test-Connection -ComputerName $server -Count 1 -Quiet -ErrorAction Stop
        $results += [PSCustomObject]@{
            Server    = $server
            Reachable = $reachable
            Timestamp = Get-Date
        }
    } catch {
        $results += [PSCustomObject]@{
            Server    = $server
            Reachable = $false
            Timestamp = Get-Date
        }
    }
}

$results | Format-Table -AutoSize
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] `process-report.ps1` vypíše 5 nejnáročnějších procesů
- [ ] `organizer.ps1` vytvoří složky a přesune soubory
- [ ] `organizer.ps1` nesehlhá, pokud složky už existují
- [ ] `ping-test.ps1` správně vyhodnotí dostupnost serverů
- [ ] Skripty jsou spustitelné v PowerShell (ne v bash)

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../05-powershell-skriptovani.md) · [Zpět na přehled](../../README.md)
