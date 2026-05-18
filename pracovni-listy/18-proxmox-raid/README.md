# Pracovní list: Proxmox RAID1 Mirror

> Procvičení vytvoření RAID1 mirroru na Proxmox VE s mdadm a LVM.

---

## Zadání

### Úkol 1: Architektura RAID1 mirroru

Podle kapitoly 18 nakresli (popiš) architekturu výsledného řešení:

1. Jaké komponenty tvoří výsledný RAID1 mirror?
2. K čemu slouží `md0` a k čemu `md1`?
3. Jaká je role LVM v tomto řešení?
4. Proč je na Proxmoxu použit mdadm + LVM místo ZFS?
5. Jaké výhody má toto řešení pro hardware s omezenými zdroji (4-8 GB RAM)?

### Úkol 2: Postup vytvoření RAID1

Seřaď následující kroky do správného pořadí a doplň chybějící:

```
[ ] Instalace mdadm
[ ] Kopie partition tabulky z prvního disku na druhý
[ ] Vytvoření RAID polí (md0, md1)
[ ] Změna typu partition na Linux RAID
[ ] Záloha dat
[ ] GRUB a initramfs konfigurace
[ ] Kopie dat na nové RAID pole
[ ] LVM na RAID (pve_raid)
[ ] Vyčištění druhého disku
[ ] Reboot a přidání druhého disku do mirroru
```

### Úkol 3: Údržba a řešení problémů

1. Jak zkontroluješ stav RAID pole? (`cat /proc/mdstat`)
2. Jak zjistíš detailní informace o `/dev/md0`? (`mdadm --detail`)
3. Co uděláš, když jeden disk v RAID 1 selže?
4. Jak přidáš nový disk jako náhradu?
5. Jak často bys měl kontrolovat stav RAID a proč?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

1. **Komponenty**: 2× SSD disk (sda, sdb), každý se 3 oddíly (BIOS boot, EFI, LVM). sda2 + sdb2 tvoří md1 (EFI RAID1), sda3 + sdb3 tvoří md0 (LVM RAID1). Na md0 je LVM volume group `pve_raid`.
2. **md0** = LVM RAID1 pro root, swap, data. **md1** = EFI boot partition (FAT32) v mirroru.
3. **LVM** umožňuje flexibilní správu oddílů (logické svazky) na RAID poli.
4. **Proč ne ZFS**: ZFS vyžaduje min. 1 GB RAM na 1 TB úložiště, mdadm+LVM má minimální režii.
5. **Výhody pro slabý hardware**: mdadm nemá režii parity ani COW, minimum RAM, stejná ochrana jako ZFS mirror.

### Úkol 2 — Správné pořadí

```
[1] Záloha dat
[2] Vyčištění druhého disku
[3] Kopie partition tabulky
[4] Změna typu partition na Linux RAID
[5] Instalace mdadm
[6] Vytvoření RAID polí (md0, md1)
[7] LVM na RAID (pve_raid)
[8] Kopie dat na nové RAID pole
[9] GRUB a initramfs
[10] Reboot a přidání druhého disku
```

### Úkol 3 — Řešení

```bash
# 1. Stav RAID
cat /proc/mdstat
# md0 : active raid1 sdb3[1] sda3[0]
#       222080 blocks super 1.2 [2/2] [UU]

# 2. Detail
sudo mdadm --detail /dev/md0

# 3. Selhání disku
# Systém dál běží (RAID 1), zjistit který disk selhal
sudo mdadm --detail /dev/md0
# Označit jako failed
sudo mdadm --manage /dev/md0 --fail /dev/sda

# 4. Přidání nového disku
sudo mdadm --manage /dev/md0 --remove /dev/sda
# Fyzicky vyměnit disk, zkopírovat partition tabulku
sudo mdadm --manage /dev/md0 --add /dev/sda
# Automatická resynchronizace

# 5. Kontrola stavu RAID:
# Denně: cat /proc/mdstat (script nebo cron)
# Týdně: mdadm --detail /dev/md0
# Důvod: včas odhalit selhání disku
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Rozumíš architektuře md0 (LVM) a md1 (EFI) v RAID1
- [ ] Znáš správné pořadí kroků pro vytvoření RAID1
- [ ] Víš, jak zkontrolovat stav RAID a řešit selhání disku
- [ ] Rozumíš, proč na Proxmoxu použít mdadm+LVM místo ZFS

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../18-proxmox-raid.md) · [Zpět na přehled](../../README.md)
