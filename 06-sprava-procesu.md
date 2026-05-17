# Správa procesů v OS

> Process Management in Operating Systems

---

## Úvod

Správa procesů je jednou z klíčových oblastí operačních systémů. Tato kapitola se zaměřuje na životní cyklus procesů, jejich stavy, možnosti monitorování a řízení, stejně jako na prostředky meziprocesové komunikace.

## 1. Vytvoření procesu

Když systém vytváří nový proces, používá k tomu systémová volání. Každý proces (kromě init/PID 1) má svého rodiče.

| Koncept | Popis | Příklad | Výstup |
|---------|-------|---------|--------------|
| `fork()` | Vytvoří kopii volajícího procesu. Rodič dostane PID dítěte, dítě dostane 0. | `pid_t pid = fork();` | Rodič: `PID=1234, child=5678` |
| `exec()` | Nahradí aktuální proces novým programem. | `execlp("ls", "ls", "-l", NULL);` | Spustí `ls -l` v aktuálním procesu |
| `spawn()` | Kombinace fork+exec (POSIX `posix_spawn`). | `posix_spawn(&pid, "/bin/ls", ...)` | Vytvoří nový proces s ls |
| `wait()` | Rodič počká na dokončení dítěte. | `waitpid(child, &status, 0);` | Blokuje, dokud dítě neskončí |
| `system()` | Spustí příkaz v shellu (blokující). | `system("echo hello");` | `hello` |

### Příklad: fork() a exec()

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        // Dítě (child process)
        printf("Dite: PID=%d, rodic=%d\n", getpid(), getppid());
        execlp("/bin/echo", "echo", "Ahoj z exec!", NULL);
    } else if (pid > 0) {
        // Rodič (parent process)
        wait(NULL);
        printf("Rodic: Dite %d dokoncilo.\n", pid);
    }
    return 0;
}
```

**Kompilace a výstup:**
```
$ gcc fork_example.c -o fork_example
$ ./fork_example
Dite: PID=5678, rodic=1234
Ahoj z exec!
Rodic: Dite 5678 dokoncilo.
```

### Fork bomb (varování)

```bash
:(){ :|:& };:
# Toto vytvoří nekonečné množství procesů -> systém zamrzne
# Nikdy nespouštět na produkčním stroji!
```

---

## 2. Monitorování procesů

### Základní nástroje

| Nástroj | Účel | Příklad | Výstup |
|---------|------|---------|--------|
| `ps` | Okamžitý snímek procesů | `ps aux` | Seznam všech procesů s detaily |
| `ps -ef` | Standardní výpis | `ps -ef --forest` | Stromová struktura procesů |
| `top` | Živý náhled, řazení dle CPU | `top` | Obnovuje se každé 2s |
| `htop` | Interaktivní, barevný | `htop` | Strom, vyhledávání, signály |
| `lsof` | Otevřené soubory procesů | `lsof -i :80` | Procesy používající port 80 |
| `/proc` | Virtuální souborový systém | `ls /proc/1234/` | Info o procesu PID 1234 |

### Příklad: ps aux

```
$ ps aux | head -5
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.4 167936 13916 ?        Ss   10:15   0:03 /sbin/init
root         2  0.0  0.0      0     0 ?        S    10:15   0:00 [kthreadd]
petrg     1234  0.5  2.1 892340 65432 ?        Sl   10:16   0:15 /usr/bin/gnome-shell
petrg     5678  0.0  0.1 245680 12345 pts/0    Ss+  10:17   0:00 bash
```

### Příklad: Strom procesů

```
$ ps -ef --forest | head -10
root         1     0  0 10:15 ?        00:00:03 /sbin/init splash
root       378     1  0 10:15 ?        00:00:01  \_ /lib/systemd/systemd-journald
root       456     1  0 10:15 ?        00:00:00  \_ /lib/systemd/systemd-udevd
petrg     1234     1  0 10:16 ?        00:00:15  \_ /usr/bin/gnome-shell
petrg     5678  1234  0 10:17 pts/0    00:00:00      \_ bash
petrg     6789  5678  0 10:18 pts/0    00:00:00          \_ ps -ef --forest
```

### Příklad: /proc filesystem

```bash
# Informace o procesu PID 1 (init)
$ cat /proc/1/status
Name:   systemd
Pid:    1
PPid:   0
Uid:    0    0    0    0
Gid:    0    0    0    0
State:  S (sleeping)

$ cat /proc/1/cmdline
/sbin/init splash

$ ls /proc/1/fd/
0  1  2  3  4  5  6  7  8  9
# Symbolické linky na otevřené soubory
```

### Příklad: lsof

```bash
# Které procesy používají port 80?
$ sudo lsof -i :80
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx    1234   root    6u  IPv4  54321      0t0  TCP *:80 (LISTEN)
nginx    1235   www-data 6u  IPv4 54321      0t0  TCP *:80 (LISTEN)

# Otevřené soubory pro PID 5678
$ lsof -p 5678
COMMAND  PID   USER   FD   TYPE     DEVICE  SIZE   NODE NAME
bash    5678  petrg  cwd   DIR       8,1    4096 12345 /home/petrg
bash    5678  petrg  txt   REG       8,1 1214512 67890 /usr/bin/bash
bash    5678  petrg  mem   REG       8,1  ...    ...   /usr/lib/libc.so.6
```

---

## 3. Řízení procesů

### Signály

| Signál | Číslo | Význam | Výchozí akce |
|--------|-------|--------|-------------|
| `SIGTERM` | 15 | Ukončení (žádost o korektní konec) | Ukončení procesu |
| `SIGKILL` | 9 | Násilné ukončení (nelze zachytit) | Okamžité ukončení |
| `SIGHUP` | 1 | Reload konfigurace / znovunačtení | Ukončení |
| `SIGSTOP` | 19 | Pozastavení procesu | Zastavení |
| `SIGCONT` | 18 | Obnovení pozastaveného procesu | Pokračování |
| `SIGINT` | 2 | Přerušení z klávesnice (Ctrl+C) | Ukončení |
| `SIGQUIT` | 3 | Core dump + ukončení | Core + konec |

### kill a killall

```bash
# Ukončení procesu (korektní)
$ kill 1234              # SIGTERM (default)
$ kill -15 1234          # explicit SIGTERM

# Násilné ukončení
$ kill -9 1234           # SIGKILL

# Reload konfigurace
$ kill -HUP 1234         # SIGHUP

# Poslat signál podle jména
$ killall nginx          # SIGTERM všem nginx procesům
$ killall -9 chrome      # SIGKILL všem chrome procesům

# Poslat signál podle jména (pkill)
$ pkill -f "python script.py"
```

### Priorita procesů: nice / renice

| Koncept | Rozsah | Význam |
|---------|--------|--------|
| `nice` | -20 az +19 | Nizsi hodnota = vyssi priorita |
| `renice` | zmena za behu | Jen root muze davat zapornou hodnotu |

```bash
# Spustit proces s nizsi prioritou
$ nice -n 10 ./velky_vycet.sh

# Spustit s vysokou prioritou (jen root)
$ sudo nice -n -5 ./dolezity_proces

# Zmenit prioritu za behu
$ renice -n 15 -p 1234

# Zobrazit prioritu
$ ps -eo pid,ni,cmd | head -5
  PID  NI COMMAND
    1   0 /sbin/init
 1234  10 ./velky_vycet.sh
 5678  -5 ./dolezity_proces
```

### cgroups (control groups)

Omezují zdroje pro skupiny procesů. Pouziva je Docker, systemd, containerd.

```bash
# Vytvoreni cgroup s omezenim CPU
$ sudo mkdir /sys/fs/cgroup/cpu/moje_cg
$ echo 50000 > /sys/fs/cgroup/cpu/moje_cg/cpu.cfs_quota_us
$ echo 1234 > /sys/fs/cgroup/cpu/moje_cg/cgroup.procs

# Zobrazeni cgroups procesu
$ cat /proc/1234/cgroup
0::/system.slice/ssh.service

# systemd bezi v cgroup hierarchii
$ systemd-cgls
Control group /:
-.slice
├─system.slice
│ ├─sshd.service (1234)
│ └─nginx.service (5678)
└─user.slice
  └─user-1000.slice
```

### systemd:

```bash
# Sprava sluzeb
$ systemctl start nginx
$ systemctl stop nginx
$ systemctl restart nginx
$ systemctl status nginx

# Povoleni/zakazani pri startu
$ systemctl enable nginx
$ systemctl disable nginx

# Zobrazeni logu sluzby
$ journalctl -u nginx -n 50 --no-pager

# Vytvoreni vlastni sluzby
$ cat /etc/systemd/system/moje-sluzba.service
```

```ini
[Unit]
Description=Moje testovaci sluzba
After=network.target

[Service]
Type=simple
User=petrg
ExecStart=/usr/local/bin/muj_skript.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

---

## 4. Stavové proměnné

| Proměnná | Popis | Získání v C | Získání v bash |
|----------|-------|-------------|----------------|
| `PID` | ID procesu | `getpid()` | `$$` |
| `PPID` | ID rodice | `getppid()` | `$PPID` |
| `UID` | ID uzivatele | `getuid()` | `$(id -u)` |
| `GID` | ID skupiny | `getgid()` | `$(id -g)` |
| `EUID` | Efektivní UID | `geteuid()` | -- |
| `EGID` | Efektivní GID | `getegid()` | -- |
| `ENV` | Proměnné prostředí | `getenv("PATH")` | `$PATH` |
| `Exit status` | Navratový kód | `wait(&status)` | `$?` |

### Příklad v C

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    printf("PID:  %d\n", getpid());
    printf("PPID: %d\n", getppid());
    printf("UID:  %d\n", getuid());
    printf("GID:  %d\n", getgid());
    printf("EUID: %d\n", geteuid());
    return 42;  // exit status
}
```

**Výstup:**
```
$ ./stavove_promenne
PID:  6789
PPID: 5678
UID:  1000
GID:  1000
EUID: 1000

$ echo $?
42
```

### Stavové proměnné v /proc

```bash
$ cat /proc/$$/status
Name:   bash
Umask:  0022
State:  S (sleeping)
Tgid:   5678
Pid:    5678
PPid:   1234
Uid:    1000    1000    1000    1000
Gid:    1000    1000    1000    1000
```

### Proměnné prostředí

```bash
# Vypsat vsechny promenne
$ env | head -5
HOME=/home/petrg
PATH=/usr/local/bin:/usr/bin:/bin
USER=petrg
SHELL=/bin/bash
PWD=/home/petrg/Project

# Získat konkretni hodnotu
$ echo $PATH
/usr/local/bin:/usr/bin:/bin

# Nastavit pro jeden proces
$ MY_VAR=ahoj ./my_script.sh
$ MY_VAR=test bash -c 'echo $MY_VAR'
test
```

---

## 5. Stavy procesu

| Stav | Značka v `ps` | Význam |
|------|---------------|--------|
| Running | `R` | Proces prave bezi nebo ceka na CPU |
| Sleeping | `S` | Ceka na udalost (I/O, timer) |
| Interruptible sleep | `S` | Lze prerusit signálem |
| Uninterruptible sleep | `D` | Ceka na I/O (napr. disk) |
| Stopped | `T` | Zastaven signálem (SIGSTOP) |
| Zombie | `Z` | Ukoncen, rodic neprevzal exit status |
| Dead | `X` | Ukoncen, ceka na odstranení |

### Životní cyklus procesu

```
      vytvoreni (fork)
           |
      +----+----+
      |         |
   Running    Zombie (pokud rodic nevola wait)
      |
   Sleeping (cekani na I/O)
      |
   Running (dostal CPU)
      |
   Stopped (SIGSTOP)
      |
   Running (SIGCONT)
      |
   Terminated (exit nebo signal)
      |
   Zombie (cekame na wait)
      |
   Odebran (rodic zavolal wait)
```

### Zombie procesy

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        printf("Dite konci.\n");
        exit(0);  // Dite skonci -> stane se zombie
    } else {
        printf("Rodic spi 30s... (dite je zombie)\n");
        sleep(30);  // Mezitim je dite zombie
        printf("Rodic konci.\n");
        // Dite bude adoptovano initem
    }
    return 0;
}
```

```bash
$ ./zombie_example &
$ ps aux | grep Z
USER   PID  ... STAT COMMAND
petrg  6789 ... Z    [zombie_example] <defunct>
```

### Sledování stavu

```bash
# Vsechny stavy
$ ps -eo pid,stat,comm | head -10
  PID STAT COMMAND
    1 Ss   systemd
  378 Ss   systemd-journald
 1234 Sl   gnome-shell
 5678 Ss   bash
 6789 R+   ps

# Jen zombie
$ ps aux | awk '{if ($8 == "Z") print}'

# Jen bezi
$ ps -e --forest | grep -E "^\ *[0-9].* R"
```

---

## 6. Meziprocesová komunikace (IPC)

### Přehled

| Mechanismus | Rychlost | Použití | Příklady |
|------------|----------|---------|----------|
| Signály | Jednoduché, omezené | Notifikace, ukončení | `kill`, `SIGTERM` |
| Roury (pipe) | Pomalé, jednosměrné | Data mezi příbuznými procesy | `ls | grep` |
| Pojmenované roury (FIFO) | Pomalé, jednosměrné | Data mezi nepříbuznými procesy | `mkfifo` |
| Socket | Rychlé, obousměrné | Síťová i lokální komunikace | TCP/Unix socket |
| Sdílená paměť | Velmi rychlé | Velké objemy dat | `shmget`, `mmap` |
| Message queue | Střední, fronta | Zprávy s prioritou | `mq_open`, `msgget` |

### 6.1 Signály

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void handler(int sig) {
    printf("Dostal jsem signal %d\n", sig);
}

int main() {
    signal(SIGINT, handler);   // Zachytit Ctrl+C
    signal(SIGTERM, handler);  // Zachytit kill

    printf("PID: %d, posli mi signal\n", getpid());
    pause();  // Pockej na signal
    printf("Pokracuji po signalu\n");
    return 0;
}
```

```bash
$ ./signal_handler &
[1] 12345
$ kill -INT 12345   # Posle SIGINT
Dostal jsem signal 2
$ kill 12345        # Posle SIGTERM
Dostal jsem signal 15
$ kill -9 12345     # SIGKILL - nelze zachytit (proces skonci)
```

### 6.2 Roury (pipe)

```bash
# Jednoducha roura v shellu
$ ls -la | grep ".md" | wc -l
15

# Pojmenovana roura (FIFO)
$ mkfifo moje_roura
$ echo "hello" > moje_roura &   # zapisovac (blokuje)
$ cat moje_roura                 # ctenar (odblokuje)
hello
```

**Příklad v C: anonymní roura**

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];
    pid_t pid;
    char buffer[128];

    pipe(pipefd);       // vytvor rouru
    pid = fork();

    if (pid == 0) {
        // Dite - cte
        close(pipefd[1]);
        read(pipefd[0], buffer, sizeof(buffer));
        printf("Dite precetlo: %s\n", buffer);
        close(pipefd[0]);
    } else {
        // Rodic - pise
        close(pipefd[0]);
        write(pipefd[1], "Ahoj z pipe!", 13);
        close(pipefd[1]);
        wait(NULL);
    }
    return 0;
}
```

```
$ ./pipe_example
Dite precetlo: Ahoj z pipe!
```

### 6.3 Socket (Unix domain)

```c
// Server
int sock = socket(AF_UNIX, SOCK_STREAM, 0);
struct sockaddr_un addr;
addr.sun_family = AF_UNIX;
strcpy(addr.sun_path, "/tmp/muj_socket");

bind(sock, (struct sockaddr*)&addr, sizeof(addr));
listen(sock, 5);
int client = accept(sock, NULL, NULL);

char buf[256];
read(client, buf, sizeof(buf));
printf("Prislo: %s\n", buf);
close(client); close(sock);
```

```c
// Klient
int sock = socket(AF_UNIX, SOCK_STREAM, 0);
struct sockaddr_un addr;
addr.sun_family = AF_UNIX;
strcpy(addr.sun_path, "/tmp/muj_socket");

connect(sock, (struct sockaddr*)&addr, sizeof(addr));
write(sock, "Ahoj pres socket!", 18);
close(sock);
```

### 6.4 Sdílená pamět (System V)

```c
#include <stdio.h>
#include <sys/shm.h>
#include <string.h>

int main() {
    int shmid = shmget(1234, 1024, IPC_CREAT | 0666);
    char *data = (char*)shmat(shmid, NULL, 0);

    if (fork() == 0) {
        // Dite - cte
        sleep(1);
        printf("Dite cte: %s\n", data);
        shmdt(data);
    } else {
        // Rodic - pise
        strcpy(data, "Ahoj pres sdilenou pamet!");
        wait(NULL);
        shmctl(shmid, IPC_RMID, NULL);
    }
    return 0;
}
```

### Prevence deadlocku a race condition

```c
// Race condition: dva procesy ctou a pisi do sdilene promenne
// Reseni: mutex / semafor

#include <semaphore.h>
#include <pthread.h>

sem_t semafor;
sem_init(&semafor, 1, 1);  // 1 = sdileny mezi procesy

sem_wait(&semafor);    // lock
// kriticka sekce
sem_post(&semafor);    // unlock
```

---

## 7. Praktické příklady

### 7.1 Sledování procesů v reálném čase

```bash
# top - zivotni cyklus
$ top -u petrg

# htop - interaktivni sprava
$ htop
# F3 - vyhledavani
# F5 - stromove zobrazeni
# F9 - poslat signal
# F6 - radit podle sloupce

# Sledovat zmeny
$ watch -n 1 'ps -eo pid,pcpu,pmem,comm --sort=-pcpu | head -10'
```

### 7.2 Ukončení procesu podle portu

```bash
# 1. Najit PID podle portu
$ sudo lsof -i :3000
COMMAND  PID   USER   FD   TYPE NODE NAME
node    6789  petrg   12u  IPv4  ... TCP *:3000 (LISTEN)

# 2. Ukoncit
$ kill 6789

# Nebo jednim radkem
$ kill $(sudo lsof -t -i :3000)
```

### 7.3 Zmena priority

```bash
# Spustit kompilaci s nizsi prioritou (nezatizit system)
$ nice -n 19 gcc huge_project.c -o huge_project

# Zvysit prioritu behoveho procesu (pokud je dulezity)
$ sudo renice -n -5 -p 1234

# Omezit CPU jednoho procesu na 50%
$ cpulimit -p 1234 -l 50
```

### 7.4 Systemd: vlastní služba

```bash
# Vytvorit sluzbu
$ sudo vim /etc/systemd/system/monitor.service

# Spustit
$ sudo systemctl daemon-reload
$ sudo systemctl start monitor
$ sudo systemctl enable monitor
$ sudo systemctl status monitor

# Sledovat log
$ journalctl -u monitor -f
```

### 7.5 Zjištění, kdo co dělá

```bash
# Nejvice CPU vytizene procesy
$ ps aux --sort=-%cpu | head -5

# Nejvice RAM
$ ps aux --sort=-%mem | head -5

# Strom procesu pro uzivatele
$ ps -u petrg -o pid,ppid,stat,cmd --forest

# Vsechny procesy v behu
$ ps -e -o pid,stat,comm | grep "^ *[0-9] R"

# Detekce zombie
$ ps aux | awk '$8 ~ /Z/ {print "ZOMBIE:", $2, $11}'
```

### 7.6 Komunikace mezi skripty pipe

```bash
# Producer
#!/bin/bash
while true; do
    echo "$(date): data"
    sleep 1
done

# Consumer
#!/bin/bash
./producer.sh | while read line; do
    echo "ZPRACOVANO: $line"
done
```

---

## Shrnutí

| Oblast | Klicovy koncept | Nejcastejsi chyba |
|--------|----------------|-------------------|
| Vytvoreni | `fork()` -> `exec()` | Zapomenout `wait()` -> zombie |
| Monitorovani | `/proc`, `ps`, `top` | Sledovat jen CPU, ne I/O |
| Rizeni | `kill`, `nice`, systemd | Pouzivat `SIGKILL` misto `SIGTERM` |
| Stavy | Running, Sleeping, Zombie | Ignorovat zombie procesy |
| IPC | Pipe, socket, shared memory | Race condition bez semaforu |
| Cgroups | Omezeni zdroju | Prekroceni limitu -> OOM killer |

### Uzitecne prikazy na jeden radek

```bash
# Kolik procesu bezi?
ps -e --no-header | wc -l

# Kolik jich patri mne?
ps -u $(whoami) --no-header | wc -l

# Kdo zere nejvic pameti?
ps aux --sort=-%mem | awk 'NR==2{print $2, $4, $11}'

# Kdo drzi port?
lsof -i :PORT

# Co zrovna bezi na tomto terminalu?
ps -t $(tty | cut -d/ -f3-)
```

> **Dulezite:** Procesy jsou zakladni jednotkou prace v OS. Spravne pochopeni jejich zivotniho cyklu, stavu a IPC mechanizmu je klicove pro efektivni spravu systemu i vyvoj aplikaci.

---

➡️ [Zpět na přehled](README.md)
