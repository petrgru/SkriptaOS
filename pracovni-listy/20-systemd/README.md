# Pracovní list: Systemd

> Procvičení správy služeb, unit souborů, journalctl, timerů a pokročilé konfigurace systemd.

---

## Zadání

### Úkol 1: Správa služeb

1. Zobraz všechny běžící služby: `systemctl list-units --type=service --state=running`
2. Zjisti stav služby SSH: `systemctl status sshd` (nebo `ssh`)
3. Zobraz závislosti SSH: `systemctl list-dependencies sshd`
4. Zjisti, jestli je služba SSH povolena při startu: `systemctl is-enabled sshd`
5. Zobraz vlastnosti služby: `systemctl show -p ExecStart,Type,Restart sshd`
6. Co znamená `Type=simple` a `Type=oneshot`?

### Úkol 2: Vytvoření vlastní služby

Vytvoř systemd service pro skript, který každých 5 minut zapíše datum do logu:

1. Vytvoř skript `/usr/local/bin/date-logger.sh`:
   ```bash
   #!/bin/bash
   echo "$(date): Logger spusten" >> /var/log/date-logger.log
   ```
2. Udělej ho spustitelným
3. Vytvoř unit soubor `/etc/systemd/system/date-logger.service`:
   - Type: oneshot
   - ExecStart: `/usr/local/bin/date-logger.sh`
4. Povol a spusť službu (stačí `systemctl start` pro test)
5. Zkontroluj log: `journalctl -u date-logger`
6. Co bys musel změnit, aby služba běžela nepřetržitě (type: simple)?

### Úkol 3: Systemd timer

1. Vytvoř timer `/etc/systemd/system/date-logger.timer`:
   - OnCalendar: `*:0/5` (každých 5 minut)
   - Unit: `date-logger.service`
2. Povol a spusť timer
3. Zkontroluj stav: `systemctl list-timers --all`
4. Zobraz logy timeru: `journalctl -u date-logger.timer`
5. Jaký je rozdíl mezi `OnCalendar` a `OnBootSec`? Kdy použít který?
6. Proč se používají timery místo cronu?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-5. Správa služeb
systemctl list-units --type=service --state=running
systemctl status sshd
systemctl list-dependencies sshd
systemctl is-enabled sshd
systemctl show -p ExecStart,Type,Restart sshd

# 6. Type=simple: hlavní proces běží na popředí (dokud neskončí)
# Type=oneshot: provede akci a skončí (často s RemainAfterExit=yes)
```

### Úkol 2 — Řešení

```bash
# 1-2. Skript
sudo tee /usr/local/bin/date-logger.sh << 'EOF'
#!/bin/bash
echo "$(date): Logger spusten" >> /var/log/date-logger.log
EOF
sudo chmod +x /usr/local/bin/date-logger.sh

# 3. Unit soubor
sudo tee /etc/systemd/system/date-logger.service << 'EOF'
[Unit]
Description=Datum logger
[Service]
Type=oneshot
ExecStart=/usr/local/bin/date-logger.sh
[Install]
WantedBy=multi-user.target
EOF

# 4-5. Spuštění
sudo systemctl daemon-reload
sudo systemctl start date-logger
journalctl -u date-logger

# 6. Pro nepřetržitý běh bys musel změnit na Type=simple
# a přidat smyčku se sleep do skriptu, nebo použít timer
```

### Úkol 3 — Řešení

```bash
# 1. Timer
sudo tee /etc/systemd/system/date-logger.timer << 'EOF'
[Unit]
Description=Spousti date-logger kazdych 5 minut
[Timer]
OnCalendar=*:0/5
Persistent=true
[Install]
WantedBy=timers.target
EOF

# 2-4. Aktivace
sudo systemctl daemon-reload
sudo systemctl enable --now date-logger.timer
systemctl list-timers --all
journalctl -u date-logger.timer

# 5. OnCalendar = absolutní čas (každých 5 minut, denně v 3:00 atd.)
# OnBootSec = relativní od startu systému
# OnCalendar pro pravidelné úlohy, OnBootSec pro jednorázové po startu

# 6. Timery jsou lepší než cron:
# - Integrované s journald (logování)
# - Lze spouštět zmeškané (Persistent=true)
# - Závislosti na službách
# - Izolace (cgroups, sandboxing)
# - Jednotná konfigurace (systemd)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš spravovat služby (start, stop, status, enable, disable)
- [ ] Dokážeš vytvořit vlastní unit soubor
- [ ] Víš, jak vytvořit timer a jaký je rozdíl oproti cronu
- [ ] Umíš číst logy služby pomocí journalctl

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../20-systemd.md) · [Zpět na přehled](../../README.md)
