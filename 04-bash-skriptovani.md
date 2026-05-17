# Bash skriptování

## Úvod

Bash je výchozí shell pro většinu Linuxových distribucí. Umí spouštět příkazy, pracovat s proměnnými, podmínkami, smyčkami a funkcemi. Tato sekce slouží jako rychlá reference pro studenty operačních systémů.

---

## Základní syntaxe

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Proměnná | Přiřazení hodnoty do jména | `NAME="World"`<br>`echo "Hello $NAME"` | `Hello World` |
| Speciální proměnná | Hodnota z prostředí | `echo $HOME` | `/home/user` |
| Podmínka (if) | Výběr podle hodnoty | `if [ -f file.txt ]; then echo "exists"; fi` | `exists` |
| Podmínka (else) | Alternativní větev | `if [ -f missing.txt ]; then echo "yes"; else echo "no"; fi` | `no` |
| Smyčka (for) | Opakování přes seznam | `for i in 1 2 3; do echo $i; done` | `1`<br>`2`<br>`3` |
| Smyčka (while) | Opakování podle podmínky | `while [ $x -lt 3 ]; do echo $x; x=$((x+1)); done` | `0`<br>`1`<br>`2` |
| Smyčka (until) | Opakování dokud není pravda | `until [ $x -ge 3 ]; do echo $x; x=$((x+1)); done` | `0`<br>`1`<br>`2` |
| Testování souborů | Kontrola vlastností | `[ -d /tmp ] && echo "is directory"` | `is directory` |
| Testování čísel | Porovnání číselných hodnot | `[ 5 -gt 3 ] && echo "bigger"` | `bigger` |
| Testování řetězců | Porovnání textových hodnot | `[ "hello" = "hello" ] && echo "match"` | `match` |

### Příklady použití

```bash
# Vytvoření proměnné a její použití
GREETING="Ahoj"
echo "$GREETING světe"
# Ahoj světe

# Podmínka s více větvemi
SCORE=85
if [ $SCORE -ge 90 ]; then
  echo "Výborně"
elif [ $SCORE -ge 70 ]; then
  echo "Dobře"
else
  echo "Středně"
fi
# Dobře

# Smyčka přes soubory
for file in *.txt; do
  echo "Soubor: $file"
done
# Soubor: a.txt
# Soubor: b.txt

# Čtení řádků ze souboru
while IFS= read -r line; do
  echo "Řádek: $line"
done < input.txt
# Řádek: první
# Řádek: druhý
```

---

## Funkce a argumenty

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Definice funkce | Blok příkazů s názvem | `greet() { echo "Hello $1"; }` | (žádný výstup) |
| Volání funkce | Spuštění s argumentem | `greet World` | `Hello World` |
| Návratová hodnota | Číslo pro volající | `my_func() { return 5; }` | `5` |
| Argumenty funkce | Parametry předané funkci | `show() { echo "První: $1"; }` | `První: Alpha` |
| Všechny argumenty | Seznam všech parametrů | `list() { echo "$@"; }` | `A B C` |
| Počet argumentů | Kolik parametrů bylo | `count() { echo "$#"; }` | `3` |

### Příklady použití

```bash
# Funkce s více argumenty
calculate() {
  local a=$1
  local b=$2
  local op=$3
  case $op in
    add) echo $((a + b));;
    sub) echo $((a - b));;
    mul) echo $((a * b));;
  esac
}

calculate 10 5 add
# 15

calculate 10 5 sub
# 5

# Funkce s výchozí hodnotou
status() {
  local level=${1:-info}
  echo "[$level] Message"
}

status
# [info] Message

status error
# [error] Message
```

---

## Přesměrování a roury

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Standardní výstup | Směrování výstupu do souboru | `ls > output.txt` | (soubor vytvořen) |
| Přidání do výstupu | Přidání na konec souboru | `echo "řádek" >> log.txt` | (přidán řádek) |
| Standardní vstup | Čtení ze souboru | `sort < unsorted.txt` | (seřazený výstup) |
| Chybový výstup | Směrování chyb do souboru | `ls -l missing.txt 2> errors.txt` | (chyba uložena) |
| Roura | Propojení dvou příkazů | `cat file.txt \| grep "hello"` | (filtrovaný výstup) |
| Potlačení výstupu | Zobrazení jen chyb | `command > /dev/null` | (žádný výstup) |
| Spojený výstup | Výstup i chyby do jednoho | `command > all.txt 2>&1` | (vše v jednom souboru) |

### Příklady použití

```bash
# Roura pro řetězení příkazů
cat access.log | grep "200 OK" | wc -l
# 42

# Smíření výstupu a chyb
ls /home /missing 2> errors.txt
# /home: file1 file2
# (chyba z /missing je v errors.txt)

# Použití roury s více příkazy
echo -e "apple\nbanana\ncherry" | sort | head -n 2
# apple
# banana

# Přesměrování výstupu funkce
greet() { echo "Hello $1"; }; greet World > greeting.txt
# (obsah greeting.txt: "Hello World")
```

---

## Chybová hlášení

| Koncept | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| Chybový výstup | Zobrazení chyb na konzoli | `ls -l non_existent.txt` | `ls: non_existent.txt: No such file` |
| Potlačení chyb | Skrytí chybového výstupu | `ls -l missing.txt 2> /dev/null` | (žádný výstup) |
| Návratový kód | Stav ukončení příkazu | `echo $?` | `0` (nebo `1`) |
| Testování návratu | Kontrola úspěchu příkazu | `ls file.txt && echo "OK" || echo "FAIL"` | `OK` nebo `FAIL` |
| Set -e | Automatický odchyl | `set -e` | (příklady viz níže) |
| Trap | Zachycení signálů | `trap 'echo "Cleanup"' EXIT` | `Cleanup` |

### Příklady použití

```bash
# Kontrola návratového kódu
ls /existing_dir
echo "Návratový kód: $?"
# Návratový kód: 0

ls /non_existent_dir
echo "Návratový kód: $?"
# ls: /non_existent_dir: No such file
# Návratový kód: 1

# Podmíněné provedení podle výsledku
ls file.txt && echo "Soubor existuje" || echo "Soubor chybí"
# Soubor existuje

# Použití set -e pro automatický odchyl
set -e
ls /existing_dir  # úspěch
ls /missing_dir   # selhání -> skript se zastaví
echo "Toto se vypíše jen pokud je předchozí úspěšný"

# Trap pro čištění při odchodu
cleanup() {
  echo "Čištění dočasných souborů..."
  rm -f /tmp/temp_file.txt
}
trap cleanup EXIT
echo "Hlavní logika"
# Čištění dočasných souborů...
```

---

## Shrnutí

- **Proměnné a podmínky** jsou základ každého Bash skriptu. Používejte `$VAR` pro přístup k hodnotám a `if/else/fi` pro rozhodování.
- **Smyčky** (`for`, `while`, `until`) umožňují opakování příkazů. Volbu mezi nimi určuje struktura dat.
- **Funkce** dělí skript na přehledné bloky. Argumenty se přistupují přes `$1`, `$2`, `$@`.
- **Přesměrování** (`>`, `>>`, `|`) řídí tok dat mezi příkazy a soubory.
- **Chybová hlášení** a návratové kódy (`$?`) pomáhají sledovat stav skriptu. Používejte `set -e` pro rychlý odchyl a `trap` pro čištění.

Bash skriptování je klíčová dovednost pro práci s Linuxem. Začněte jednoduchými skripty a postupně přidávejte složitější struktury.

---

➡️ [Zpět na přehled](README.md)
