# Pracovní list: Správa uživatelů a skupin

> Procvičení vytváření a správy uživatelů, skupin a oprávnění v Linuxu.

---

## Zadání

### Úkol 1: Vytvoření uživatelů

1. Vytvoř uživatele `student1` s domovským adresářem a bashem jako shellem
2. Vytvoř uživatele `student2` s UID 1050 a přidej ho do skupiny `sudo`
3. Nastav oběma uživatelům heslo (stačí zapsat příkaz)
4. Zamkni účet `student2` (bez mazání)
5. Vytvoř systémového uživatele `backup` bez domovského adresáře a shellu
6. Zkontroluj, že uživatelé existují v `/etc/passwd`

### Úkol 2: Správa skupin

1. Vytvoř skupiny `developers` a `devops`
2. Přidej uživatele `student1` do skupiny `developers`
3. Přidej uživatele `student2` do obou skupin najednou
4. Zobraz skupiny, ve kterých je `student1`
5. Odeber `student2` ze skupiny `devops`
6. Vytvoř skupinu `project-alpha` s GID 3000

### Úkol 3: Sudo a práva

1. Zobraz obsah `/etc/sudoers` (pomocí `sudo visudo -c` pro kontrolu syntaxe, nebo `cat`)
2. Vytvoř soubor `/etc/sudoers.d/developers` s pravidlem: skupina `developers` může spouštět všechny příkazy bez hesla
3. Jaké je správné oprávnění pro soubory v `/etc/sudoers.d/`?
4. Vytvoř alias pro skupinu v sudoers: `Cmnd_Alias BACKUP = /usr/bin/rsync, /usr/bin/tar` a povol skupině `backup` tyto příkazy
5. Smaž všechny vytvořené uživatele i s domovskými adresáři

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-2. Vytvoření uživatelů
sudo useradd -m -s /bin/bash student1
sudo useradd -m -u 1050 -G sudo -s /bin/bash student2

# 3. Nastavení hesel
sudo passwd student1
sudo passwd student2

# 4. Zamknutí účtu
sudo passwd -l student2

# 5. Systémový uživatel
sudo useradd -r -s /usr/sbin/nologin backup

# 6. Ověření
tail -5 /etc/passwd
```

### Úkol 2 — Řešení

```bash
# 1. Vytvoření skupin
sudo groupadd developers
sudo groupadd devops

# 2-3. Přidání do skupin
sudo gpasswd -a student1 developers
sudo usermod -aG developers,devops student2

# 4. Zobrazení skupin
groups student1
# student1 : student1 developers

# 5. Odebrání
sudo gpasswd -d student2 devops

# 6. Skupina s GID
sudo groupadd -g 3000 project-alpha
```

### Úkol 3 — Řešení

```bash
# 1. Ověření sudoers
sudo visudo -c  # kontrola syntaxe

# 2. Pravidlo pro skupinu
echo '%developers ALL=(ALL) NOPASSWD: ALL' | sudo tee /etc/sudoers.d/developers

# 3. Oprávnění: 440 (vlastník root, skupina root)
sudo chmod 440 /etc/sudoers.d/developers

# 4. Alias v sudoers
echo 'Cmnd_Alias BACKUP = /usr/bin/rsync, /usr/bin/tar
%backup ALL=(root) BACKUP' | sudo tee /etc/sudoers.d/backup

# 5. Smazání uživatelů
sudo userdel -r student1
sudo userdel -r student2
sudo userdel -r backup
# Smazání skupin
sudo groupdel developers
sudo groupdel devops
sudo groupdel project-alpha
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš vytvořit uživatele s různými parametry (shell, skupiny, UID)
- [ ] Dokážeš spravovat skupiny a přiřazovat uživatele
- [ ] Víš, jak správně vytvořit sudoers pravidlo
- [ ] Rozumíš principu systémových uživatelů (bez shellu)

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../11-sprava-uzivatelu.md) · [Zpět na přehled](../../README.md)
