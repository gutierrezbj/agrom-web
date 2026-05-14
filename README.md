# agrom-web

Landing estática de **AgroM** — servicio de aplicación aérea agrícola con DJI Agras T50 bajo paraguas operacional Drovinci, en España.

Deploy: <https://agrom.es>

---

## Stack

- **HTML estático puro** (sin build, sin Node, sin framework).
- **Tailwind CSS** vía Play CDN (`cdn.tailwindcss.com`), configurado inline.
- **Google Fonts**: Instrument Serif (display) · DM Sans (body) · IBM Plex Mono (mono).
- **Identidad visual**: FitoLink design system v2 (heredada en AgroM).
- Single page (`index.html`) + página legal (`legal.html`).

## Servir local

```bash
cd agrom-web
python3 -m http.server 8000
# → http://localhost:8000
```

O abrir directamente `index.html` en navegador (algunas funciones —favicon data URI, mailto handler— funcionan mejor sirviendo vía HTTP).

## Estructura

```
agrom-web/
├── index.html         # landing v2 (página principal)
├── legal.html         # aviso legal + privacidad + cookies (RGPD)
├── README.md          # este archivo
├── LICENSE            # All rights reserved
├── .gitignore         # excluye .DS_Store, .vscode/, etc.
└── assets/
    └── og-image.png   # 1200x630 para Open Graph (TODO v1.0 — placeholder)
```

Sin `package.json`. Sin Node. Sin build.

## Deploy

Recomendado: **Vercel**.

```bash
npm i -g vercel       # si no está instalado
vercel                # vincula el repo, primer deploy en *.vercel.app
# Vercel Dashboard → Settings → Domains → add "agrom.es"
# Copiar A 76.76.21.21 al DNS Hostinger (raíz @) + CNAME www → cname.vercel-dns.com
# HTTPS automático vía Let's Encrypt
```

Alternativas: GitHub Pages, Cloudflare Pages, VPS Hostinger.

## Reglas de copy (CRITICAL_no_inventar)

Esta landing está auditada contra la regla #1 del proyecto:
**no afirmar capacidades que no existen en producción**.

**Vocabulario prohibido en el copy** (estas capacidades son de AgroOps Sprint 1 o FitoLink — NO de AgroM):

- ❌ Geofence digital del vuelo
- ❌ Telemetría AEMET en vivo
- ❌ Firma digital del agricultor
- ❌ Informe técnico digital con NDVI / Sentinel-2
- ❌ Conservación 5 años (lo real es 1 año)
- ❌ Cuaderno de campo digital
- ❌ "Evidencia técnica defendible" (lenguaje legal inflado)

**Lo que sí prometemos** (todo verificado contra realidad operativa):

- ✅ DJI Agras T50 bajo NPTA Drovinci
- ✅ Piloto cualificado, ROPO + RPAS
- ✅ Verificación meteorológica previa al vuelo (AEMET)
- ✅ Albarán firmado en finca por el agricultor
- ✅ Parte de trabajo archivado **1 año**
- ✅ Marco RD 1311/2012, RC aeronáutica

Antes de tocar copy: revisar el spec v2 en Notion y la memoria `CRITICAL_no_inventar.md`.

## Identidad

### 8 tokens FitoLink

| Token | Hex | Uso |
|---|---|---|
| `--primary` | `#46632e` | Verde principal (CTAs, eyebrow) |
| `--primary-dark` | `#354b23` | Hover |
| `--surface` | `#fdf8f0` | Fondo body |
| `--surface-elevated` | `#f5e6cc` | Fondo cards |
| `--border` | `#d4a85a` | Hairlines |
| `--text-strong` | `#18230f` | Texto principal |
| `--text-soft` | `#6b7280` | Labels soft |
| `--brand-accent` | `#d45220` | Terra / acentos |

### Tipografía

- **Display**: Instrument Serif (Regular 400 + Italic 400 — no es variable, no hay weight 900). El énfasis dramático se logra con `font-style: italic`.
- **Body**: DM Sans (300–700 variable). Default 400, body 15px / line-height 1.5.
- **Mono**: IBM Plex Mono (300/400/500/600).

### Wordmark

`Agro` Regular + `M` Italic. Aplicado en header, footer y favicon SVG inline.

## TODOs visibles

- [ ] Validar empíricamente `30+ ha/jornada` tras las primeras 5 jornadas reales. Si la media operativa < 30, ajustar a `Hasta 30 ha/jornada según cultivo`.
- [ ] Sustituir `assets/og-image.png` (placeholder) por imagen real 1200×630 con el wordmark sobre `#fdf8f0`.
- [ ] Verificar que `Cualificado · 90 h MAPA` en la ficha técnica del piloto (bajo el SVG hero) es exacto. Si no, generalizar a `Cualificado · ROPO + RPAS`.
- [ ] Verificar `Mínimo operativo de jornada 500 €` en §005 Tarifa.
- [ ] Añadir `sitemap.xml` + `robots.txt`.
- [ ] Activar Vercel Analytics o Plausible si se quieren métricas.
- [ ] Si el `mailto` fallback pierde leads, migrar a Web3Forms / Formspree (free tier).

## Licencia

Ver [LICENSE](LICENSE). Todos los derechos reservados.

## Commit history

- `feat: web AgroM v2 — identidad FitoLink + copy auditado` — migración v1 → v2 (tokens, fuentes, copy auditado, SVG limpio, form mailto, legal RGPD).
