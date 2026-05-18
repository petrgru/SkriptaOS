# Pracovní list: Práva souborů a adresářů

> Procvičení SUID, SGID, Sticky bit, ACL, umask a capabilities v Linuxu.

---

## Zadání

### Úkol 1: Speciální bity (SUID, SGID, Sticky)

1. Vytvoř adresářovou strukturu: `mkdir -p /tmp/prava/{bin,shared,tmp}`
2. Vytvoř soubor `/tmp/prava/bin/skript.sh` s právy `755`
3. Nastav na něj SUID bit: `chmod u+s /tmp/prava/bin/skript.sh`
4. Ověř SUID bit pomocí `ls -la` — jak vypadá?
5. Nastav na `/tmp/prava/shared` SGID bit
6. Vytvoř v něm soubor a ověř, že dědí skupinu adresáře
7. Nastav Sticky bit na `/tmp/prava/tmp`
8. Najdi všechny SUID soubory v systému (`find / -perm -4000`) a zapiš 3 příklady

### Úkol 2: ACL

1. Vytvoř adresář `/tmp/prava/acl_test` a soubor `data.txt`
2. Přidej ACL pro uživatele `nobody` s právy rwx
3. Přidej ACL pro skupinu `users` s právy rx
4. Zobraz aktuální ACL pomocí `getfacl`
5. Nastav výchozí ACL (default) na adresář pro skupinu `developers` s právy rwx
6. Odeber ACL pro uživatele `nobody`
7. Smaž všechny ACL ze souboru (`setfacl -b`)

### Úkol 3: umask a capabilities

1. Zjisti aktuální umask (`umask`)
2. Vypočítej výsledná oprávnění:
   - umask 077 → nový soubor: ?  nový adresář: ?
   - umask 022 → nový soubor: ?  nový adresář: ?
   - umask 002 → nový soubor: ?  nový adresář: ?
3. Dočasně nastav umask na `077` a vytvoř soubor — ověř jeho práva
4. Zjisti capabilities procesu: `getcap /usr/bin/ping` — co znamená `cap_net_raw`?
5. Jaký je rozdíl mezi SUID a capabilities? Který je bezpečnější?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
mkdir -p /tmp/prava/{bin,shared,tmp}
touch /tmp/prava/bin/skript.sh

chmod 755 /tmp/prava/bin/skript.sh
chmod u+s /tmp/prava/bin/skript.sh
# SUID: v ls -la je 's' místo 'x' u vlastníka: -rwsr-xr-x

chmod g+s /tmp/prava/shared
touch /tmp/prava/shared/novy_soubor
ls -la /tmp/prava/shared/novy_soubor
# Skupina bude stejná jako u adresáře shared

chmod +t /tmp/prava/tmp
# Sticky bit: 't' místo 'x' u ostatních: drwxr-xr-t

find / -perm -4000 -type f 2>/dev/null
# /usr/bin/su, /usr/bin/sudo, /usr/bin/passwd
```

### Úkol 2 — Řešení

```bash
mkdir -p /tmp/prava/acl_test
touch /tmp/prava/acl_test/data.txt

setfacl -m u:nobody:rwx /tmp/prava/acl_test/data.txt
setfacl -m g:users:rx /tmp/prava/acl_test/data.txt
getfacl /tmp/prava/acl_test/data.txt

setfacl -m d:g:developers:rwx /tmp/prava/acl_test

setfacl -x u:nobody /tmp/prava/acl_test/data.txt
setfacl -b /tmp/prava/acl_test/data.txt
```

### Úkol 3 — Řešení

```bash
# 1. Aktuální umask
umask  # typicky 0022

# 2. Výpočet
# umask 077: soubor = 600 (-rw-------), adresář = 700 (drwx------)
# umask 022: soubor = 644 (-rw-r--r--), adresář = 755 (drwxr-xr-x)
# umask 002: soubor = 664 (-rw-rw-r--), adresář = 775 (drwxrwxr-x)

# 3. Dočasné nastavení
umask 077
touch /tmp/prava/test_umask
ls -la /tmp/prava/test_umask  # -rw-------
umask 022  # vrátit zpět

# 4. Capabilities
getcap /usr/bin/ping
# /usr/bin/ping = cap_net_raw+ep
# cap_net_raw umožňuje ping otevřít raw socket (posílat ICMP)
# Bez této capability by ping potřeboval SUID nebo root

# 5. Capabilities jsou bezpečnější než SUID
# SUID: celý proces běží jako root -> velký prostor pro útok
# capabilities: jen specifická oprávnění (např. jen raw socket)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Víš, jak nastavit SUID, SGID a Sticky bit
- [ ] Umíš přidat, zobrazit a odebrat ACL
- [ ] Dokážeš vypočítat výsledná práva z umasku
- [ ] Rozumíš rozdílu mezi SUID a capabilities

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../12-prava-souboru.md) · [Zpět na přehled](../../README.md)
