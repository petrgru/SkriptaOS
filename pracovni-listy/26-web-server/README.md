# Pracovní list: Webový server

> Procvičení Apache, Nginx, PHP-FPM, reverse proxy a logování webových serverů.

---

## Zadání

### Úkol 1: Apache2 — virtuální hostitel

1. Nainstaluj Apache2 (pokud není): `sudo apt install apache2`
2. Zkontroluj stav: `sudo systemctl status apache2`
3. Vytvoř adresář pro web: `sudo mkdir -p /var/www/mujweb`
4. Vytvoř index stránku:
   ```bash
   echo '<h1>Muj web</h1>' | sudo tee /var/www/mujweb/index.html
   ```
5. Vytvoř virtuálního hostitele `/etc/apache2/sites-available/mujweb.conf`:
   ```apache
   <VirtualHost *:80>
       ServerName mujweb.local
       DocumentRoot /var/www/mujweb
   </VirtualHost>
   ```
6. Povol hostitele: `sudo a2ensite mujweb.conf`
7. Reload: `sudo systemctl reload apache2`
8. Ověř: `curl http://localhost`

### Úkol 2: Nginx — reverse proxy

1. Nainstaluj Nginx: `sudo apt install nginx`
2. Zastav Apache2 (pokud běží) a spusť Nginx
3. Vytvoř Nginx server blok v `/etc/nginx/sites-available/proxy`:
   ```nginx
   server {
       listen 80;
       server_name proxy.local;
       location / {
           proxy_pass http://localhost:3000;
       }
   }
   ```
4. Povol server blok: `sudo ln -s /etc/nginx/sites-available/proxy /etc/nginx/sites-enabled/`
5. Otestuj konfiguraci: `sudo nginx -t`
6. Spusť testovací server na portu 3000: `python3 -m http.server 3000 &`
7. Ověř proxy: `curl http://localhost`
8. Jaký je rozdíl mezi Apache a Nginx?

### Úkol 3: Logování a TLS

1. Zobraz přístupový log Apache: `tail -20 /var/log/apache2/access.log`
2. Zobraz chybový log Apache: `tail -20 /var/log/apache2/error.log`
3. Zobraz přístupový log Nginx: `tail -20 /var/log/nginx/access.log`
4. Analyzuj log:
   ```bash
   # Kolik požadavků?
   wc -l /var/log/apache2/access.log
   # Kolik unikátních IP?
   awk '{print $1}' /var/log/apache2/access.log | sort -u | wc -l
   # Nejčastější stavové kódy
   awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -rn
   ```
5. K čemu slouží ACME (Let's Encrypt) a jak bys nastavil HTTPS?
6. Co je PHP-FPM a kdy ho použít?

---

## Řešení

<details>
<summary>Zobrazit řešení (klikni)</summary>

### Úkol 1 — Řešení

```bash
# 1-4. Instalace a příprava
sudo apt install apache2
sudo systemctl status apache2
sudo mkdir -p /var/www/mujweb
echo '<h1>Muj web</h1>' | sudo tee /var/www/mujweb/index.html

# 5. Virtuální hostitel
sudo tee /etc/apache2/sites-available/mujweb.conf << 'EOF'
<VirtualHost *:80>
    ServerName mujweb.local
    DocumentRoot /var/www/mujweb
    ErrorLog ${APACHE_LOG_DIR}/mujweb_error.log
    CustomLog ${APACHE_LOG_DIR}/mujweb_access.log combined
</VirtualHost>
EOF

# 6-8. Aktivace
sudo a2ensite mujweb.conf
sudo systemctl reload apache2
curl http://localhost
```

### Úkol 2 — Řešení

```bash
# 1-4. Nginx reverse proxy
sudo apt install nginx
sudo systemctl stop apache2
sudo systemctl start nginx

sudo tee /etc/nginx/sites-available/proxy << 'EOF'
server {
    listen 80;
    server_name proxy.local;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/proxy /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# 5-7. Test
python3 -m http.server 3000 &
curl http://localhost

# 8. Apache: modulární, .htaccess, thread/process based
# Nginx: event-driven, vysoký výkon, reverse proxy
```

### Úkol 3 — Řešení

```bash
# 1-4. Logy
tail -20 /var/log/apache2/access.log
tail -20 /var/log/apache2/error.log

wc -l /var/log/apache2/access.log
awk '{print $1}' /var/log/apache2/access.log | sort -u | wc -l
awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -rn

# 5. ACME (Let's Encrypt) = automatické SSL certifikáty
# sudo apt install certbot python3-certbot-nginx
# sudo certbot --nginx -d example.com

# 6. PHP-FPM = FastCGI Process Manager pro PHP
# Používá se, když webový server potřebuje zpracovat PHP
# Apache: mod_php (vestavěný)
# Nginx: PHP-FPM (externí proces)
```

</details>

---

## Kontrola funkčnosti

Ověř, že:

- [ ] Umíš nainstalovat a nakonfigurovat Apache i Nginx
- [ ] Rozumíš konceptu virtuálního hostitele a reverse proxy
- [ ] Dokážeš analyzovat logy webového serveru
- [ ] Víš, k čemu slouží ACME/Let's Encrypt a PHP-FPM

---

## Sebehodnocení

Ohodnoť své pochopení látky (1 = výborně, 5 = vůbec):

- [ ] 1 — Zvládám bez váhání, vše chápu
- [ ] 2 — Zvládám s mírnou pomocí, většinu chápu
- [ ] 3 — Potřebuji se ještě podívat do kapitoly
- [ ] 4 — Většinu jsem nepochopil
- [ ] 5 — Vůbec netušim, kapitola mi není jasná

---

➡️ [Zpět na kapitolu](../../26-web-server.md) · [Zpět na přehled](../../README.md)
