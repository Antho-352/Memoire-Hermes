# frigo-labo.fr — Site B2B Équipement Froid Laboratoires

## Identité

- **Path local** : `/Users/anthonyrusso/frigo-labo/frigo-labo-astro/`
- **Domaine** : frigo-labo.fr
- **Email** : contact@frigo-labo.fr
- **Type** : Site vitrine B2B (équipement froid laboratoires et pharmaceutique)
- **Stack** : Astro 5 + Tailwind CSS 4 + WordPress headless (WPGraphQL)

## Architecture

- SSG par défaut
- Blog WordPress via `wp.frigo-labo.fr` (intégré WPGraphQL)
- 8 pages .astro dont blog `/blog/index.astro` + `/blog/[slug].astro`
- Formulaire de contact PHP simple (`public/contact-handler.php`)

## Fonts

Inter (300-800), self-hosted (Google Fonts chargée depuis BaseLayout).

## Pages (8)

`/index`, `/gamme-medicale`, `/laboratoires-scientifiques`, `/caracteristiques-innovations`, `/services-maintenance`, `/a-propos`, `/contact`, `/blog`

## Redirect subdomain

⚠️ `wp.frigo-labo.fr` → pas encore configuré. À faire avant mise en production.

## Structure

```
src/layouts/BaseLayout.astro
src/components/
  Header, Footer, MobileMenu, ScrollReveal
  Hero, Card, CardGrid, CtaBanner, FaqAccordion, ContactForm
src/pages/  (8 pages + blog/)
src/styles/global.css  (Tailwind 4 @theme + custom CSS)
```
