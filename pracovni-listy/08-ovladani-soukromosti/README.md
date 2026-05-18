# Pracovní list: Správa logů a prostředí

> Procvičení práce s logy, systémovými proměnnými, SSH a GPG klíči.

---

## Zadání

### Úkol 1: Práce s logy

1. Zobraz posledních 10 řádků z `/var/log/syslog` (nebo `/var/log/messages`)
2. Vyfiltruj řádky obsahující "error" (case-insensitive) z posledních 50 řádků logu
3. Zjisti, kolik řádků přibylo do syslogu za posledních 5 minut (použij `journalctl --since`)
4. Najdi soubory v `/var/log/` větší než 100 MB

### Úkol 2: Prostředí a proměnné

1. Vypiš všechny proměnné prostředí a najdi hodnotu `$HOME`, `$USER`, `$SHELL`
2. Dočasně nastav proměnnou `MY_VAR="Testovaci hodnota"` a ověř, že existuje
3. Spusť `bash -c 'echo $MY_VAR'` — proč je výstup prázdný?
4. Nastav `MY_VAR` jako exportovanou proměnnou a zopakuj krok 3
5. Trvale přidej `MY_VAR` do `~/.bashrc` (stačí napsat, jak bys to udělal)

### Úkol 3: SSH audit

1. Zkontroluj oprávnění adresáře `~/.ssh` — jaká mají být?
2. Pokud existuje `~/.ssh/id_ed25519` nebo `~/.ssh/id_rsa`, zkontroluj jeho oprávnění
3. Vygeneruj nový SSH pár (Ed25519) do `/tmp/test_key` (bez passphrase)
4. Zobraz veřejný klíč a vysvětli jednotlivé části (typ, klíč, komentář)
5. Proč musí mít `~/.ssh/` oprávnění 700 a klíče 600?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1. Posledních 10 řádků
tail -10 /var/log/syslog
# nebo: journalctl -n 10 --no-pager

# 2. Filtrování error
tail -50 /var/log/syslog | grep -i error

# 3. Za posledních 5 minut
journalctl --since "5 minutes ago" --no-pager | wc -l

# 4. Soubory nad 100 MB
find /var/log -type f -size +100M -ls 2>/dev/null
```

### Úkol 2 — Řešení

```bash
# 1. Výpis proměnných
env | grep -E '^(HOME|USER|SHELL)='
# Výstup: HOME=/home/user, USER=petrg, SHELL=/bin/bash

# 2. Dočasná proměnná
MY_VAR="Testovaci hodnota"
echo $MY_VAR  # Testovaci hodnota

# 3. bash -c nevidí MY_VAR, protože není exportovaná
bash -c 'echo $MY_VAR'  # (prázdný výstup)

# 4. Export
export MY_VAR
bash -c 'echo $MY_VAR'  # Testovaci hodnota

# 5. Trvalé přidání
echo 'export MY_VAR="Testovaci hodnota"' >> ~/.bashrc
# Poté: source ~/.bashrc
```

### Úkol 3 — Řešení

```bash
# 1. Kontrola oprávnění
ls -la ~/.ssh/
# Adresář musí mít 700, klíče 600, veřejné klíče 644

# 2. Oprava oprávnění
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519 ~/.ssh/authorized_keys 2>/dev/null
chmod 644 ~/.ssh/*.pub ~/.ssh/known_hosts 2>/dev/null

# 3. Generování nového klíče
ssh-keygen -t ed25519 -f /tmp/test_key -N "" -C "test@example.com"

# 4. Zobrazení veřejného klíče
cat /tmp/test_key.pub
# ssh-ed25519 AAAAC3... comment@email.com
# Části: typ (ed25519) | veřejný klíč (base64) | komentář

# 5. Proč restriktivní oprávnění?
# SSH klient odmítne pracovat, pokud mají soubory volnější práva
# Chrání soukromý klíč před krádeží jiným uživatelem
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš filtrovat logy pomocí `grep` a `journalctl`
- [ ] Rozumíš rozdílu mezi lokální a exportovanou proměnnou
- [ ] Víš, jaká oprávnění má mít `~/.ssh/` a proč
- [ ] Dokážeš vygenerovat SSH klíč a vysvětlit jeho strukturu

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../08-ovladani-soukromosti.md) · [Zpět na přehled](../../README.md)
