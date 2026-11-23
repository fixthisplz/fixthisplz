# FixThisPlz — Brand kit (v3)

Identité visuelle vectorisée à partir de l'**icône approuvée**, et non d'une nouvelle interprétation.

## Le dessin maître

- **4 coins violets** de capture (haut gauche, haut droit, bas gauche, bas droit).
- **Cercle rouge irrégulier** retracé depuis le visuel validé, avec le petit retour de trait au sommet.
- **Point d'exclamation** sombre.
- **3 traits en éventail de couleurs distinctes** : corail, violet et orange.
- **Pastille verte et check blanc**, sans dégradé.
- Aplats de couleurs, aucune ombre ni lueur dans les fichiers vectoriels.

## Palette

| Usage | Valeur hex |
|---|---|
| Capture / violet | `#6825D5` |
| Annotation / rouge | `#F74445` |
| Accent orange | `#FF7B22` |
| Résolu / vert | `#0BC75B` |
| Encre / texte | `#21153E` |
| Fond sombre | `#1B1530` |

Déclinaison pour fonds sombres : violet `#B696FF`, corail `#FF6565`, orange `#FF9B4B`, vert `#23D46D`.

## Utilisation recommandée

| Support | Fichier conseillé |
|---|---|
| GitHub organisation | `png/avatars/github-light-512.png` |
| Avatar sombre | `png/avatars/github-dark-512.png` |
| npm | `png/avatars/npm-light-512.png` |
| README clair | `svg/logos/logo-horizontal-light.svg` |
| README sombre | `svg/logos/logo-horizontal-dark.svg` |
| Dashboard | `svg/icons/icon-primary-light.svg` |
| Favicon | `web/favicon.svg` ou `web/favicon.ico` |
| iOS / PWA | `web/apple-touch-icon.png` / `web/icon-192.png` / `web/icon-512.png` |
| Bouton Signaler | `svg/icons/icon-widget-currentcolor.svg` ou `svg/icons/icon-report-light.svg` |
| Preview Open Graph | `png/social/open-graph-light.png` / `png/social/open-graph-dark.png` |

**Attention :** l'icône de signalement ne contient **pas** le check vert, qui indique une correction terminée. La version monochrome est préférable sur un widget client si la marque du site doit rester dominante.

## Dossiers

- `svg/icons/` — icône principale, état signalement, résolu, compacte, monochrome, clair/sombre.
- `svg/logos/` — lockups horizontaux, empilés et wordmarks avec contours typographiques vectoriels (pas de police à installer).
- `svg/social/` — illustrations de partage.
- `png/avatars/` — GitHub / npm.
- `png/icons/` — tailles courantes et widget.
- `png/logos/` — rendus PNG 2x.
- `png/favicons/` — variantes et tailles web.
- `web/` — favicon SVG / ICO, touch icons, manifeste, CSS tokens, HTML exemple.
- `docs/` — planches de prévisualisation.

## Remarques de production

- Le SVG est basé sur des **contours vectorisés depuis la dernière image approuvée** : le côté manuscrit est réellement repris de la référence.
- Toutes les couleurs des SVG sont **unies**, et les exports PNG sont rendus depuis ces SVG.
- La typographie est Inter Display ExtraBold pour le logo et Inter Medium pour le slogan. Les **contours des lettres** sont inclus dans les SVG ; aucun fichier de police n'est distribué.
- À **16 px**, les détails deviennent nécessairement très petits : le kit propose une version `compact` sans traits d'accent secondaires.
- Le SVG `icon-widget-currentcolor.svg` peut être coloré via la propriété CSS `color` lorsqu’il est inséré **inline** ; avec une balise `<img>`, utiliser plutôt une des variantes de couleur.
- Les assets sont pour l'identité du projet open source ; les fichiers sont fournis pour l'usage du projet.
