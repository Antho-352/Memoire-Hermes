# Linkttp — Marketplace B2B Backlinks & Sites Web

## Identité

- **Path local** : `/Users/anthonyrusso/link-ttp/`
- **Domaine** : https://linkttp.com
- **Type** : B2B marketplace (backlinks + sites à vendre + campagnes netlinking)
- **GitHub** : `Antho-352/link-ttp` (repo privé)
- **Stack** : Next.js 15 (App Router) + next-intl v4 + Drizzle ORM + NextAuth v5 + PayPal + Resend + reCAPTCHA v3

## Architecture

### Structure clé
```
src/app/[locale]/(public)/  — Pages publiques (bilingual EN/FR)
src/app/(auth)/             — Auth pages (/signin, no i18n)
src/app/admin/              — Admin back-office (EN only)
src/app/api/                — API routes
src/lib/actions/            — Server Actions
src/lib/validation/         — Zod schemas
src/lib/db/                 — Drizzle schema + connection
```

### Internationalisation
- **EN** (défaut, pas de préfixe) + **FR** (`/fr/` préfixe)
- Navigation/redirect : toujours `Link` et `redirect` de `@/i18n/navigation` (pas `next/link`)
- Colonnes DB localisées : `*Fr` suffix (`descriptionFr`, `nameFr`, etc.)
- Helper : `localized(item, 'description', locale)` → retourne `descriptionFr` si locale='fr'

### Design system — 3 Gems
- **Emerald** (#0D9263) — backlinks, CTAs primaires
- **Ruby** (#E84040) — sites à vendre, CTAs conversion
- **Sapphire** (#2563EB) — campagnes netlinking
- Composant : `<GemBadge tier="..." />`

### Pages publiques
- `/` Homepage, `/backlinks` catalogue + `/backlinks/[id]` détail
- `/buy-websites` catalogue + `/buy-websites/[id]` détail
- `/netlinking-campaign` campagne full-service
- `/cart` panier, `/order/[slug]`, `/order/multi`, `/order/pack/[slug]`
- `/order/payment`, `/order/confirmation`, `/order/success`
- `/track` tracking commandes (ID-based), `/watchlist`, `/how-it-works`
- `/packs/[slug]`, `/terms-of-service`, `/privacy`, `/legal`, `/[slug]` CMS

### Client-side state (localStorage)
- `linkttp_watchlist` — sites sauvegardés
- `linkttp_cart` — panier
- `linkttp_hearts` / `linkttp_hearts_reset` — gamification (5 cœurs, reset 24h)

## Déploiement

- **Path serveur** : `/home/linkttp/repositories/link-ttp`
- **PM2** : `link-ttp` sur port 3005
- **Deploy** : `./deploy.sh` depuis machine locale
- `.env.production` non versionné — doit être copié dans `.next/standalone/` post-build
- Logs : `pm2 logs link-ttp`

```bash
./deploy.sh                # Full deploy
./deploy.sh --skip-deps    # Skip npm install
./deploy.sh --skip-db      # Skip db:push
```

**Build en cas de crash** (ENOENT .next) :
```bash
ssh server "cd /home/linkttp/repositories/link-ttp && rm -rf .next && npm run build"
```

## Crons actifs (à compléter sur serveur)

```bash
*/5 * * * * /home/linkttp/monitoring/system-monitor.sh   # ✅ Monitoring 5min
0 3 * * * curl ... /api/cron/sync-metrics                 # ✅ SEO sync daily
0 4 * * 1 curl ... /api/cron/sync-metrics?mode=full       # ✅ SEO full sync hebdo
0 * * * * curl ... /api/cron/uptime                       # ⚠️ À ajouter
0 9 * * * curl ... /api/cron/tf-alerts                    # ⚠️ À ajouter
```

## Services externes

- **PayPal** : PAYPAL_CLIENT_ID, PAYPAL_SECRET
- **Resend** : RESEND_API_KEY (emails transactionnels)
- **Google Search Console** : GSC_SERVICE_ACCOUNT_EMAIL, GSC_PRIVATE_KEY
- **Bing Webmaster** : BING_WEBMASTER_API_KEY
- **DomDetailer** : DOMDETAILER_API_KEY (TF/CF/RD + Moz DA/PA)
- **reCAPTCHA v3** : NEXT_PUBLIC_RECAPTCHA_SITE_KEY, RECAPTCHA_SECRET_KEY
- **Telegram** : TELEGRAM_BOT_TOKEN, TELEGRAM_CHAT_ID (notifications admin)
- **Brevo** : BREVO_API_KEY, BREVO_LIST_ID (email marketing)

## Sécurité — règles obligatoires

```
Chaque nouvelle server action / API route DOIT :
1. Authentifier : requireAdmin() pour mutations admin
2. Valider : Zod .parse() sur toutes les données entrantes
3. Sanitiser : escapeHtml() emails, sanitizeHtml() CMS dangerouslySetInnerHTML
4. Rate limiter : rateLimit() sur les endpoints publics
```

- SQL : toujours Drizzle ORM query builders, jamais raw SQL string concatenation
- XSS : un seul `dangerouslySetInnerHTML` → CMS pages, toujours sanitized
- Uploads : magic bytes validation (`file-type`), filename regenerated `${createId()}${ext}`
- Auth : NextAuth v5 JWT, `timingSafeEqual()` pour comparaison passwords

## Fonctionnalités désactivées (code en place, facilement réactivable)

- **Sword Cursor** : uncomment import + `<SwordCursor />` dans `src/app/(public)/layout.tsx`
- **Fairy Helper** : uncomment import + `<FairyHelper />` dans `src/app/(public)/layout.tsx`
- **Map page** : re-ajouter `{ name: 'Map', href: '/map' }` dans navItems (PublicHeader.tsx) et footerLinks.resources (PublicFooter.tsx)

## Convention git

- Parenthèses dans les paths : `git add "src/app/(public)/page.tsx"` (guillemets obligatoires)
- DB schema change → toujours `npm run db:push` sur serveur après deploy

## Network Dashboard

Path admin : `/admin/network`
- Monitoring CPU/RAM/disk (Node.js `os` module + shell)
- Uptime check horaire via cron
- Sales tracking avec CSV import
- Trust Flow monitoring avec alertes Telegram
- GSC URL Inspection API (max 50 URLs, cooldown 7j)

Seuils monitoring :
- CPU : ⚠️ >10, 🚨 >11 (sur 12 threads)
- RAM : ⚠️ <10% libre (~6GB), 🚨 <5% libre de 64GB
- Disk : ⚠️ >85%, 🚨 >90% de 416GB
