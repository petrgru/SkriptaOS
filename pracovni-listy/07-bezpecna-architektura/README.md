# Pracovní list: Bezpečná architektura OS

> Procvičení souborové bezpečnosti, ACL, SUID/SGID a uživatelských oprávnění.

---

## Zadání

### Úkol 1: Nastavení oprávnění souborů

Vytvoř adresářovou strukturu a nastav práva:

```bash
mkdir -p /tmp/bezpecnost/{data,skripty,logy}
touch /tmp/bezpecnost/data/secret.txt
touch /tmp/bezpecnost/skripty/deploy.sh
touch /tmp/bezpecnost/logy/app.log
```

1. Nastav `deploy.sh` jako spustitelný pro vlastníka a skupinu (`750`)
2. Nastav `secret.txt` pouze pro čtení vlastníka (`400`)
3. Nastav `app.log` pro čtení/zápis vlastníka a čtení skupiny (`640`)
4. Změň vlastníka adresáře `data` na uživatele `nobody` (pokud existuje, jinak `root`)

### Úkol 2: ACL

1. Vytvoř soubor `/tmp/bezpecnost/shared/project.txt`
2. Pomocí ACL přidej uživateli `nobody` právo čtení a zápisu
3. Ověř ACL pomocí `getfacl`
4. Nastav výchozí ACL na adresář `/tmp/bezpecnost/shared/` pro skupinu `users` (čtení a spouštění)
5. Zkus odebrat ACL položku pro uživatele `nobody`

### Úkol 3: SUID a Sticky bit

1. Najdi všechny SUID binárky v systému pomocí `find / -perm -4000`
2. Zjisti, k čemu slouží `/usr/bin/passwd` a proč má SUID bit
3. Vytvoř adresář `/tmp/bezpecnost/tmp_shared` a nastav na něj Sticky bit (`chmod +t`)
4. Ověř, že Sticky bit je nastaven (`ls -ld`)
5. Vysvětli, co Sticky bit chrání a kde se běžně používá

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
mkdir -p /tmp/bezpecnost/{data,skripty,logy}
touch /tmp/bezpecnost/data/secret.txt
touch /tmp/bezpecnost/skripty/deploy.sh
touch /tmp/bezpecnost/logy/app.log

# 1. deploy.sh: rwxr-x--- (750)
chmod 750 /tmp/bezpecnost/skripty/deploy.sh

# 2. secret.txt: r-------- (400)
chmod 400 /tmp/bezpecnost/data/secret.txt

# 3. app.log: rw-r----- (640)
chmod 640 /tmp/bezpecnost/logy/app.log

# 4. Změna vlastníka
sudo chown nobody:nogroup /tmp/bezpecnost/data  # nebo nobody:nobody
# Alternativně: sudo chown root:root /tmp/bezpecnost/data
```

### Úkol 2 — Řešení

```bash
mkdir -p /tmp/bezpecnost/shared
touch /tmp/bezpecnost/shared/project.txt

# Přidání ACL pro nobody
setfacl -m u:nobody:rw /tmp/bezpecnost/shared/project.txt

# Ověření
getfacl /tmp/bezpecnost/shared/project.txt
# # file: project.txt
# user:nobody:rw-
# ...

# Výchozí ACL
setfacl -m d:g:users:rx /tmp/bezpecnost/shared/

# Odebrání ACL
setfacl -x u:nobody /tmp/bezpecnost/shared/project.txt
```

### Úkol 3 — Řešení

```bash
# 1. SUID binárky
find / -perm -4000 -type f 2>/dev/null
# Typický výstup: /usr/bin/su, /usr/bin/sudo, /usr/bin/passwd, /usr/bin/pkexec

# 2. Proč passwd potřebuje SUID?
# passwd musí zapisovat do /etc/shadow, který patří rootovi
# Bez SUID by běžný uživatel nemohl změnit své heslo

# 3. Sticky bit
sudo mkdir /tmp/bezpecnost/tmp_shared
sudo chmod +t /tmp/bezpecnost/tmp_shared
ls -ld /tmp/bezpecnost/tmp_shared
# drwxr-xr-t ... (t na místě x pro ostatní)

# 4. Sticky bit chrání před mazáním cizích souborů
# Použití: /tmp (každý může vytvářet, ale jen vlastník mazat)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Víš, jak nastavit oktálová i symbolická práva (`chmod 755` i `chmod u=rwx,g=rx,o=r`)
- [ ] Umíš přidat a odebrat ACL položku
- [ ] Rozumíš, proč mají binárky jako `passwd` a `sudo` SUID bit
- [ ] Víš, k čemu slouží Sticky bit na `/tmp`

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../07-bezpecna-architektura.md) · [Zpět na přehled](../../README.md)
