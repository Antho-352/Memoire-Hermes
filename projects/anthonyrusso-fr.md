# anthonyrusso.fr — Site Consulting SEO Personnel

## Identité

- **Path local** : `/Users/anthonyrusso/anthonyrusso-fr/`
- **Domaine** : https://anthonyrusso.fr
- **GitHub** : Antho-352/anthonyrusso-fr
- **Type** : Site consulting SEO perso
- **Stack** : Astro 6 SSR + WordPress headless (wp.anthonyrusso.fr) + PM2 + Node.js 22+

## Architecture rendu

**SSR (Server-Side Rendered)** :
- `/` — Homepage (3 derniers posts WP)
- `/blog` — Index blog paginé (WP)
- `/blog/[slug]` — Articles WordPress
- `/[...path]` — Catch-all WordPress permalinks

**Prerendered (statique au build)** :
- `/netlinking-sur-mesure/*` — Pages evergreen SEO
- `/audit-netlinking/*` — Pages evergreen audit
- `/mon-reseau`, `/resultats`, `/mentions-legales`
- `/404` — page jeu canvas

Prérendu via `export const prerender = true;` dans le frontmatter.

## WordPress headless

- API : `https://wp.anthonyrusso.fr/wp-json/wp/v2`
- Redirect subdomain configuré ✅ (voir rules/wordpress.md)
- Contenu sanitisé via `src/lib/html.ts` avant rendu

## Déploiement

- **Serveur** : cPanel shared hosting avec SSH
- **Path serveur** : `/home/anth543/anthonyrusso-fr/`
- **Port** : 3006 (`HOST=0.0.0.0`)
- **PM2** : process `anthonyrusso`, lance `dist/server/entry.mjs`
- **Apache proxy** : IPv6 `[::1]:3006`

```bash
# Via GitHub (recommandé)
git add . && git commit -m "message" && git push
ssh anth543@server "cd /home/anth543/anthonyrusso-fr && git pull && npm run build && npx pm2 restart anthonyrusso"

# Via deploy.sh (build local + scp + restart)
./deploy.sh
```

**Env vars** : `WORDPRESS_API_URL`, `HOST=0.0.0.0`, `PORT=3006`

## Layouts

1. `BaseLayout` — `<head>`, SEO, theme toggle, navigation
2. `PageLayout` — extends BaseLayout + CalendlyWidget sidebar
3. `BlogPostLayout` — extends BaseLayout, structure blog

## Composants interactifs

- **404 Game** : canvas-based catching game avec net mechanic
- **Checklists interactives** (A1, E8) : vanilla JS, checkboxes, CTAs conditionnels
- **Link Gap table** (A2) : tableau éditable, export CSV, toggle priorités

## Structured data

JSON-LD sur les evergreen pages : Article, FAQPage, BreadcrumbList.
Pattern : définir dans le frontmatter, stringifier dans `<script type="application/ld+json">`.

## Sitemap dynamique

- `/sitemap.xml` — généré dynamiquement (fetch WP posts + pages statiques)
- Caché 1h (Cache-Control)
- L'intégration @astrojs/sitemap est désactivée (ne peut pas inclure les pages SSR blog)

## Problèmes courants

- **403 WP API** : vérifier CORS headers sur wp.anthonyrusso.fr
- **503 prod** : Apache proxy mal configuré ou PM2 down → `pm2 status` + Apache error logs
- **Pages prerendered pas à jour** : rebuild + restart PM2
- **Blog posts absents** : tester `curl https://wp.anthonyrusso.fr/wp-json/wp/v2/posts?per_page=1`
- **Scores jeu non sauvés** : vérifier `data/scores.json` writable. Rate limit : 5 req/min par IP
