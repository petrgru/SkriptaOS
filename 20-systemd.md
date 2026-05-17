# Systemd

> Service Manager for Linux

---

## Úvod

**Systemd** je správce systému a služeb pro Linux, který nahrazuje tradiční SysV init. Běží jako proces s PID 1 a stará se o inicializaci systému po startu jádra, správu služeb, socketů, timerů a mount pointů.

Hlavní rozdíly oproti SysV init:

- **Paralelní start** — služby se startují souběžně, nikoliv sekvenčně
- **Unit soubory** — konfigurace v deklarativním INI formátu, žádné shell skripty
- **Automatické závislosti** — systemd řeší pořadí startu podle definovaných vazeb
- **Cgroup integrace** — každá služba běží ve vlastní cgroup pro izolaci zdrojů
- **Socket activation** — služba se spustí až při prvním připojení na socket
- **Journald** — centralizované binární logování s pokročilým filtrováním

Systemd je zpětně kompatibilní se SysV init skripty. Pokud služba nemá unit soubor, systemd se pokusí spustit `/etc/init.d/skript`. Staré příkazy `service` a `chkconfig` jsou přesměrovány na `systemctl`.

> **POZNÁMKA:** Základní příkazy systemctl (start/stop/restart/status/enable/disable) a jednoduchý unit soubor jsou popsány v [kapitole 6 – Správa procesů](06-sprava-procesu.md). Tato kapitola se věnuje pokročilým technikám.

---

## 1. Správa služeb

Základní příkazy systemctl znáte z kapitoly 6. Nyní se podíváme na pokročilé možnosti.

```bash
# Výpis jednotek podle typu a stavu
$ systemctl list-units --type=service
$ systemctl list-units --type=service --state=running,exited,failed
$ systemctl list-unit-files --type=service
$ systemctl list-unit-files --state=masked

# Zobrazení unit souboru (i s drop-in úpravami)
$ systemctl cat sshd

# Všechny vlastnosti jednotky (včetně výchozích hodnot)
$ systemctl show sshd
$ systemctl show -p ExecStart sshd
$ systemctl show -p Type,Restart,CPUQuota sshd

# Strom závislostí (i reverzní)
$ systemctl list-dependencies sshd
$ systemctl list-dependencies --reverse sshd

# Rychlá kontrola stavu
$ systemctl is-active sshd
active
$ systemctl is-enabled sshd
enabled
$ systemctl is-failed sshd
active
```

---

## 2. Unit soubory

Unit soubory jsou konfigurační soubory v INI formátu. Každý typ jednotky má svou příponu: `.service`, `.timer`, `.socket`, `.target`, `.mount`, `.path`, `.slice`.

### Sekce

| Sekce | Účel |
|-------|------|
| `[Unit]` | Metadata, závislosti, popis |
| `[Service]` | Konfigurace služby (jen pro .service) |
| `[Install]` | Kam se jednotka instaluje (při enable) |
| `[Timer]` | Konfigurace timeru (jen pro .timer) |

### [Unit] sekce

```ini
[Unit]
Description=Moje vlastní služba
Documentation=man:my-service(8) https://example.com/docs
After=network.target postgresql.service
Wants=postgresql.service
Requires=network.target
Conflicts=apache2.service
```

- `After`, `Before` — definují pořadí startu
- `Requires` — pokud selže, jednotka se také zastaví
- `Wants` — měkčí závislost (služba se pokusí spustit, ale nevyžaduje)
- `Conflicts` — neslučitelné služby

### [Service] sekce

Typy služeb podle `Type`:

| Typ | Chování |
|-----|---------|
| `simple` | Hlavní proces běží na popředí (výchozí) |
| `forking` | Proces se po startu odpojí (fork + rodič končí) |
| `oneshot` | Provede akci a skončí (často s RemainAfterExit=yes) |
| `notify` | Po inicializaci pošle notifikaci přes sd_socket |
| `dbus` | Registruje se na D-Bus, systemd čeká na jméno |

```ini
[Service]
Type=simple
User=backup
Group=backup
ExecStart=/usr/local/bin/backup.sh
ExecStop=/usr/local/bin/backup-stop.sh
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=10
TimeoutStopSec=30
```

### [Install] sekce

```ini
[Install]
WantedBy=multi-user.target
RequiredBy=network.target
Alias=backup-runner.service
```

- `WantedBy` — `systemctl enable` vytvoří symbolický link v `multi-user.target.wants/`
- `RequiredBy` — silnější vazba (jako `Requires` v opačném směru)
- `Alias` — alternativní název

---

## 3. Targety

Targety představují cílové stavy systému. Nahrazují runlevely ze SysV init.

| SysV runlevel | systemd target | Popis |
|---------------|----------------|-------|
| 0 | `poweroff.target` | Vypnutí systému |
| 1 | `rescue.target` | Jednouživatelský režim (základní údržba) |
| 2,3,4 | `multi-user.target` | Víceuživatelský režim bez GUI |
| 5 | `graphical.target` | Víceuživatelský režim s GUI |
| 6 | `reboot.target` | Restart systému |

```bash
# Zjištění a změna výchozího targetu
$ systemctl get-default
graphical.target
$ sudo systemctl set-default multi-user.target

# Přepnutí do jiného targetu (za běhu)
$ sudo systemctl isolate multi-user.target
$ sudo systemctl isolate rescue.target

# Zobrazení závislostí targetu
$ systemctl list-dependencies multi-user.target

# Rescue a emergency režim
$ sudo systemctl rescue     # mountnuté filesystémy, jednouživatelský
$ sudo systemctl emergency  # pouze root fs, read-only
$ sudo systemctl reboot
```

---

## 4. Timery

Timery jsou moderní náhrada za cron. Každý timer vyžaduje párový `.service` soubor.

### Syntaxe OnCalendar

```bash
# Formát: Den Týden *-*-* Hodina:Minuta:Sekunda
OnCalendar=Mon *-*-* 02:00:00     # Každé pondělí ve 2:00
OnCalendar=daily                  # Denně v půlnoci
OnCalendar=hourly                 # Každou hodinu
OnCalendar=*-*-* 00,12:00:00     # Každý den v 0:00 a 12:00
OnCalendar=Sun 23:00:00          # Každou neděli ve 23:00
```

### Ukázka: timer + service

Timer a jeho párová služba pro denní zálohu v 3:00:

```ini
# /etc/systemd/system/daily-backup.timer
[Unit]
Description=Timer pro denní zálohu
[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true
Unit=daily-backup.service
[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/daily-backup.service
[Unit]
Description=Spuštění denní zálohy
[Service]
Type=oneshot
ExecStart=/usr/local/bin/daily-backup.sh
User=backup
```

```bash
# Aktivace timeru
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now daily-backup.timer

# Seznam timerů
$ systemctl list-timers --all
```

Volba `Persistent=true` znamená, že pokud byl systém v době plánovaného spuštění vypnutý, timer se spustí ihned po startu (catch-up).

---

## 5. Socket activation

Socket activation umožňuje spustit službu až ve chvíli, kdy na její socket dorazí první připojení. Šetří to zdroje a zrychluje start systému.

1. Systemd vytvoří socket (naslouchá na portu)
2. Při příchozím připojení systemd spustí přidruženou službu
3. Služba převezme socket a začne obsluhovat klienty

### Příklad: sshd přes socket

```ini
# /lib/systemd/system/sshd.socket
[Unit]
Description=SSH Socket pro per-connection

[Socket]
ListenStream=22
Accept=yes

[Install]
WantedBy=sockets.target
```

```bash
# Seznam všech aktivních socketů
$ systemctl list-sockets

# Výstup:
# Listen               Unit                  Activates
# 0.0.0.0:22          sshd.socket           sshd.service
```

### Maskování

```bash
# Úplné zakázání (nelze spustit ani ručně — link na /dev/null)
$ sudo systemctl mask apache2

# Odmaskování
$ sudo systemctl unmask apache2
```

---

## 6. Resource control

Systemd integruje cgroups v2 pro omezení zdrojů přímo v unit souborech nebo pomocí `systemctl set-property`.

### Direktivy

| Direktiva | Význam |
|-----------|--------|
| `CPUQuota=` | Maximální % CPU (např. 50 %, 200 % pro multi-core) |
| `MemoryMax=` | Maximální paměť v bytech (absolutní limit) |
| `MemoryHigh=` | Soft limit (při překročení začne throttling) |
| `TasksMax=` | Maximální počet úloh (procesů + vláken) |
| `IOReadBandwidthMax=` | Limit čtení z disku (např. 10M) |
| `IOWriteBandwidthMax=` | Limit zápisu na disk |

```bash
# Omezení běžící služby
$ sudo systemctl set-property nginx CPUQuota=50%
$ sudo systemctl set-property nginx MemoryMax=500M
$ sudo systemctl set-property nginx IOReadBandwidthMax=/dev/sda:10M

# Ověření
$ systemctl show -p CPUQuota nginx
CPUQuota=50%

# Spuštění procesu s omezením
$ sudo systemd-run --scope -p MemoryMax=500M -p CPUQuota=50% /usr/bin/myapp
$ systemd-run --user --scope -p MemoryMax=200M firefox

# Sledování cgroups v reálném čase
$ systemd-cgtop
$ systemd-cgls
```

---

## 7. Journalctl — pokročilé

Základní `journalctl -u nginx -n 50` znáte z kapitoly 6. Nyní se podíváme na pokročilé filtrování.

### Filtrování

```bash
# Časové okno
$ journalctl --since "1 hour ago" --until "10 min ago"
$ journalctl --since yesterday --until "now"

# Priorita a boot
$ journalctl -p err -b                # Chyby z aktuálního bootu
$ journalctl -p warning -b -1          # Varování z předchozího bootu

# Filtrování podle polí
$ journalctl _SYSTEMD_UNIT=sshd.service
$ journalctl _PID=1234
$ journalctl _UID=0
$ journalctl _SYSTEMD_UNIT=sshd.service _PID=1234    # AND logika
```

### Strukturovaný výstup

```bash
# JSON formát se všemi poli
$ journalctl -u sshd -o json-pretty
{
    "__REALTIME_TIMESTAMP" : "1735200000000000",
    "_SYSTEMD_UNIT" : "sshd.service",
    "_PID" : "1234",
    "MESSAGE" : "Accepted publickey for petrg from 10.0.0.1"
}

# Další formáty
$ journalctl -u sshd -o verbose    # Všechna metadata
$ journalctl -u sshd -o short-iso  # S ISO časem
```

### Správa velikosti

```bash
# Velikost logů na disku
$ journalctl --disk-usage

# Omezení velikosti a stáří
$ sudo journalctl --vacuum-size=200M
$ sudo journalctl --vacuum-time=2weeks

# Ruční rotace
$ sudo journalctl --rotate
```

---

## 8. Drop-in konfigurace

Drop-in soubory umožňují změnit konfiguraci služby bez úpravy původního unit souboru. Při aktualizaci balíčku se původní soubor přepíše, ale drop-in adresář zůstává nedotčen.

```bash
# Interaktivní editace (otevře editor)
$ sudo systemctl edit sshd

# Vytvoření drop-inu s vlastním názvem
$ sudo systemctl edit --drop-in=limits.conf sshd
```

Příklad `/etc/systemd/system/ssh.service.d/override.conf`:

```ini
[Service]
Restart=always
RestartSec=5
LimitNOFILE=65536
Environment=MY_APP_CONFIG=/etc/myapp/production.conf
```

```bash
# Zobrazení efektivní konfigurace (originál + drop-in)
$ systemctl cat sshd

# Výpis změn oproti originálu
$ systemctl diff sshd

# Vrácení na výchozí konfiguraci (smaže drop-in)
$ sudo systemctl revert sshd

# Změny se projeví po reloadu
$ sudo systemctl daemon-reload
$ sudo systemctl restart sshd
```

---

## 9. Debugging a diagnostika

Systemd obsahuje vestavěné nástroje pro analýzu bootu a ladění.

### Analýza bootu

```bash
# Celkový čas bootu
$ systemd-analyze
Startup finished in 2.345s (kernel) + 4.567s (init) = 6.912s

# Čas startu jednotlivých služeb
$ systemd-analyze blame
5.123s postgresql.service
2.456s networkd-dispatcher.service
1.234s fstrim.service
0.890s ssh.service

# Kritická cesta (nejdelší řetězec závislostí)
$ systemd-analyze critical-chain
multi-user.target @4.567s
└─postgresql.service @3.456s + 1.111s
  └─network.target @2.345s
```

### Validace a ladění

```bash
# Ověření syntaxe unit souborů
$ sudo systemd-analyze verify /etc/systemd/system/*.service

# Graf závislostí (vygeneruje SVG)
$ systemd-analyze dot multi-user.target | dot -Tsvg > boot-graph.svg

# Detailní ladící logy
$ sudo journalctl -p debug -b
$ sudo journalctl -p debug -b -u sshd

# Diagnostika selhané služby
$ systemctl status failedservice.service
$ journalctl -u failedservice.service -p err
```

### Core dumps

```bash
# Seznam core dumpů
$ coredumpctl list

# Info o konkrétním (podle PID)
$ coredumpctl info 1234

# Spuštění gdb s core dumpem
$ coredumpctl debug 1234
```

---

## 10. Užitečné triky

```bash
# Maskování a reset
$ sudo systemctl mask apache2       # Úplné zakázání
$ sudo systemctl unmask apache2
$ sudo systemctl reset-failed sshd  # Reset stavu failed

# Restart systemd (PID 1) — bezpečné, služby zůstávají
$ sudo systemctl daemon-reexec

# Reload konfigurace a signály
$ sudo systemctl daemon-reload
$ sudo systemctl reload sshd
$ sudo systemctl kill -s HUP sshd
$ sudo systemctl stop --signal=SIGTERM nginx

# Spuštění příkazu s omezením
$ sudo systemd-run -p MemoryMax=100M -p CPUQuota=25% /usr/bin/curl https://example.com
$ systemd-run --user --unit=my-task -p MemoryMax=200M /usr/bin/myapp
$ sudo systemd-run --unit=timeout-task -p RuntimeMaxSec=30 /usr/bin/slow-command
```

---

## Shrnutí

| Kategorie | Příkaz | Popis |
|-----------|--------|-------|
| Informace | `systemctl list-units --type=service --state=running` | Seznam běžících služeb |
| Informace | `systemctl cat/show sshd` | Zobrazení / vlastnosti služby |
| Informace | `systemctl list-dependencies sshd` | Strom závislostí |
| Targety | `systemctl get-default` | Výchozí target |
| Targety | `systemctl isolate multi-user.target` | Přepnutí targetu |
| Timery | `systemctl list-timers` | Seznam timerů |
| Socket | `systemctl list-sockets` | Aktivní sockety |
| Resource | `systemctl set-property nginx CPUQuota=50%` | Omezení CPU |
| Resource | `systemd-cgtop` | Sledování cgroups |
| Logy | `journalctl --since "1 hour ago"` | Logy za časové okno |
| Logy | `journalctl -p err -b` | Chyby z aktuálního bootu |
| Logy | `journalctl --vacuum-size=200M` | Omezení logů |
| Drop-in | `systemctl edit / revert sshd` | Úprava / vrácení konfigurace |
| Diagnostika | `systemd-analyze blame` | Doba bootu per služba |
| Diagnostika | `systemd-analyze critical-chain` | Kritická cesta |
| Triky | `systemctl mask apache2` | Maskování služby |
| Triky | `systemctl daemon-reexec` | Restart systemd (PID 1) |

> **POZNÁMKA:** Základní příkazy systemctl (start/stop/restart/status/enable/disable) jsou popsány v [kapitole 6 – Správa procesů](06-sprava-procesu.md).

---

➡️ [Zpět na přehled](README.md)
