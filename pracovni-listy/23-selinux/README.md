# Pracovní list: SELinux

> Procvičení SELinux režimů, kontextů, politik a troubleshootingu.

---

## Zadání

### Úkol 1: SELinux režimy

1. Zjisti aktuální režim SELinux: `getenforce`
2. Zobraz detailní stav: `sestatus`
3. Zapiš hodnoty:
   - SELinux status:
   - Loaded policy name:
   - Current mode:
   - Mode from config file:
4. Dočasně přepni do permissive režimu: `sudo setenforce 0`
5. Ověř změnu: `getenforce`
6. Přepni zpět do enforcing: `sudo setenforce 1`
7. Jaký je rozdíl mezi režimy enforcing, permissive a disabled?

### Úkol 2: Práce s kontexty

1. Zobraz kontext souboru: `ls -Z /etc/passwd`
2. Zobraz kontext procesu: `ps -eZ | head -5`
3. Najdi kontext pro httpd (Apache): `ps -eZ | grep httpd` (pokud není nainstalován, zkus `grep systemd`)
4. Změň kontext souboru na `httpd_sys_content_t`:
   ```bash
   sudo touch /tmp/test_selinux.html
   sudo chcon -t httpd_sys_content_t /tmp/test_selinux.html
   ls -Z /tmp/test_selinux.html
   ```
5. Vrať kontext zpět: `sudo restorecon /tmp/test_selinux.html`
6. Co znamená formát kontextu `user:role:type:level`? Vysvětli každou část.

### Úkol 3: Troubleshooting SELinux

1. Zkontroluj log deny přístupů: `sudo ausearch -m avc -ts recent`
   (nebo `sudo cat /var/log/audit/audit.log | grep AVC | tail -20`)
2. Pokud auditd není k dispozici: `sudo journalctl | grep -i selinux | tail -20`
3. Vygeneruj politiku pro deníky:
   ```bash
   sudo ausearch -m avc -ts recent | audit2allow -M mypolicy
   ```
4. Aplikuj politiku: `sudo semodule -i mypolicy.pp`
5. K čemu slouží `audit2allow`?
6. Jaký je rozdíl mezi SELinux a AppArmor?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-3. Stav SELinux
getenforce
sestatus
# SELinux status:                 enabled
# Loaded policy name:             targeted
# Current mode:                   enforcing
# Mode from config file:          enforcing

# 4-6. Změna režimu
sudo setenforce 0  # permissive
sudo setenforce 1  # enforcing

# 7. Režimy:
# Enforcing: politika se vynucuje, přístup blokován
# Permissive: porušení se loguje, ale neblokuje (ladění)
# Disabled: SELinux vypnut (vyžaduje reboot)
```

### Úkol 2 — Řešení

```bash
# 1-3. Kontexty
ls -Z /etc/passwd
# system_u:object_r:passwd_file_t:s0 /etc/passwd

ps -eZ | head -5
# unconfined_u:unconfined_r:unconfined_t:s0 ...

# 4-5. Změna kontextu
sudo touch /tmp/test_selinux.html
sudo chcon -t httpd_sys_content_t /tmp/test_selinux.html
ls -Z /tmp/test_selinux.html
sudo restorecon /tmp/test_selinux.html

# 6. Formát kontextu: user:role:type:level
# user = SELinux uživatel (system_u, unconfined_u)
# role = role (object_r pro soubory, system_r pro procesy)
# type = nejdůležitější část — rozhoduje o přístupu
# level = bezpečnostní úroveň (MLS)
```

### Úkol 3 — Řešení

```bash
# 1-2. Deny logy
sudo ausearch -m avc -ts recent 2>/dev/null
# nebo:
sudo journalctl | grep -i selinux | tail -20

# 3-4. Generování a aplikace politiky
sudo ausearch -m avc -ts recent | audit2allow -M mypolicy
sudo semodule -i mypolicy.pp

# 5. audit2allow vytvoří SELinux modul z deny logů
# Používá se pro rychlé řešení blokovaných přístupů

# 6. SELinux vs AppArmor:
# SELinux: label-based (kontexty), složitější, RHEL/Fedora
# AppArmor: path-based (cesty), jednodušší, Ubuntu/Debian
# Oba řeší stejný problém (MAC), ale jiným přístupem
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Víš, jak zjistit aktuální režim SELinuxu
- [ ] Rozumíš formátu kontextu user:role:type:level
- [ ] Umíš změnit kontext a použít restorecon
- [ ] Víš, jak troubleshootovat SELinux deny přes ausearch a audit2allow

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../23-selinux.md) · [Zpět na přehled](../../README.md)
