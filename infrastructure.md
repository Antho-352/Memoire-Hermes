# Infrastructure — Serveurs, Sites, Hébergement

## Hébergement principal

- **OVHcloud Kimsufi** — dédié LEMP (Linux/Nginx/MySQL/PHP), héberge la majorité des sites WP
- **cPanel/WHM** — interface de gestion (version 134.0.20), Debian, Apache 2.4.66
- **Cloudflare** — proxy actif sur tous les sites (vraies IPs conservées via mod_remoteip)
- **Kimsufi cPanel** — héberge seoflix.fr (même patterns que les autres sites WP)

## Serveur dédié

- **OS** : Debian (OVH dédié)
- **Panel** : cPanel/WHM 134.0.20
- **Apache** : 2.4.66
- **PHP** : 8.1+ (strict types)
- **MySQL** : 8.0+
- **mod_remoteip** : auto-configuré par cPanel → vraies IPs des crawlers même derrière Cloudflare
- **Logs Apache** : `/usr/local/apache/domlogs/<domain>` + `<domain>-ssl_log`
- **Archives logs** : `/home/<cpanel_user>/logs/<domain>-Mois-Année.gz`

## Sites et domaines actifs

| Domaine | Stack | Statut | Path local |
|---------|-------|--------|------------|
| anthonyrusso.fr | Astro 6 SSR + WP headless | Live | `/Users/anthonyrusso/anthonyrusso-fr/` |
| linkttp.com | Next.js 15 + Drizzle + PG | Live | `/Users/anthonyrusso/link-ttp/` |
| seoflix.fr | WP + plugin custom + thème FSE | En cours | `/Users/anthonyrusso/seoflix/` |
| lecampdubivouaqueur.fr | WP + ARW Pulse + arw-pack-outdoor | Live | generatepress/ |
| lesdelicesdeb.fr | WP + ARW Pulse + arw-pack-delices | En cours | generatepress/ |
| citymoto.fr | WP + ARW Pulse (skin urban) | Pas encore live | generatepress/ |
| vai-calcio.fr | Astro 5 + WP headless | En cours | `/Users/anthonyrusso/calcio/` |
| amenagement-labo.fr | Astro 5 + Tailwind 4 + WP headless | Live | `/Users/anthonyrusso/amenagement-labo/amenagement-labo-fr/` |
| frigo-labo.fr | Astro 5 + Tailwind 4 + WP headless | En cours | `/Users/anthonyrusso/frigo-labo/frigo-labo-astro/` |
| simulation-immobilier.fr | WP custom theme + plugin | En cours | `/Users/anthonyrusso/immo/simulation-immobilier/` |
| wp.anthonyrusso.fr | WordPress headless (blog) | Live | Serveur OVH |
| wp.frigo-labo.fr | WordPress headless (blog) | En cours | Serveur OVH |
| wp.psm-labo.fr | WordPress headless | Live | Serveur OVH |
| wp.amenagement-labo.fr | WordPress headless | Live | Serveur OVH |
| wp.vai-calcio.fr | WordPress headless | En cours | Serveur OVH |

## Thème ARW Pulse — sites utilisant le thème

| Site | Skin | Pack niche | Notes |
|------|------|------------|-------|
| citymoto.fr | urban.json | — (patterns défaut) | Moto urbaine, affiliation |
| lecampdubivouaqueur.fr | field.json | arw-pack-outdoor | Outdoor, spots/trails, Leaflet |
| lesdelicesdeb.fr | citrus.json | arw-pack-delices | Cuisine-performance, médias |

## Architecture standard Astro + WP headless

- **Site principal** : Astro 5+ statique sur `site.fr`
- **Blog WordPress** : headless WP sur `wp.site.fr` avec WPGraphQL
- **Critique** : subdomain WP DOIT rediriger vers domaine principal via .htaccess (voir rules/wordpress.md)

## Déploiements

### Astro/Node (anthonyrusso.fr, vai-calcio.fr)

- PM2 process manager
- Apache reverse proxy → Node.js
- SSH : `git pull && npm run build && pm2 restart <name>`

### anthonyrusso.fr spécifique

- Path serveur : `/home/anth543/anthonyrusso-fr/`
- Port : 3006 (`HOST=0.0.0.0`)
- Apache proxy : `/etc/apache2/conf.d/userdata/ssl/2_4/anth543/anthonyrusso.fr/proxy.conf`
- IPv6 : `[::1]:3006` (Node.js bind IPv6 par défaut)

### linkttp.com

- Path serveur : `/home/linkttp/repositories/link-ttp`
- PM2 process : `link-ttp` sur port 3005
- Deploy : `./deploy.sh` depuis machine locale
- Logs : `pm2 logs link-ttp`
- Crons actifs : monitoring 5min, SEO sync daily 3h, SEO full sync weekly lundi 4h

### WordPress (upload zip)

- Pas de CLI disponible sur le serveur pour certains projets (seoflix, sites immo)
- Upload zip via WP Admin → Extensions/Thèmes → Téléverser
- Zip avec dossier parent obligatoire (voir rules/wordpress.md)

## Redirect WordPress subdomain (obligatoire sur tous les sites Astro+WP headless)

Voir `rules/wordpress.md` pour le pattern .htaccess complet. Appliqué à :
- ✅ anthonyrusso.fr → wp.anthonyrusso.fr
- ✅ psm-labo.fr → wp.psm-labo.fr
- ✅ amenagement-labo.fr → wp.amenagement-labo.fr
- ⚠️ frigo-labo.fr → wp.frigo-labo.fr (pas encore configuré)

## Projets locaux (paths machine Anthony)

```
/Users/anthonyrusso/
├── Memoire-Hermes/           ← ce repo
├── generatepress/            ← ARW Pulse thème + packs niche
│   ├── arw-pulse/            ← thème principal
│   ├── arw-pack-outdoor/
│   ├── arw-pack-delices/
│   ├── arw-pack-maison/
│   └── arw-pack-cuendet/
├── link-ttp/                 ← Linkttp marketplace
├── seoflix/                  ← Seoflix aggregateur
├── anthonyrusso-fr/          ← Site perso Astro
├── calcio/                   ← Vai calcio
├── immo/                     ← Workspace immobilier
├── audit-logs/               ← Pipeline audit logs SEO/GEO
├── amenagement-labo/
├── frigo-labo/
├── kiosque-media/
└── web-factory/              ← Système multi-agents web builder
```
