# HANDOVER · agrom-web

> Estado vivo del proyecto al **14-may-2026**. Antes de viaje del 15-may.
> Leer este archivo primero al volver de cualquier interrupción.

## ✅ Lo que está hecho y vivo

`https://agrom.es` operativa con SSL Let's Encrypt (válido hasta 2026-08-12, auto-renew configurado). HTTP → HTTPS redirect activo. `www.agrom.es` resuelve por CNAME al apex.

| Sistema | Estado |
|---|---|
| Repo [`gutierrezbj/agrom-web`](https://github.com/gutierrezbj/agrom-web) | 4 commits en `main`, último `20c40df` |
| Container `agrom-web` (Servidor 1, `/opt/apps/agrom-web/`) | nginx:1.27-alpine en `127.0.0.1:3180`, healthy |
| Nginx vhost `/etc/nginx/sites-available/agrom.es` | activo, proxy → `:3180` |
| DNS `agrom.es` (Hostinger panel) | A `@` → `72.62.41.234` (TTL 3000) · CNAME `www` → `agrom.es` |
| SSL `/etc/letsencrypt/live/agrom.es/` | E7 cert, expira 12-ago-2026 |
| `/opt/scripts/healthcheck.sh` | entrada `AgroWeb\|agrom-web\|docker` registrada, cron `*/5 *` |
| SA99 servers MongoDB | `vps-prod.projects.AgroWeb = {agrom-web, agrom.es}` |
| SA99 dashboard "Red de Servidores" | muestra `AgroWeb · agrom.es · agrom-web` en Servidor 1 ✓ |
| Catálogo Infraestructura SRS (Notion) | AgroWeb registrado §2/§4/§5/§7, offset siguiente +190 |

## 🔑 Datos operativos clave

- **Email único**: `johnj@agrom.es`
- **Teléfono**: `+34 605 498 659`
- **Servidor 1**: `ssh root@72.62.41.234` (acceso SSH key-based, ya configurado)
- **Repo path en VPS**: `/opt/apps/agrom-web/`
- **Container name**: `agrom-web`
- **Puerto interno**: `127.0.0.1:3180`
- **Offset SRS**: `+180`
- **Telegram bot alertas**: SA99 Bot (8693037155) · chat `6262984444`

## 📋 Pendientes (no bloqueantes, ordenados por prioridad)

### Bloqueantes blandos (antes de campaña marketing seria)
1. **`assets/og-image.png`** → es un placeholder generado con PIL. Diseñar imagen real 1200×630 con wordmark sobre `#f3f5ee` + claim. Sustituir el archivo y `git push`.
2. **Validar `30+ ha/jornada`** empíricamente tras las primeras 5 jornadas reales. Si la media < 30, ajustar copy a `Hasta 30 ha/jornada según cultivo`.
3. **Verificar "Cualificado · 90 h MAPA"** en la ficha técnica del piloto (bajo SVG hero). Si no es exacto, generalizar a `Cualificado · ROPO + RPAS`.
4. **Verificar "Mínimo operativo de jornada 500 €"** en §005 Tarifa.

### Mejoras (cuando haya tiempo)
5. Añadir `sitemap.xml` + `robots.txt`.
6. Activar **Vercel Analytics** o **Plausible** si quieres métricas reales (la landing hoy no tiene analytics).
7. Si el form `mailto` pierde leads (alguien sin cliente de correo no envía), migrar a **Web3Forms** o **Formspree** (free tier, ~250 envíos/mes).
8. Subir TTL DNS `agrom.es` de `3000` a `14400` cuando estés seguro de que el deploy es estable (mantenerlo bajo facilita rollbacks de DNS rápidos).

### Limpieza de docs externos
9. **Commit del seed file SA99**: `/Users/juanguti/dev/sa99/SA99/backend/app/modules/infra/service.py` tiene la actualización (rename + AgroWeb) pero no está commiteada. Ver §"Tareas que dejé sin commit" abajo.

## 🚀 Cómo retomar en 1 minuto desde otra máquina

```bash
# 1. Clonar y entrar
git clone https://github.com/gutierrezbj/agrom-web.git
cd agrom-web

# 2. Leer estos 3 archivos (en orden)
cat CLAUDE.md          # contexto general del proyecto
cat HANDOVER.md        # este archivo, estado vivo y pendientes
cat DEPLOY.md          # receta de deploy si hay que reinstalar todo

# 3. Si solo es cambiar copy / fix CSS:
# ... editar index.html / legal.html ...
git add -A && git commit -m "..." && git push
ssh root@72.62.41.234 'cd /opt/apps/agrom-web && git pull && docker compose restart'

# 4. Smoke test
curl -sI https://agrom.es | head -1
```

## 📜 Tareas que dejé sin commit (revisar antes de tocar)

### Repo `/Users/juanguti/dev/sa99/SA99/`
Archivo modificado pero NO commiteado:
- `backend/app/modules/infra/service.py` — diff: `+6 / -2`. Cambios:
  - `vps-prod.name`: "VPS Producción" → **"Servidor 1"**
  - `vps-staging.name`: "VPS Staging" → **"Servidor 2"**
  - `vps-prod.projects.AgroWeb` añadido

Para commitear:
```bash
cd /Users/juanguti/dev/sa99/SA99
git diff backend/app/modules/infra/service.py   # revisar
git add backend/app/modules/infra/service.py
git commit -m "infra(seed): rename VPS Producción/Staging → Servidor 1/2 + add AgroWeb"
git push
```

Esto asegura que un re-seed limpio (colección `servers` vacía) parta del estado correcto. La data viva en MongoDB ya está bien sin esto.

## 🛟 Si algo está caído al volver del viaje

### `https://agrom.es` no responde
```bash
ssh root@72.62.41.234
docker ps | grep agrom-web                 # ¿está UP?
docker logs agrom-web --tail 30
docker compose -f /opt/apps/agrom-web/docker-compose.yml restart
systemctl status nginx
nginx -t                                    # ¿sintaxis OK?
journalctl -u nginx -n 50                  # logs nginx recientes
```

### SSL caducó o falla
```bash
ssh root@72.62.41.234
certbot certificates                       # ver estado
certbot renew --dry-run                    # test renovación
certbot renew                              # forzar renovación
systemctl reload nginx
```

### El healthcheck alerta de `agrom-web DOWN`
```bash
ssh root@72.62.41.234
bash /opt/scripts/healthcheck.sh           # ejecutar manual
tail -20 /var/log/srs-healthcheck.log      # ver historial
```

### El dashboard SA99 deja de mostrar AgroWeb
Verificar la colección `servers`:
```bash
ssh root@72.62.41.234 'docker exec sa99-mongo mongosh -u sa99 -p sa99dev --authenticationDatabase admin --quiet --eval "use(\"sa99\"); JSON.stringify(db.servers.findOne({_id:\"vps-prod\"}).projects.AgroWeb, null, 2)"'
```
Si falta, re-aplicar el `db.servers.updateOne` documentado en el "Checklist de Kickoff — Protocolo" (Notion `3257981f08ef8191b135d5da2bc759d1`, sección "Registro en SA99 InfraService").

### Bug nginx `gzip duplicate` aparece de nuevo
Significa que alguien volvió a tocar `/etc/nginx/nginx.conf` añadiendo el bloque `gzip` que se quitó el 14-may. La fuente canónica es `/etc/nginx/conf.d/srs-compression.conf`. Backup de la config buena: `/etc/nginx/nginx.conf.bak.2026-05-14-pre-gzip-fix`. Para arreglar:
```bash
sed -i 's|^\([[:space:]]*\)gzip on;|\1# gzip on; -- moved to conf.d/srs-compression.conf|' /etc/nginx/nginx.conf
# ... (resto del fix documentado en el log de esta sesión)
nginx -t && systemctl reload nginx
```

## 📚 Referencias canónicas (Notion)

- **Catálogo de Infraestructura SRS** (estado vivo del parque): [3217981f08ef81828e31edfcc9b78414](https://www.notion.so/3217981f08ef81828e31edfcc9b78414)
- **Spec v2 aplicado**: [Web AgroM v2 — Spec de reescritura](https://www.notion.so/3607981f08ef81a4bdccdedc74455c0b)
- **Identity tokens v2 aplicados**: [Identity Tokens v2](https://www.notion.so/35e7981f08ef81d9849bda7482e35828) — **OJO**: el spec original decía `--surface: #fdf8f0` (earth-50). Lo correcto extraído del CSS vivo de FitoLink es `#f3f5ee` (brand-50). Si futuras revisiones contradicen, mantener el valor canónico de FitoLink live.
- **Cuaderno de Protocolos AgroM** (lecciones operativas): [35d7981f08ef81329a43f0daea4447c1](https://www.notion.so/35d7981f08ef81329a43f0daea4447c1)
- **Plantilla Checklist Kickoff SRS**: [3257981f08ef8191b135d5da2bc759d1](https://www.notion.so/3257981f08ef8191b135d5da2bc759d1)

## ⚠️ Memorias de Claude Code que rigen aquí

Cargadas automáticamente en cada sesión de este workspace:
- `CRITICAL_no_inventar.md` — regla #1 del proyecto. Verificar copy contra realidad.
- `consult-base-proyectos-primero.md` — antes de bootstrap, consultar el Catálogo SRS, no los SETUP.md sueltos.
- `srs-servidor-1-servidor-2.md` — nomenclatura correcta de servidores.

---

**Última edición**: 14-may-2026 antes del viaje del 15-may. Buen viaje 🛸
