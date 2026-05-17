# PowerShell skriptování

## Úvod

PowerShell je skriptovací jazyk a shell od Microsoftu postavený na .NET. Na rozdíl od klasických shellů (Bash, CMD) nepracuje s textem, ale s objekty. Každý příkaz (cmdlet) vrací strukturovaná data, která lze dále zpracovávat. Tato sekce slouží jako rychlá reference pro studenty operačních systémů.

---

## Základní syntaxe

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Proměnná | Přiřazení hodnoty | `$name = "World"`<br>`Write-Host "Hello $name"` | `Hello World` |
| Datový typ | Určení typu hodnoty | `$num = 42; $num.GetType()` | `Int32` |
| Číselný typ | Celé číslo / desetinné | `$a = [int]42; $b = [double]3.14` | `42` / `3.14` |
| Řetězec | Textová hodnota | `$text = "Ahoj"; $text.Length` | `4` |
| Pole | Seznam hodnot | `$arr = @(1, 2, 3); $arr[0]` | `1` |
| HashTable | Slovník klíč-hodnota | `$h = @{Name="Petr";Age=22}`<br>`$h["Name"]` | `Petr` |
| Podmínka (if) | Rozhodování podle hodnoty | `if ($x -gt 5) { "velké" }` | `velké` |
| Podmínka (else) | Alternativní větev | `if ($x -gt 5) { "A" } else { "B" }` | `B` |
| Podmínka (elseif) | Více větví | `if ($x -gt 5) { "A" } elseif ($x -gt 2) { "B" }` | `B` |
| Podmínka (switch) | Vícenásobná volba | `switch ($x) { 1 {"one"} 2 {"two"} }` | `one` |
| Smyčka (for) | Počítaná smyčka | `for ($i=0; $i -lt 3; $i++) { $i }` | `0`<br>`1`<br>`2` |
| Smyčka (foreach) | Pro každý prvek | `foreach ($i in @(1,2,3)) { $i * 2 }` | `2`<br>`4`<br>`6` |
| Foreach-Object | Pipeline varianta | `@(1,2,3) \| ForEach-Object { $_ * 2 }` | `2`<br>`4`<br>`6` |
| Smyčka (while) | Opakování podle podmínky | `$i=0; while ($i -lt 3) { $i; $i++ }` | `0`<br>`1`<br>`2` |
| Smyčka (do-while) | Alespoň jedno provedení | `$i=0; do { $i; $i++ } while ($i -lt 3)` | `0`<br>`1`<br>`2` |

### Operátory porovnání

PowerShell používá pojmenované operátory namísto symbolů:

| Operátor | Význam | Příklad | Výstup |
|---------|-------|---------|--------|
| `-eq` | Rovno | `5 -eq 5` | `True` |
| `-ne` | Nerovno | `5 -ne 3` | `True` |
| `-gt` | Větší než | `5 -gt 3` | `True` |
| `-lt` | Menší než | `5 -lt 3` | `False` |
| `-ge` | Větší nebo rovno | `5 -ge 5` | `True` |
| `-le` | Menší nebo rovno | `5 -le 3` | `False` |
| `-like` | Porovnání se zástupným znakem | `"hello" -like "h*"` | `True` |
| `-match` | Porovnání s regulárním výrazem | `"hello" -match "^h"` | `True` |
| `-contains` | Obsahuje prvek | `@(1,2,3) -contains 2` | `True` |

### Příklady použití

```powershell
# Proměnné a datové typy
$name = "Petr"
$age = 22
$height = [double]1.85
Write-Host "$name je $age let a měří $height m"
# Petr je 22 let a měří 1.85 m

# Pole a hashtable
$students = @("Anna", "Petr", "Eva")
$grades = @{
    Anna = 85
    Petr = 92
    Eva  = 78
}
Write-Host "První student: $($students[0])"
Write-Host "Známka Petra: $($grades['Petr'])"
# První student: Anna
# Známka Petra: 92

# Podmínka s více větvemi
$score = 85
if ($score -ge 90) {
    "Výborně"
} elseif ($score -ge 70) {
    "Dobře"
} else {
    "Středně"
}
# Dobře

# Smyčka foreach přes hashtable
$user = @{Name="Petr"; Role="Admin"; OS="Windows"}
foreach ($key in $user.Keys) {
    "$key = $($user[$key])"
}
# Name = Petr
# Role = Admin
# OS = Windows
```

---

## Funkce

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Definice funkce | Blok kódu s názvem | `function Hello { "Hello World" }` | (žádný výstup) |
| Volání funkce | Spuštění funkce | `Hello` | `Hello World` |
| Parametr funkce | Vstupní hodnota | `function Greet { param([string]$n) "Hello $n" }` | (žádný výstup) |
| Volání s parametrem | Předání hodnoty | `Greet -n "Petr"` | `Hello Petr` |
| Typovaný parametr | Omezení typu | `param([int]$a, [int]$b)` | `$a + $b` |
| Návratová hodnota | Výstup z funkce | `function Add { return $a + $b }` | `součet` |
| Výchozí hodnota | Default parametru | `param([string]$n = "Guest")` | `Guest` |
| Vícero hodnot | Funkce vrací více hodnot | `function Calc { $a+$b; $a*$b }` | `součet`<br>`součin` |

### Příklady použití

```powershell
# Funkce s typovanými parametry a výchozí hodnotou
function Get-Power {
    param(
        [int]$Base,
        [int]$Exponent = 2
    )
    $result = [Math]::Pow($Base, $Exponent)
    return $result
}

Get-Power -Base 5 -Exponent 3
# 125

Get-Power -Base 4
# 16

# Funkce vracející vlastní objekt
function New-Student {
    param([string]$Name, [int]$Score)
    [PSCustomObject]@{
        Jmeno = $Name
        Skore = $Score
        Status = if ($Score -ge 60) { "Prošel" } else { "Neprošel" }
    }
}

New-Student -Name "Petr" -Score 85
# Jmeno Skore Status
# ----- ----- ------
# Petr    85 Prošel

# Funkce s polem parametrů
function Get-Sum {
    param([int[]]$Numbers)
    $sum = 0
    foreach ($n in $Numbers) { $sum += $n }
    return $sum
}

Get-Sum -Numbers 1, 2, 3, 4, 5
# 15
```

---

## Objektová struktura

Základní rozdíl mezi PowerShell a tradičními shelly: PowerShell vrací objekty, ne text. Každý objekt má vlastnosti (properties) a metody (methods).

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Get-Member | Zjištění struktury objektu | `Get-Process \| Get-Member` | `TypeName: System.Diagnostics.Process` |
| Vlastnosti | Přístup k datům objektu | `(Get-Process -Id $pid).ProcessName` | `powershell` |
| Metody | Volání akcí na objektu | `(Get-Date).AddDays(7)` | `2026-05-24 12:00:00` |
| Vlastní objekt | Vytvoření objektu | `[PSCustomObject]@{Name="Petr";Role="Admin"}` | `Name Role` |
| Add-Member | Přidání vlastnosti | `$obj \| Add-Member -NotePropertyName "X" -Value 1` | (přidá vlastnost) |
| Select-Object | Výběr vlastností | `Get-Process \| Select-Object Name, CPU` | `Name CPU` |

### Příklady použití

```powershell
# Zjištění typu a vlastností objektu
Get-Process -Id $pid | Get-Member
# TypeName: System.Diagnostics.Process
# Name        MemberType
# ----        ----------
# ProcessName Property
# CPU         Property
# Id          Property
# StartTime   Property
# ...

# Přístup k vlastnostem objektu
$proc = Get-Process -Id $pid
Write-Host "Proces: $($proc.ProcessName), ID: $($proc.Id)"
Write-Host "Spuštěn: $($proc.StartTime)"
Write-Host "Paměť: $([math]::Round($proc.WorkingSet64 / 1MB)) MB"
# Proces: powershell, ID: 12345
# Spuštěn: 5/17/2026 10:30:00
# Paměť: 245 MB

# Vytvoření vlastního objektu
$student = [PSCustomObject]@{
    Jmeno   = "Petr Novák"
    Obor    = "Informatika"
    Ročník  = 2
    Aktivní = $true
}
Write-Host "$($student.Jmeno) - $($student.Obor), $($student.Ročník). ročník"
# Petr Novák - Informatika, 2. ročník

# Kolekce objektů a jejich zpracování
$students = @(
    [PSCustomObject]@{Jmeno="Anna"; Body=85}
    [PSCustomObject]@{Jmeno="Petr"; Body=92}
    [PSCustomObject]@{Jmeno="Eva"; Body=78}
)
$students | ForEach-Object {
    "$($_.Jmeno): $($_.Body) bodů"
}
# Anna: 85 bodů
# Petr: 92 bodů
# Eva: 78 bodů
```

---

## Pipeline

PowerShell pipeline předává objekty (nikoli text). To umožňuje filtrovat, řadit a transformovat data bez parsování textového výstupu.

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Základní pipeline | Předání objektů mezi cmdlety | `Get-Process \| Where-Object { $_.CPU -gt 10 }` | (filtrované procesy) |
| Where-Object | Filtrování podle podmínky | `Get-Service \| Where-Object { $_.Status -eq "Running" }` | (běžící služby) |
| Select-Object | Výběr vlastností | `Get-Process \| Select-Object Name, HandleCount` | `Name HandleCount` |
| Sort-Object | Řazení objektů | `Get-Process \| Sort-Object CPU -Descending` | (seřazené procesy) |
| Group-Object | Seskupení podle hodnoty | `Get-Process \| Group-Object ProcessName` | `Count Name` |
| Measure-Object | Agregace (suma, průměr) | `Get-Process \| Measure-Object WorkingSet -Average` | `Average: 12345678` |
| Select -First | Omezení počtu výsledků | `Get-Process \| Select-Object -First 5` | (prvních 5 procesů) |
| Select -Last | Poslední N výsledků | `Get-Process \| Select-Object -Last 3` | (poslední 3 procesy) |
| Where-Object -Property | Filtrace podle vlastnosti | `Get-Service \| Where-Object Status -eq Running` | (běžící služby) |

### Příklady použití

```powershell
# Filtrování a řazení procesů
Get-Process | Where-Object { $_.CPU -gt 0 } | Sort-Object CPU -Descending | Select-Object -First 5
# NPM(K)  PM(M)   WS(M)  CPU(s)  ProcessName
# ------  -----   -----  ------  -----------
#    50    120     85     45.2   chrome
#    20     45     32     12.5   explorer
# ...

# Řetězení cmdletů
Get-Service |
    Where-Object { $_.Status -eq 'Running' } |
    Select-Object Name, DisplayName, ServiceType |
    Sort-Object Name
# Name            DisplayName                   ServiceType
# ----            -----------                   -----------
# BFE             Base Filtering Engine          Win32OwnProcess
# Dhcp            DHCP Client                    Win32ShareProcess
# ...

# Seskupení a počítání
Get-Process | Group-Object ProcessName | Sort-Object Count -Descending | Select-Object -First 5
# Count Name
# ----- ----
#     9 svchost
#     5 chrome
#     3 SearchIndexer
# ...

# Agregační funkce
Get-Process | Measure-Object WorkingSet64 -Average -Maximum -Minimum
# Average  Maximum   Minimum
# -------  -------   -------
# 45000000 85000000   500000

# Použití $_ (aktuální objekt v pipeline)
1..10 | ForEach-Object { $_ * $_ }
# 1
# 4
# 9
# 16
# 25
# 36
# 49
# 64
# 81
# 100
```

### Cmdlety pro správu systému

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Get-Process | Seznam běžících procesů | `Get-Process -Name notepad` | `Handles NPM(K) PM(K) WS(K) ...` |
| Get-Service | Seznam služeb | `Get-Service \| Where-Object Status -eq Running` | (běžící služby) |
| Get-ChildItem | Seznam souborů (ls) | `Get-ChildItem C:\ -Directory` | (složky v C:\) |
| Get-Content | Obsah souboru (cat) | `Get-Content file.txt` | (obsah souboru) |
| Get-Date | Aktuální datum a čas | `Get-Date -Format "yyyy-MM-dd"` | `2026-05-17` |
| Stop-Process | Ukončení procesu | `Stop-Process -Name notepad -Force` | (ukončí proces) |
| Start-Service | Spuštění služby | `Start-Service Spooler` | (spustí službu) |
| Get-Help | Nápověda k cmdletu | `Get-Help Get-Process -Examples` | (příklady použití) |

---

## Přesměrování

PowerShell podporuje jak klasické přesměrování proudů, tak cmdlety pro export dat.

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Výstup do souboru | Uložení do textového souboru | `Get-Process > proc.txt` | (soubor vytvořen) |
| Přidání do souboru | Apenování na konec | `"řádek" >> log.txt` | (řádek přidán) |
| Chybový výstup | Směrování chyb do souboru | `Get-Item missing.txt 2> err.txt` | (chyba uložena) |
| Varovný výstup | Směrování varování do souboru | `Write-Warning "test" 3> warn.txt` | (varování uloženo) |
| Verbose výstup | Detailní výstup do souboru | `Write-Verbose "detail" -Verbose 4> verbose.txt` | (detail uložen) |
| Out-File | Formátovaný textový výstup | `Get-Process \| Out-File proc.txt` | (formátovaný text) |
| Export-Csv | Export do CSV (objekty) | `Get-Process \| Export-Csv proc.csv` | `Name,CPU,Handles` |
| Export-Clixml | Serializace objektů do XML | `Get-Process \| Export-Clixml proc.xml` | (XML soubor) |
| ConvertTo-Json | JSON řetězec | `Get-Process \| ConvertTo-Json` | `[{"Name":"...","CPU":...}]` |
| ConvertTo-Html | HTML tabulka | `Get-Process \| ConvertTo-Html` | (HTML stránka) |

### Příklady použití

```powershell
# Uložení seznamu procesů do souboru
Get-Process | Select-Object Name, CPU, WorkingSet64 | Export-Csv -Path procesy.csv -NoTypeInformation
# Obsah procesy.csv:
# "Name","CPU","WorkingSet64"
# "chrome","45.2","85000000"
# "explorer","12.5","32000000"

# JSON výstup
Get-Process -Name "notepad" | Select-Object ProcessName, Id, CPU | ConvertTo-Json
# {
#     "ProcessName": "notepad",
#     "Id": 5678,
#     "CPU": 2.34
# }

# Přesměrování chyb
Get-Item "C:\neexistuje.txt" 2> chyby.txt
Get-Content chyby.txt
# Get-Item: Cannot find path 'C:\neexistuje.txt' because it does not exist.

# Out-File s formátováním
Get-Process -Name "explorer" | Out-File -FilePath info.txt -Width 200
# (výstup s plnou šířkou bez ořezávání)
```

---

## Chybová hlášení

PowerShell nabízí robustní mechanismy pro zpracování chyb, včetně strukturovaného try/catch.

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| try/catch | Zachycení chyby | `try { 1/0 } catch { "Chyba: $_" }` | `Chyba: Attempted to divide by zero` |
| finally | Blok provedený vždy | `try { ... } finally { "Cleanup" }` | `Cleanup` |
| $Error | Pole všech chyb v sezení | `$Error[0]` | (poslední chyba) |
| $? | Úspěch posledního příkazu | `$?` | `True` nebo `False` |
| Write-Error | Ruční vyvolání chyby | `Write-Error "Selhalo"` | (červená chyba) |
| $ErrorAction- Preference | Režim zpracování chyb | `$ErrorActionPreference = "Stop"` | (zastaví skript) |
| -ErrorAction | Přepínač pro jeden cmdlet | `Get-Item file.txt -ErrorAction SilentlyContinue` | (potlačí chybu) |
| -ErrorVariable | Uložení chyby do proměnné | `Get-Item file.txt -ErrorVariable err` | (chyba v $err) |
| $_ v catch | Aktuální chyba v bloku | `catch { $_.Exception.Message }` | (text chyby) |
| $LASTEXITCODE | Návratový kód externího programu | `ping.exe localhost; $LASTEXITCODE` | `0` |

### Hodnoty $ErrorActionPreference

| Hodnota | Chování |
|---------|---------|
| `Continue` (výchozí) | Zobrazí chybu a pokračuje |
| `Stop` | Zastaví skript při první chybě |
| `SilentlyContinue` | Potlačí zobrazení chyby |
| `Ignore` | Zcela ignoruje chybu (jen PowerShell 3+) |
| `Inquire` | Zeptá se uživatele, co dělat |

### Příklady použití

```powershell
# try/catch/finally
try {
    $result = 1 / 0
    "Toto se neprovede"
}
catch {
    "Nastala chyba: $($_.Exception.Message)"
}
finally {
    "Toto se provede vždy"
}
# Nastala chyba: Attempted to divide by zero
# Toto se provede vždy

# Použití -ErrorAction Stop pro zachycení chyby
try {
    Get-Item "C:\neexistuje.txt" -ErrorAction Stop
    "Soubor nalezen"
}
catch {
    "Chyba: $($_.Exception.Message)"
}
# Chyba: Cannot find path 'C:\neexistuje.txt' because it does not exist.

# Kontrola úspěchu příkazu
Get-Process -Name "notepad" -ErrorAction SilentlyContinue
if ($?) {
    "Proces notepad běží"
} else {
    "Proces notepad neběží"
}
# (výstup závisí na stavu systému)

# Práce s $Error
# 1..3 | ForEach-Object { "$_" / 0 }  # vyvolá chyby
$Error.Count
# 3
$Error[0] | Select-Object *
# (detail poslední chyby)

# Write-Error a ukončení skriptu
function Test-Value {
    param([int]$Value)
    if ($Value -lt 0) {
        Write-Error "Hodnota nesmí být záporná" -ErrorAction Stop
    }
    "Hodnota: $Value"
}

try {
    Test-Value -Value -5
}
catch {
    "Funkce selhala: $($_.Exception.Message)"
}
# Funkce selhala: Hodnota nesmí být záporná

# Zachycení chyb externích programů
ping.exe -n 1 192.0.2.1
# Odpověď od 192.0.2.1: ...
if ($LASTEXITCODE -eq 0) {
    "Ping úspěšný"
} else {
    "Ping selhal"
}
```

---

## Praktické příklady

### 1. Monitorování procesů

```powershell
# Výpis top 5 procesů podle využití paměti
Get-Process |
    Sort-Object WorkingSet64 -Descending |
    Select-Object -First 5 ProcessName, @{Name="MB";Expression={[math]::Round($_.WorkingSet64/1MB, 2)}} |
    Format-Table -AutoSize

# ProcessName     MB
# -----------   ----
# chrome       145.23
# explorer      85.50
# powershell    62.34
# ...
```

### 2. Správa služeb

```powershell
# Zastavené služby - kontrola
$stopped = Get-Service | Where-Object { $_.Status -eq 'Stopped' -and $_.StartType -eq 'Automatic' }
Write-Host "Počet zastavených automatických služeb: $($stopped.Count)"

# Výpis prvních 5
$stopped | Select-Object -First 5 Name, DisplayName, Status, StartType | Format-Table -AutoSize

# Name            DisplayName                 Status  StartType
# ----            -----------                 ------  ---------
# WSearch         Windows Search              Stopped Automatic
# Spooler         Print Spooler               Stopped Automatic
# ...
```

### 3. Práce se soubory

```powershell
# Hledání velkých souborů
$largeFiles = Get-ChildItem C:\Temp -Recurse -File -ErrorAction SilentlyContinue |
    Where-Object { $_.Length -gt 10MB } |
    Sort-Object Length -Descending |
    Select-Object -First 5 FullName, @{Name="MB";Expression={[math]::Round($_.Length/1MB, 2)}}

$largeFiles | Format-Table -AutoSize
# FullName                                MB
# --------                              ----
# C:\Temp\backup.iso                   450.00
# C:\Temp\dump.dmp                     120.50
# ...

# Počet souborů podle typu
Get-ChildItem C:\Temp -Recurse -File -ErrorAction SilentlyContinue |
    Group-Object Extension |
    Sort-Object Count -Descending |
    Select-Object -First 5 Count, Name

# Count Name
# ----- ----
#   120 .log
#    85 .txt
#    45 .tmp
# ...
```

### 4. Síťová diagnostika

```powershell
# Aktivní TCP spojení
Get-NetTCPConnection |
    Where-Object State -eq Established |
    Group-Object RemotePort |
    Sort-Object Count -Descending |
    Select-Object -First 5 Count, Name

# Count Name
# ----- ----
#     8 443
#     3 80
#     1 22
# ...

# Test dostupnosti více serverů
$servers = @("google.com", "github.com", "neexistujici-domena.xyz")
foreach ($server in $servers) {
    $result = Test-Connection -ComputerName $server -Count 1 -Quiet -ErrorAction SilentlyContinue
    if ($result) {
        Write-Host "[OK] $server je dostupný" -ForegroundColor Green
    } else {
        Write-Host "[FAIL] $server není dostupný" -ForegroundColor Red
    }
}
# [OK] google.com je dostupný
# [OK] github.com je dostupný
# [FAIL] neexistujici-domena.xyz není dostupný
```

### 5. Generování reportu do CSV

```powershell
# Komplexní report systému
$report = @(
    [PSCustomObject]@{
        Kategorie   = "System"
        Property    = "Hostname"
        Value       = $env:COMPUTERNAME
    }
    [PSCustomObject]@{
        Kategorie   = "System"
        Property    = "OS"
        Value       = (Get-CimInstance Win32_OperatingSystem).Caption
    }
    [PSCustomObject]@{
        Kategorie   = "CPU"
        Property    = "Usage"
        Value       = "$((Get-CimInstance Win32_Processor).LoadPercentage)%"
    }
    [PSCustomObject]@{
        Kategorie   = "Memory"
        Property    = "Total GB"
        Value       = "$([math]::Round((Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory/1GB, 2)) GB"
    }
)

$report | Export-Csv -Path system-report.csv -NoTypeInformation -Encoding UTF8
Write-Host "Report uložen do system-report.csv"
Get-Content system-report.csv
# "Kategorie","Property","Value"
# "System","Hostname","DESKTOP-ABC123"
# "System","OS","Microsoft Windows 11 Pro"
# "CPU","Usage","15%"
# "Memory","Total GB","16.00 GB"
```

### 6. Skript s argumenty a chybovou kontrolou

```powershell
<#
.SYNOPSIS
    Skript pro zálohování složky
.PARAMETER Source
    Zdrojová složka
.PARAMETER Destination
    Cílová složka
#>

param(
    [Parameter(Mandatory=$true)]
    [string]$Source,

    [Parameter(Mandatory=$true)]
    [string]$Destination,

    [switch]$Compress
)

try {
    # Kontrola existence zdroje
    if (-not (Test-Path $Source)) {
        throw "Zdrojová složka '$Source' neexistuje"
    }

    # Vytvoření cílové složky
    if (-not (Test-Path $Destination)) {
        New-Item -Path $Destination -ItemType Directory -Force | Out-Null
        Write-Host "Vytvořena cílová složka: $Destination"
    }

    # Kopírování souborů
    $files = Get-ChildItem $Source -File
    Write-Host "Kopíruji $($files.Count) souborů..."
    foreach ($file in $files) {
        $target = Join-Path $Destination $file.Name
        Copy-Item -Path $file.FullName -Destination $target -Force
        Write-Host "  OK: $($file.Name)"
    }

    # Volitelná komprese
    if ($Compress) {
        $zipPath = "$Destination.zip"
        Compress-Archive -Path $Destination -DestinationPath $zipPath -Force
        Write-Host "Složka komprimována do: $zipPath"
    }

    Write-Host "Zálohování dokončeno."
}
catch {
    Write-Error "Zálohování selhalo: $($_.Exception.Message)"
    exit 1
}
```

---

## Shrnutí

- **Základní syntaxe** používá `$proměnné`, `@()` pro pole a `@{}` pro hashtables. Podmínky používají pojmenované operátory (`-eq`, `-gt`, `-like`).
- **Funkce** se definují klíčovým slovem `function`. Parametry se typují pomocí `[typ]$nazev`. Návratová hodnota je vše, co funkce vypíše.
- **Objekty** jsou základem PowerShellu. Každý cmdlet vrací objekty s vlastnostmi a metodami. `Get-Member` odhalí strukturu libovolného objektu.
- **Pipeline** (`|`) předává objekty mezi cmdlety. `Where-Object` filtruje, `Select-Object` vybírá vlastnosti, `Sort-Object` řadí.
- **Přesměrování** (`>`, `>>`, `2>`) funguje stejně jako v jiných shellech. `Export-Csv`, `ConvertTo-Json` a `Out-File` poskytují pokročilý export.
- **Chyby** se ošetřují přes `try/catch/finally`. `$ErrorActionPreference` řídí chování při chybě. `-ErrorAction` přepisuje chování pro jeden příkaz.

PowerShell je mnohem víc než shell. Je to plnohodnotný skriptovací jazyk s přístupem k .NET knihovnám, WMI a REST API. Začněte jednoduchými cmdlety a postupně přidávejte pipeline, vlastní objekty a chybovou kontrolu.

---

➡️ [Zpět na přehled](README.md)
