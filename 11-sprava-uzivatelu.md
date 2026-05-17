# Správa uživatelů a skupin

> User and Group Management in Linux

---

## Úvod

> **Související:** Základní přehled uživatelů a skupin viz [07 - Bezpečná architektura](07-bezpecna-architektura.md)

Správa uživatelů a skupin je základní dovedností každého administrátora Linuxu. Tato příručka pokrývá vytváření a úpravu uživatelských účtů, správu skupin, systémové soubory pro ukládání identit, práva superuživatele a nástroje pro získávání informací o uživatelích.

---

## 1. Vytváření a správa uživatelů

Základní nástroje pro vytváření, úpravu a mazání uživatelských účtů.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `useradd -m` | Vytvoří uživatele s domovským adresářem | `sudo useradd -m -s /bin/bash -c "Alice Smith" alice` | (žádný výstup) |
| `useradd -G` | Vytvoří uživatele a přidá do skupin | `sudo useradd -m -G sudo,docker alice` | (žádný výstup) |
| `usermod -aG` | Přidá uživatele do doplňkových skupin | `sudo usermod -aG docker alice` | (žádný výstup) |
| `usermod -l` | Změní přihlašovací jméno | `sudo usermod -l alice-new alice` | (žádný výstup) |
| `usermod -d` | Změní domovský adresář | `sudo usermod -d /home/alice-new -m alice` | (žádný výstup) |
| `userdel` | Smaže uživatelský účet | `sudo userdel alice` | (žádný výstup) |
| `userdel -r` | Smaže uživatele i s domovským adresářem | `sudo userdel -r alice` | (žádný výstup) |
| `passwd` | Nastaví nebo změní heslo | `sudo passwd alice` | `New password:`<br>`Retype new password:`<br>`passwd: password updated successfully` |
| `passwd -l` | Zamkne účet (uzamkne heslo) | `sudo passwd -l alice` | `passwd: password expiry information changed.` |
| `passwd -u` | Odemkne účet | `sudo passwd -u alice` | `passwd: password expiry information changed.` |
| `passwd -d` | Smaže heslo (účet bez hesla) | `sudo passwd -d alice` | `passwd: password expiry information changed.` |
| `passwd -e` | Vynutí změnu hesla při příštím přihlášení | `sudo passwd -e alice` | `passwd: password expiry information changed.` |

```bash
# Vytvoření uživatele s konkrétním UID a primární skupinou
sudo useradd -m -u 1050 -g developers -s /usr/bin/zsh -c "Bob Developer" bob

# Přidání do více skupin najednou
sudo usermod -aG sudo,developers,docker bob

# Uzamčení a odemčení účtu
sudo passwd -l bob    # zamknout
sudo passwd -u bob    # odemknout

# Vynucení změny hesla při příštím přihlášení
sudo chage -d 0 bob
```

---

## 2. Správa skupin

Skupiny umožňují hromadně spravovat oprávnění pro více uživatelů.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `groupadd` | Vytvoří novou skupinu | `sudo groupadd developers` | (žádný výstup) |
| `groupadd -g` | Vytvoří skupinu s konkrétním GID | `sudo groupadd -g 2001 developers` | (žádný výstup) |
| `groupadd -r` | Vytvoří systémovou skupinu (GID < 1000) | `sudo groupadd -r syslog` | (žádný výstup) |
| `groupmod -n` | Přejmenuje skupinu | `sudo groupmod -n devs developers` | (žádný výstup) |
| `groupmod -g` | Změní GID skupiny | `sudo groupmod -g 2002 developers` | (žádný výstup) |
| `groupdel` | Smaže skupinu | `sudo groupdel developers` | (žádný výstup) |
| `gpasswd -a` | Přidá uživatele do skupiny | `sudo gpasswd -a alice developers` | `Adding user alice to group developers` |
| `gpasswd -d` | Odebere uživatele ze skupiny | `sudo gpasswd -d alice developers` | `Removing user alice from group developers` |
| `gpasswd -M` | Nastaví seznam členů skupiny | `sudo gpasswd -M alice,bob,carol developers` | (žádný výstup) |
| `groups` | Zobrazí skupiny uživatele | `groups alice` | `alice : alice sudo developers docker` |
| `newgrp` | Dočasně změní primární skupinu | `newgrp developers` | (spustí nový shell) |

```bash
# Vytvoření skupiny a přiřazení uživatelů
sudo groupadd project-alpha
sudo gpasswd -M alice,bob,carol project-alpha

# Zobrazení členů skupiny
grep project-alpha /etc/group
# project-alpha:x:1003:alice,bob,carol

# Dočasné přepnutí primární skupiny
newgrp project-alpha
# po ukončení shellu (exit) se vrátíte zpět
```

---

## 3. Systémové soubory

Linux ukládá informace o uživatelích a skupinách do textových souborů v `/etc`.

### /etc/passwd

Struktura: `username:password:UID:GID:comment:home:shell`

| Pole | Význam | Příklad |
|------|--------|---------|
| username | Přihlašovací jméno | `alice` |
| password | Vzdy `x` (hash je v `/etc/shadow`) | `x` |
| UID | Identifikátor uživatele | `1001` |
| GID | Identifikátor primární skupiny | `1001` |
| comment | Cele jmeno, kontakt (GECOS) | `Alice Smith` |
| home | Cesta k domovskemu adresari | `/home/alice` |
| shell | Prihlasovaci shell | `/bin/bash` |

### /etc/shadow

Struktura: `username:hash:lastchange:min:max:warn:inactive:expire`

| Pole | Význam | Příklad |
|------|--------|---------|
| username | Prihlasovaci jmeno | `alice` |
| hash | Hash hesla ($type$salt$hash) | `$y$j9T$...` |
| lastchange | Datum posledni zmeny (dny od 1.1.1970) | `19876` |
| min | Minimalni stari hesla (dny) | `0` |
| max | Maximalni stari hesla (dny) | `99999` |
| warn | Varovani pred vyprsenim (dny) | `7` |
| inactive | Dny po vyprseni pred deaktivaci | (prazdne) |
| expire | Datum deaktivace uctu | (prazdne) |

### /etc/group

Struktura: `groupname:password:GID:members`

| Pole | Význam | Příklad |
|------|--------|---------|
| groupname | Nazev skupiny | `developers` |
| password | Heslo skupiny (obvykle prazdne) | `x` |
| GID | Identifikátor skupiny | `2001` |
| members | Seznam clenu (carkou oddeleny) | `alice,bob` |

### Užitečné příkazy

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `getent` | Dotaz do systemovych databazi | `getent passwd alice` | `alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash` |
| `getent group` | Dotaz na skupinu | `getent group developers` | `developers:x:2001:alice,bob` |
| `vipw` | Bezpecna editace /etc/passwd | `sudo vipw` | (otevre editor) |
| `vigr` | Bezpecna editace /etc/group | `sudo vigr` | (otevre editor) |
| `pwck` | Kontrola konzistence /etc/passwd | `sudo pwck` | `user 'alice': directory /home/alice exists` |
| `grpck` | Kontrola konzistence /etc/group | `sudo grpck` | (zadna chyba = prazdny vystup) |

```bash
# Všichni uživatelé v systému
getent passwd

# Všechny skupiny
getent group

# Kontrola existence uživatele
getent passwd alice || echo "Uzivatel neexistuje"

# Běžní uživatelé (UID >= 1000)
awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd
```

---

## 4. Práva superuživatele

> **⚠️ VAROVÁNÍ:** `/etc/sudoers` vzdy editujte prikazem `visudo`, ktery kontroluje syntaxi. Chyba v sudoers muze znemoznit pristup k root uctu.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `sudo` | Spusti prikaz jako root | `sudo apt update` | (vystup apt) |
| `sudo -u` | Spusti prikaz jako jiny uzivatel | `sudo -u alice whoami` | `alice` |
| `sudo -i` | Interaktivni shell jako root | `sudo -i` | `root@host:~#` |
| `sudo -k` | Zneplatni cached sudo | `sudo -k` | (zadny vystup) |
| `visudo` | Bezpecna editace sudoers | `sudo visudo` | (otevre editor) |
| `su -` | Prepne na root (vyzaduje root heslo) | `su -` | `root@host:~#` |
| `su -c` | Spusti prikaz jako jiny uzivatel | `su -c "whoami" alice` | `alice` |

### Syntaxe /etc/sudoers

```bash
# Uživatel může spouštět všechny příkazy
alice    ALL=(ALL:ALL) ALL

# Uživatel bez hesla
bob      ALL=(ALL) NOPASSWD: ALL

# Uživatel smí jen vybrané příkazy
carol    ALL=(ALL) /usr/bin/apt, /usr/bin/systemctl

# Skupina %developers (vše bez hesla)
%developers ALL=(ALL) NOPASSWD: ALL

# Aliasy příkazů pro přehlednost
Cmnd_Alias SERVICES = /usr/bin/systemctl *, /usr/bin/journalctl *
dave ALL=(ALL) SERVICES
```

---

## 5. Informace o uživatelích

Nastroje pro zjisteni, kdo je prave prihlasen a jake ma atributy.

| Příkaz | Popis | Příklad | Výstup |
|--------|-------|---------|--------|
| `who` | Aktualne prihlaseni uzivatele | `who` | `alice   tty7     2025-01-15 10:30`<br>`bob     pts/0    2025-01-15 11:00 (192.168.1.5)` |
| `w` | Podrobnejsi informace (co delaji) | `w` | `10:45:30 up 3 days, 1:15, 2 users, load: 0.08`<br>`USER TTY  FROM       LOGIN@  IDLE WHAT`<br>`alice tty7 :0         10:30  2:25 -bash`<br>`bob pts/0 192.168.1.5 11:00  0:30 vim script.sh` |
| `id` | UID, GID a skupiny uzivatele | `id alice` | `uid=1001(alice) gid=1001(alice) groups=1001(alice),4(adm),27(sudo)` |
| `last` | Historie prihlaseni | `last -n 5` | `alice   tty7  :0    Mon Jan 13 10:30 - 15:45 (05:15)`<br>`bob     pts/0 192.1 Mon Jan 13 09:00 - 11:30 (02:30)` |
| `finger` | Detailni informace o uzivateli | `finger alice` | `Login: alice       Name: Alice Smith`<br>`Directory: /home/alice  Shell: /bin/bash`<br>`Last login Mon Jan 13 10:30 on tty7` |
| `logname` | Jmeno puvodniho prihlasovaciho uzivatele | `logname` | `alice` |

```bash
# Kdo je právě přihlášen?
who

# Co dělají?
w

# Moje vlastní ID
id

# Historie přihlášení alice
last alice

# Neúspěšné pokusy o přihlášení
sudo lastb -n 10
```

---

## Shrnutí

- **Uživatele** vytváříte `useradd`, upravujete `usermod`, mažete `userdel`. Hesla spravuje `passwd`.
- **Skupiny** vytváří `groupadd`, členy spravuje `gpasswd`, zobrazuje `groups`. Dočasné přepnutí řeší `newgrp`.
- **Systémové soubory** (`/etc/passwd`, `/etc/shadow`, `/etc/group`) obsahují identitu uživatelu. Editují se bezpecne `vipw` a `vigr`.
- **Superuživatel** pouziva `sudo`. Konfigurace `/etc/sudoers` se edituje vyhradne `visudo`.
- **Informace** o prihlasenych ziskate pres `who`, `w`, `last` a `id`.

---

➡️ [Zpět na přehled](README.md)
