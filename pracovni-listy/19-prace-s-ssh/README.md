# Pracovní list: Práce s SSH

> Procvičení generování SSH klíčů, konfigurace serveru, tunelování a přenosu souborů.

---

## Zadání

### Úkol 1: SSH klíče a přihlášení

1. Vygeneruj pár SSH klíčů (Ed25519) do `/tmp/muj_klic` bez passphrase
2. Zobraz veřejný klíč: `cat /tmp/muj_klic.pub`
3. Zkopíruj veřejný klíč do `~/.ssh/authorized_keys` (pro test na localhost)
   ```bash
   cat /tmp/muj_klic.pub >> ~/.ssh/authorized_keys
   ```
4. Přihlas se na localhost: `ssh -i /tmp/muj_klic localhost`
5. Jaká oprávnění musí mít `~/.ssh/authorized_keys`?
6. Co je to passphrase a proč ji používat?

### Úkol 2: Konfigurace SSH serveru

1. Zobraz konfiguraci SSH serveru: `cat /etc/ssh/sshd_config | grep -v '^#' | grep -v '^$'`
2. Najdi a zapiš hodnoty:
   - `Port` — na jakém portu SSH naslouchá?
   - `PermitRootLogin` — je povoleno přihlášení roota?
   - `PasswordAuthentication` — je povoleno přihlášení heslem?
   - `PubkeyAuthentication` — je povolena autentizace klíčem?
3. Jaká je doporučená bezpečná konfigurace pro SSH server? (alespoň 3 doporučení)
4. Po změně konfigurace SSH serveru je nutné restartovat službu — jaký příkaz?

### Úkol 3: SSH tunel a přenos souborů

1. Vytvoř soubor `test.txt` s obsahem "SSH test"
2. Zkopíruj ho na localhost pomocí `scp`:
   ```bash
   scp -i /tmp/muj_klic test.txt localhost:/tmp/
   ```
3. Zkopíruj stejný soubor pomocí `rsync -av`
4. Vysvětli, k čemu slouží SSH tunel (port forwarding):
   - Lokální tunel: `ssh -L 8080:localhost:80 server`
   - Vzdálený tunel: `ssh -R 8080:localhost:80 server`
5. Jaký je rozdíl mezi `scp` a `rsync` přes SSH?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-3. Generování a instalace klíče
ssh-keygen -t ed25519 -f /tmp/muj_klic -N ""
cat /tmp/muj_klic.pub >> ~/.ssh/authorized_keys

# 4. Přihlášení
ssh -i /tmp/muj_klic localhost

# 5. authorized_keys musí mít 600
chmod 600 ~/.ssh/authorized_keys

# 6. Passphrase je heslo k soukromému klíči
# Chrání klíč, i když někdo získá soubor
# Bez passphrase stačí soubor ukrást a klíč lze použít
```

### Úkol 2 — Řešení

```bash
# 1-2. Konfigurace
grep -E '^(Port|PermitRootLogin|PasswordAuthentication|PubkeyAuthentication)' /etc/ssh/sshd_config
# Port 22
# PermitRootLogin prohibit-password
# PasswordAuthentication yes
# PubkeyAuthentication yes

# 3. Bezpečná konfigurace:
# - Zakázat root login: PermitRootLogin no
# - Zakázat hesla: PasswordAuthentication no (pouze klíče)
# - Změnit port (např. 2222) proti automatickým skenům
# - Používat jen SSH klíče (Ed25519)
# - Omezit uživatele: AllowUsers alice bob

# 4. Restart služby
sudo systemctl restart sshd
```

### Úkol 3 — Řešení

```bash
# 1-2. SCP
echo "SSH test" > test.txt
scp -i /tmp/muj_klic test.txt localhost:/tmp/

# 3. rsync
rsync -av -e "ssh -i /tmp/muj_klic" test.txt localhost:/tmp/

# 4. SSH tunel:
# Lokální: zpřístupní vzdálený port lokálně
#   ssh -L 8080:localhost:80 server
#   → localhost:8080 → server:80
# Vzdálený: zpřístupní lokální port vzdáleně
#   ssh -R 8080:localhost:80 server
#   → server:8080 → localhost:80

# 5. rsync umí přenášet jen změny (rozdílově)
# scp vždy kopíruje celý soubor
# rsync šetří šířku pásma při opakovaném přenosu
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš vygenerovat SSH klíče a přihlásit se
- [ ] Znáš základní bezpečnostní doporučení pro SSH
- [ ] Rozumíš rozdílu mezi lokálním a vzdáleným tunelem
- [ ] Víš, kdy použít scp a kdy rsync

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../19-prace-s-ssh.md) · [Zpět na přehled](../../README.md)
