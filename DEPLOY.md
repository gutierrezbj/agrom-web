# DEPLOY · agrom-web

Convención **SRS** (Cuaderno Base de Proyectos + Catálogo de Infraestructura, Notion).

| Recurso | Valor |
|---|---|
| Proyecto | agrom-web |
| Offset SRS | **+180** |
| Frontend (interno) | `127.0.0.1:3180` → container nginx:alpine |
| Dominio | `agrom.es` (raíz + `www`) |
| Servidor 1 | `72.62.41.234` (Hostinger · Ubuntu 24.04 · Nginx 1.24) — donde está desplegado agrom-web |
| Servidor 2 | `187.77.71.102` (Hostinger · Ubuntu 22.04) — otra opción de despliegue SRS |
| Path en servidor | `/opt/apps/agrom-web/` |
| Reverse proxy | Nginx host externo (en VPS) |
| SSL | Certbot + Let's Encrypt (auto-renew) |
| Monitorización | `/opt/scripts/healthcheck.sh` → Telegram bot SA99 (chat `6262984444`) |

---

## 0 · Pre-deploy: registrar en Notion

Antes del primer deploy, **registrar `agrom-web` en el Catálogo de Infraestructura SRS** (Notion `3217981f08ef81828e31edfcc9b78414`):

- **Sec. 2 (Catálogo de Proyectos)**: `AgroWeb · agrom-web · Nginx Alpine · 3180 · agrom.es · Servidor 1`
- **Sec. 4 (Convención de Puertos)**: `AgroWeb · +180 · 3180 · — · — · —`
- Actualizar "Siguiente offset libre" a **+190**

Si esto colisiona con un offset ya tomado, abortar y consultar antes.

---

## 1 · DNS (Hostinger)

Panel Hostinger → DNS de `agrom.es` → añadir:

| Tipo | Nombre | Valor | TTL |
|---|---|---|---|
| A | `@` | `72.62.41.234` | 14400 |
| A | `www` | `72.62.41.234` | 14400 |

Esperar propagación (5–30 min). Validar:

```bash
dig +short agrom.es
dig +short www.agrom.es
# Debe responder 72.62.41.234
```

---

## 2 · Deploy en Servidor 1 (`72.62.41.234`)

```bash
ssh root@72.62.41.234

# Crear directorio y clonar
cd /opt/apps
git clone https://github.com/gutierrezbj/agrom-web.git
cd agrom-web

# Levantar el container
docker compose up -d --build

# Verificar
docker ps | grep agrom-web                              # status: Up (healthy)
curl -s http://127.0.0.1:3180/health                    # → ok
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:3180/  # → 200

# ⚠️ SRS: verificar que NO está expuesto a 0.0.0.0
ss -tlnp | grep ':3180' | grep '0.0.0.0'                # debe estar vacío
```

---

## 3 · Nginx vhost (host externo, en el VPS)

```bash
cat > /etc/nginx/sites-available/agrom.es <<'NGINX'
server {
    listen 80;
    listen [::]:80;
    server_name agrom.es www.agrom.es;

    # Redirige www → apex (Certbot añadirá el bloque HTTPS arriba)
    location / {
        proxy_pass http://127.0.0.1:3180;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 30s;
    }

    # Endpoint para healthcheck.sh externo
    location = /health {
        proxy_pass http://127.0.0.1:3180/health;
        access_log off;
    }
}
NGINX

ln -sf /etc/nginx/sites-available/agrom.es /etc/nginx/sites-enabled/agrom.es
nginx -t
systemctl reload nginx
```

---

## 4 · SSL (Certbot / Let's Encrypt)

```bash
certbot --nginx -d agrom.es -d www.agrom.es --redirect --agree-tos -m johnj@agrom.es --no-eff-email
# El --redirect añade redirección 80→443 automática.

# Verificar
curl -sI https://agrom.es | head -1                # HTTP/2 200
curl -sI http://agrom.es | head -1                 # 301 → https://agrom.es
```

Renovación: cron de Certbot ya configurado en el VPS (`systemctl status certbot.timer`).

---

## 5 · Registrar en monitorización

Editar `/opt/scripts/healthcheck.sh` en el Servidor 1 y añadir al array `SERVICES`:

```bash
"AgroWeb|agrom-web|docker"
```

Esperar 5 min — debería llegar a Telegram (`SA99 Bot`, chat `6262984444`) una notificación UP.

Test manual del healthcheck:

```bash
bash /opt/scripts/healthcheck.sh
```

---

## 6 · Smoke test post-deploy

```bash
# Desde tu máquina
curl -sI https://agrom.es | head -1                                # 200
curl -s https://agrom.es | grep -c "AgroM"                         # >0
curl -s https://agrom.es/legal.html | grep -c "Aviso legal"        # >0
curl -sI https://agrom.es/assets/og-image.png | head -1            # 200

# Verifica visualmente:
# - https://agrom.es (Chrome + Safari iOS)
# - Header "Agro" + italic "M"
# - Stats strip: 35 € · 30+ ha · NPTA · Aceptamos emergencias
# - §004 Trazabilidad con pull-quote "Vendemos trabajo documentado"
# - Form en §007 abre Mail.app prerellenado a johnj@agrom.es
# - Footer link "Aviso legal" → legal.html funciona
```

---

## 7 · Updates posteriores

```bash
ssh root@72.62.41.234
cd /opt/apps/agrom-web
git pull
docker compose restart    # el container relee los volúmenes ro (no rebuild necesario)
```

Si cambias `docker-compose.yml` o `nginx.conf`:

```bash
docker compose up -d --build --force-recreate
```

---

## 8 · Rollback rápido

```bash
ssh root@72.62.41.234
cd /opt/apps/agrom-web
git log --oneline -5
git checkout <SHA-anterior>
docker compose restart
# (volver a main cuando esté fixed: git checkout main && git pull)
```

---

## 9 · Alternativa: deploy en Servidor 2 (`187.77.71.102`)

Ambos servidores SRS están en producción. agrom-web está en Servidor 1. Si en el futuro se mueve o duplica a Servidor 2, repetir §2 cambiando la IP. El offset +180 es el mismo en ambos — no choca porque cada VPS tiene su propio espacio de puertos.

---

## Referencias

- Cuaderno Base de Proyectos SRS: Notion `3407981f08ef816d9704db6dbff2299b`
- Catálogo de Infraestructura SRS: Notion `3217981f08ef81828e31edfcc9b78414`
- Regla #1: [CRITICAL_no_inventar](../../.claude/projects/-Users-juanguti-Library-CloudStorage-OneDrive-Personal-02-SR-docs-SRS---AGRO/memory/CRITICAL_no_inventar.md)
- Memoria: consult-base-proyectos-primero (rige sobre cualquier SETUP.md o bundle del repo)
