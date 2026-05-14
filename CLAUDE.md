# agrom-web — Claude Code Project Context

> Documento que Claude Code lee al abrir este repo. Mantener actualizado al final de cada sesión.

## What is this

Landing comercial estática de **AgroM** — servicio de aplicación aérea agrícola con DJI Agras T50 bajo paraguas operacional Drovinci. Single-page HTML + Tailwind CDN, sirviendo en `https://agrom.es` desde el 14-may-2026.

## Current State

- **Sprint actual**: v1.0 producción
- **Version**: commit [`20c40df`](https://github.com/gutierrezbj/agrom-web/commit/20c40df) en `main`
- **Live**: `https://agrom.es` + `https://www.agrom.es` (SSL Let's Encrypt, expira 2026-08-12)
- **Servidor**: Servidor 1 (`72.62.41.234`, Hostinger, Ubuntu 24.04). Offset SRS **+180** → container nginx alpine en `127.0.0.1:3180`.
- **Monitorizado en**: `/opt/scripts/healthcheck.sh` (cron `*/5 *`, alerta Telegram bot SA99) + dashboard SA99 "Red de Servidores"
- **Registrado en**: Catálogo de Infraestructura SRS (Notion §2/§4/§5/§7)
- **TODOs abiertos**: ver `HANDOVER.md` §Pendientes

## Tech Stack

- HTML estático puro (sin build, sin Node, sin framework)
- Tailwind CSS vía Play CDN (`cdn.tailwindcss.com`) configurado inline
- Google Fonts: **Instrument Serif** (display) · **DM Sans** (body) · **IBM Plex Mono** (mono)
- Identidad visual: FitoLink design system v2 (heredada)
- Container: `nginx:1.27-alpine` con HTML/assets montados como volúmenes `:ro`
- Reverse proxy: Nginx host (en el VPS) con SSL Certbot

## Project Structure

```
agrom-web/
├── index.html              # landing v2 (1100 líneas, monolítico)
├── legal.html              # aviso legal + privacidad RGPD (LOPDGDD)
├── docker-compose.yml      # container nginx alpine offset +180
├── nginx.conf              # vhost interno del container
├── DEPLOY.md               # receta paso a paso de despliegue SRS
├── HANDOVER.md             # estado vivo del proyecto (leer primero)
├── CLAUDE.md               # este archivo
├── README.md               # descripción pública del repo
├── LICENSE                 # All rights reserved
├── .gitignore
└── assets/
    └── og-image.png        # 1200×630 placeholder Open Graph
```

## Key Patterns

### Regla #1 — CRITICAL_no_inventar
Esta landing está **auditada contra capacidades reales**. NO añadir copy que prometa lo siguiente (son AgroOps Sprint 1 o FitoLink, NO AgroM):
- ❌ Geofence digital · Telemetría AEMET en vivo · Firma digital · Informe técnico digital · NDVI/Sentinel · Cuaderno de campo digital · "Evidencia técnica defendible" · Conservación 5 años

Lo que SÍ prometemos (verificado contra operación real):
- ✅ DJI Agras T50 bajo NPTA Drovinci · ROPO + RPAS · Verificación meteo previa AEMET · Albarán firmado en finca · Parte de trabajo archivado **1 año** · Marco RD 1311/2012 · RC aeronáutica

Smoke test del copy (debe devolver 0 líneas):
```bash
grep -i "geofence\|telemetría\|firma digital\|informe digital\|NDVI\|sentinel\|5 años\|trazabilidad documentada\|comercial@agrom\|600 000 000" index.html legal.html
```

### Identidad — 8 tokens FitoLink

| Token | Hex | Uso |
|---|---|---|
| `--primary` | `#46632e` | Verde principal (CTAs, eyebrow) |
| `--primary-dark` | `#354b23` | Hover |
| `--surface` | `#f3f5ee` | **brand-50** — fondo body (NO usar `#fdf8f0` earth-50, ese es para cards) |
| `--surface-elevated` | `#f5e6cc` | Fondo cards / parch |
| `--border` | `#d4a85a` | Hairlines |
| `--text-strong` | `#18230f` | Texto principal |
| `--text-soft` | `#6b7280` | Labels soft |
| `--brand-accent` | `#d45220` | Terra / acentos (solo texto grande — falla AA en body 14px) |

### Tipografía

- **Instrument Serif** solo tiene Regular 400 + Italic 400 (NO es variable, NO hay weight 900). El énfasis dramático se logra con `font-style: italic`.
- **DM Sans** body 15px / line-height 1.5 / default weight 400, 500 para énfasis.
- **Wordmark**: `Agro` Regular + `M` Italic (header + footer + favicon SVG inline).

### Form de contacto
Sin backend. `onsubmit` JS construye un `mailto:johnj@agrom.es` prerellenado con los campos. RGPD: link al `legal.html` debajo del botón con copy honesto sobre el comportamiento.

## Deploy

Una sesión normal (cambio de copy / fix CSS):

```bash
# Local (esta máquina)
cd ~/Desktop/Repositorios_GIT/agrom-web
# ... editar index.html / legal.html / assets ...
git add -A && git commit -m "..."
git push origin main

# Servidor 1 (aplicar cambios vivos)
ssh root@72.62.41.234
cd /opt/apps/agrom-web
git pull
docker compose restart   # el container monta los HTML como volume :ro
# (no rebuild necesario salvo que cambies docker-compose.yml o nginx.conf)
```

Smoke test post-deploy:

```bash
curl -sI https://agrom.es | head -1                        # HTTP/2 200
curl -s https://agrom.es | grep -c "AgroM"                 # >0
curl -s https://agrom.es/health                            # ok
ssh root@72.62.41.234 'docker ps | grep agrom-web'         # Up (healthy)
```

Rollback:
```bash
ssh root@72.62.41.234
cd /opt/apps/agrom-web
git log --oneline -5
git checkout <SHA-anterior>
docker compose restart
```

Receta completa de **primer despliegue** (clonar, vhost, Certbot, healthcheck): ver `DEPLOY.md`.

## Convenciones que rigen aquí

1. **Regla #1 del proyecto**: `CRITICAL_no_inventar.md` (memoria del SRS) — verificar todo copy contra realidad operativa.
2. **Catálogo de Infraestructura SRS** (Notion `3217981f08ef81828e31edfcc9b78414`) — fuente canónica de offsets, servidores, dominios. Antes de bootstrap consultar ahí, NO los SETUP.md de cualquier bundle.
3. **Servidores SRS**: `Servidor 1` (72.62.41.234) y `Servidor 2` (187.77.71.102), ambos producción. NUNCA usar `PROD`/`STAGING` como nombre de servidor (corregido 14-may-2026).
4. **Email único de operación**: `johnj@agrom.es` · Teléfono: `+34 605 498 659`.
