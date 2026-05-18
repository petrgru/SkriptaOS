# Pracovní list: RAID a Redundance disků

> Procvičení RAID úrovní, mdadm, LVM, ZFS a SnapRAID.

---

## Zadání

### Úkol 1: RAID úrovně — teorie

1. Doplň tabulku:

| RAID level | Min. disků | Kapacita | Odolnost |
|------------|-----------|----------|----------|
| RAID 0     | ?         | ?        | ?        |
| RAID 1     | ?         | ?        | ?        |
| RAID 5     | ?         | ?        | ?        |
| RAID 6     | ?         | ?        | ?        |
| RAID 10    | ?         | ?        | ?        |

2. Máš-li 3 disky po 1 TB, kolik užitečné kapacity máš u RAID 5?
3. Kdy bys použil RAID 0 místo RAID 1? A kdy naopak?
4. Proč RAID nenahrazuje zálohování? Uveď 3 příklady, před kterými RAID nechrání.

### Úkol 2: Simulace mdadm RAID 1

1. Vytvoř 2 obrazové soubory jako simulaci disků:
   ```bash
   dd if=/dev/zero of=/tmp/disk1.img bs=1M count=100
   dd if=/dev/zero of=/tmp/disk2.img bs=1M count=100
   ```
2. Nastav loop zařízení:
   ```bash
   sudo losetup -fP /tmp/disk1.img
   sudo losetup -fP /tmp/disk2.img
   ```
3. Zjisti názvy loop zařízení (`losetup -a`)
4. Vytvoř RAID 1 pole:
   ```bash
   sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1
   ```
5. Zkontroluj stav: `cat /proc/mdstat`
6. Vytvoř na `/dev/md0` souborový systém a připoj ho
7. Simuluj selhání prvního disku: `sudo mdadm --manage /dev/md0 --fail /dev/loop0`

### Úkol 3: LVM snapshot

1. Vytvoř fyzický svazek na `/dev/md0`:
   ```bash
   sudo pvcreate /dev/md0
   ```
2. Vytvoř volume group: `sudo vgcreate vg_test /dev/md0`
3. Vytvoř logical volume: `sudo lvcreate -L 50M -n lv_test vg_test`
4. Vytvoř FS a připoj: `sudo mkfs.ext4 /dev/vg_test/lv_test`
5. Vytvoř snapshot: `sudo lvcreate -L 10M -s -n lv_snap /dev/vg_test/lv_test`
6. K čemu slouží snapshot? A k čemu se používá LVM oproti přímému mdadm RAIDu?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

| RAID level | Min. disků | Kapacita           | Odolnost       |
|------------|-----------|--------------------|----------------|
| RAID 0     | 2         | N × min(disk)      | ✗ (0 disků)   |
| RAID 1     | 2         | 1 × min(disk)      | ✓ (1 disk)    |
| RAID 5     | 3         | (N-1) × min(disk)  | ✓ (1 disk)    |
| RAID 6     | 4         | (N-2) × min(disk)  | ✓ (2 disky)   |
| RAID 10    | 4         | N/2 × min(disk)    | ✓ (1-2 disky) |

2. RAID 5 se 3×1TB = (3-1) × 1TB = 2TB užitečné kapacity
3. RAID 0: když potřebuješ max. výkon a data nejsou kritická (cache, dočasné soubory). RAID 1: když potřebuješ ochranu dat (systémový disk).
4. RAID nechrání proti: smazání souboru, ransomware (zašifruje všechny disky), požáru/krádeži, chybě softwaru.

### Úkol 2 — Řešení

```bash
# 1-2. Příprava
dd if=/dev/zero of=/tmp/disk1.img bs=1M count=100
dd if=/dev/zero of=/tmp/disk2.img bs=1M count=100
sudo losetup -fP /tmp/disk1.img
sudo losetup -fP /tmp/disk2.img
losetup -a

# 3-4. Vytvoření RAID 1
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1
cat /proc/mdstat
# md0 : active raid1 loop1[1] loop0[0]
#       102400 blocks super 1.2 [2/2] [UU]

# 5. Simulace selhání
sudo mdadm --manage /dev/md0 --fail /dev/loop0
cat /proc/mdstat
# md0 : active raid1 loop1[1] loop0[0](F)
# [2/1] [_U]
```

### Úkol 3 — Řešení

```bash
# 1-4. LVM na RAID
sudo pvcreate /dev/md0
sudo vgcreate vg_test /dev/md0
sudo lvcreate -L 50M -n lv_test vg_test
sudo mkfs.ext4 /dev/vg_test/lv_test
sudo mount /dev/vg_test/lv_test /mnt

# 5. Snapshot
sudo lvcreate -L 10M -s -n lv_snap /dev/vg_test/lv_test
# Snapshot zachytí stav v čase, umožňuje konzistentní zálohu
# LVM oproti mdadm: flexibilní změna velikosti, snapshoots
# mdadm: čistý RAID výkon, jednodušší
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Znáš výpočet kapacity pro různé RAID úrovně
- [ ] Víš, že RAID nenahrazuje zálohování a proč
- [ ] Rozumíš principu mirroru (RAID 1) a parity (RAID 5/6)
- [ ] Víš, k čemu slouží LVM snapshot

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../17-raid-redundance-disku.md) · [Zpět na přehled](../../README.md)
