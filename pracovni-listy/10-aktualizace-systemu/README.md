# Pracovní list: Aktualizace systému

> Procvičení správy balíčků, aktualizací a rollbacku v Linuxu.

---

## Zadání

### Úkol 1: apt údržba

Proveď kompletní údržbu systému pomocí apt:

1. Aktualizuj seznam balíčků (`apt update`)
2. Zjisti, kolik balíčků lze upgradovat (`apt list --upgradable 2>/dev/null | wc -l`)
3. Proveď upgrade všech balíčků (`apt upgrade`)
4. Odstraň nepotřebné závislosti (`apt autoremove`)
5. Vyčisti cache stažených balíčků (`apt clean`)
6. Najdi balíček podle názvu nebo popisu: "text editor" (`apt search text editor`)

Zapiš všechny použité příkazy a klíčové výstupy (počty upgradovaných balíčků atd.).

### Úkol 2: Instalace a správa balíčků

1. Nainstaluj balíček `tree` (pokud ještě není)
2. Zjisti informace o balíčku: `apt show tree` — zapiš verzi, velikost a závislosti
3. Najdi, kterému balíčku patří soubor `/bin/ls` (`dpkg -S /bin/ls`)
4. Vypiš všechny nainstalované balíčky obsahující "ssh" (`dpkg -l | grep ssh`)
5. Odstraň balíček `tree` (pouze `remove`, ne `purge`)
6. Zkus `apt install sl` — co tento balíček dělá?

### Úkol 3: Simulace rollbacku

1. Nainstaluj balíček `nano` (pokud není)
2. Zjisti číslo poslední transakce v `/var/log/dpkg.log` (`tail -20 /var/log/dpkg.log`)
3. Odstraň `nano` a znovu ho nainstaluj
4. Jak bys postupoval při rollbacku aktualizace, která rozbila systém?
5. K čemu slouží `dpkg --get-selections` a `apt list --installed`?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1. Aktualizace seznamu
sudo apt update

# 2. Počet upgradovatelných balíčků
apt list --upgradable 2>/dev/null | grep -c upgradable

# 3. Upgrade
sudo apt upgrade -y

# 4. Odstranění závislostí
sudo apt autoremove --purge -y

# 5. Čištění cache
sudo apt clean

# 6. Hledání balíčku
apt search text editor
# Najde např. vim, nano, emacs
```

### Úkol 2 — Řešení

```bash
# 1. Instalace tree
sudo apt install tree

# 2. Informace o balíčku
apt show tree
# Verze: 2.1.0-1, Velikost: 74.2 kB

# 3. Kterému balíčku patří soubor
dpkg -S /bin/ls
# coreutils: /bin/ls

# 4. Balíčky s ssh
dpkg -l | grep ssh
# openssh-client, openssh-server, ssh-import-id...

# 5. Odstranění
sudo apt remove tree

# 6. sl - parní lokomotiva (zábavný balíček)
sudo apt install sl
sl  # jedoucí lokomotiva v terminálu
```

### Úkol 3 — Řešení

```bash
# 1. Instalace nano
sudo apt install nano

# 2. Poslední transakce
tail -20 /var/log/dpkg.log

# 3. Přes instalace
sudo apt remove nano
sudo apt install nano

# 4. Rollback aktualizace:
# a. Zjistit, co se změnilo: dpkg.log
# b. Znovu nainstalovat starší verzi: apt install <balicek>=<verze>
# c. V extrémním případě záloha ze snapshotu
# d. Důležité: před velkou aktualizací vždy záloha!

# 5. Užitečné příkazy
dpkg --get-selections  # výpis všech balíčků a jejich stavu
apt list --installed   # alternativní výpis
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš provést kompletní apt údržbu (update, upgrade, autoremove, clean)
- [ ] Dokážeš najít balíček a zjistit jeho informace
- [ ] Víš, jak zjistit, kterému balíčku patří soubor
- [ ] Rozumíš principu rollbacku a proč je důležitý

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../10-aktualizace-systemu.md) · [Zpět na přehled](../../README.md)
