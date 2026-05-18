# Pracovní list: Bezpečnostní audit

> Procvičení kontroly integrity souborů, detekce rootkitů, AIDE a bezpečnostního auditu.

---

## Zadání

### Úkol 1: Kontrolní součty (checksumy)

1. Vytvoř soubor `test.txt` s libovolným obsahem
2. Vytvoř SHA-256 hash souboru: `sha256sum test.txt > test.txt.sha256`
3. Ověř hash: `sha256sum -c test.txt.sha256`
4. Změň obsah souboru a znovu ověř hash — co se stane?
5. Vytvoř baseline pro více souborů:
   ```bash
   sha256sum /etc/passwd /etc/shadow /etc/group > baseline.sha256
   sha256sum -c baseline.sha256
   ```
6. K čemu slouží kontrola integrity pomocí hashů?

### Úkol 2: Detekce rootkitů a AIDE

1. Nainstaluj `rkhunter` a `chkrootkit`:
   ```bash
   sudo apt install rkhunter chkrootkit
   ```
2. Spusť kontrolu rootkitů:
   ```bash
   sudo rkhunter --check --skip-keypress
   ```
   (Pouze informativně — na ostrém systému)
3. Nainstaluj AIDE: `sudo apt install aide`
4. Inicializuj AIDE databázi (baseline):
   ```bash
   sudo aideinit
   # Vytvoří /var/lib/aide/aide.db.new
   ```
5. Přejmenuj databázi: `sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db`
6. Spusť kontrolu: `sudo aide --check`
7. K čemu AIDE slouží a jak často bys měl kontrolu spouštět?

### Úkol 3: Ověření balíčků a automatizace

1. Ověř integritu nainstalovaných balíčků:
   ```bash
   sudo debsums -s 2>/dev/null
   ```
   (Pokud `debsums` není, nainstaluj ho)
2. Zkontroluj, zda nějaký balíček změnil své soubory
3. Najdi podezřelé soubory:
   - SUID soubory: `find / -perm -4000 -type f 2>/dev/null`
   - Soubory bez vlastníka: `find / -nouser -o -nogroup 2>/dev/null`
   - World-writable soubory v /etc: `find /etc -type f -perm -o+w 2>/dev/null`
4. Vytvoř jednoduchý cron skript pro pravidelný audit:
   ```bash
   #!/bin/bash
   aide --check > /var/log/audit/$(date +%Y%m%d).log
   rkhunter --check --skip-keypress >> /var/log/audit/$(date +%Y%m%d).log
   ```
5. Jaké další bezpečnostní kontroly bys doporučil?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-3. Checksumy
echo "Testovaci data" > test.txt
sha256sum test.txt > test.txt.sha256
sha256sum -c test.txt.sha256
# test.txt: OK

# 4. Po změně souboru
echo "Zmenena data" > test.txt
sha256sum -c test.txt.sha256
# test.txt: FAILED

# 5. Baseline více souborů
sha256sum /etc/passwd /etc/shadow /etc/group > baseline.sha256
sha256sum -c baseline.sha256

# 6. Kontrola integrity odhalí neoprávněné změny souborů
```

### Úkol 2 — Řešení

```bash
# 1. Instalace
sudo apt install rkhunter chkrootkit aide

# 2. Rootkit kontrola
sudo rkhunter --check --skip-keypress

# 3-6. AIDE (Advanced Intrusion Detection Environment)
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
sudo aide --check

# 7. AIDE vytvoří baseline (databázi hashů) a pak kontroluje změny
# Doporučená frekvence: denně (cron), týdně na méně kritických systémech
```

### Úkol 3 — Řešení

```bash
# 1-2. Kontrola balíčků
sudo apt install debsums
sudo debsums -s  # jen změněné soubory

# 3. Podezřelé soubory
find / -perm -4000 -type f 2>/dev/null
find / -nouser -o -nogroup 2>/dev/null
find /etc -type f -perm -o+w 2>/dev/null

# 4. Cron skript pro audit
sudo mkdir -p /var/log/audit
sudo tee /etc/cron.daily/security-audit << 'EOF'
#!/bin/bash
aide --check > /var/log/audit/$(date +%Y%m%d).log
rkhunter --check --skip-keypress >> /var/log/audit/$(date +%Y%m%d).log
EOF
sudo chmod +x /etc/cron.daily/security-audit

# 5. Další kontroly:
# - Pravidelný audit logů (logwatch)
# - Sledování neúspěšných přihlášení (fail2ban)
# - Lynis (bezpečnostní audit)
# - Ověření firewall pravidel
# - Kontrola otevřených portů (nmap)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš vytvořit a ověřit SHA-256 hash
- [ ] Víš, jak nainstalovat a spustit AIDE
- [ ] Dokážeš najít SUID soubory, orphaned soubory a world-writable files
- [ ] Znáš nástroje pro detekci rootkitů (rkhunter, chkrootkit)

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../25-bezpecnostni-audit.md) · [Zpět na přehled](../../README.md)
