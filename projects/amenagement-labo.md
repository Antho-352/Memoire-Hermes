# amenagement-labo.fr — Site B2B Aménagement Laboratoires

## Identité

- **Path local** : `/Users/anthonyrusso/amenagement-labo/amenagement-labo-fr/`
- **Domaine** : amenagement-labo.fr
- **Email** : contact@amenagement-labo.fr
- **Type** : Site vitrine B2B (aménagement et équipement de laboratoires)
- **Stack** : Astro 5 + Tailwind CSS 4 + WordPress headless (WPGraphQL)

## Architecture

- SSG par défaut
- Blog WordPress via `wp.amenagement-labo.fr` (pas encore connecté au dernier état connu)
- 10 pages .astro
- Formulaire de contact PHP simple (`public/contact-handler.php`)

## Design

| Token | Valeur | Usage |
|-------|--------|-------|
| `primary` | `#1E4D3A` | Vert foncé — titres, backgrounds, CTA |
| `secondary` | `#C9A96E` | Or — accents, badges, séparateurs |
| `bg-alt` | `#FAFAF8` | Fond alternatif |
| `text-dark` | `#171717` | Texte courant |

Fonts : Cormorant Garamond (headings) + Inter (body), Google Fonts.

## Pages (10)

`/`, `/conception-laboratoires`, `/installation-equipements-techniques`, `/revetements-finitions`, `/consommables-equipements`, `/normes-reglementations`, `/secteurs-activite`, `/projets-realisations`, `/conseils-actualites`, `/contact-devis`

## Composants clés

- `BaseLayout` — head, SEO, nav, footer, MobileMenu, ScrollReveal
- `Hero` — titre (supports `set:html`), sous-titre, bgImage, CTAs, overlay, badge, height
- `CtaBanner`, `FaqAccordion`, `CardGrid`, `Card`, `SectorGrid`, `SpecTable`, `ContactForm`, `LabSimulator`

## Redirect subdomain

✅ Configuré (wp.amenagement-labo.fr → amenagement-labo.fr)
