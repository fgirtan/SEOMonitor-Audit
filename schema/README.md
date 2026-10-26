# Schema.org structured data — pagini de soluții SEOmonitor

Cod JSON-LD pentru trei pagini de soluții de pe www.seomonitor.com. Niciuna
dintre cele trei pagini nu avea structured data în momentul auditului
(`0` blocuri `application/ld+json` în sursa HTML a fiecăreia).

| Pagină | Fișier | Entitate principală |
|---|---|---|
| https://www.seomonitor.com/automation | `automation.jsonld` / `.html` | `SoftwareApplication` + `FAQPage` + `HowTo` |
| https://www.seomonitor.com/win-more-pitches | `win-more-pitches.jsonld` / `.html` | `SoftwareApplication` + `HowTo` |
| https://www.seomonitor.com/increase-retention | `increase-retention.jsonld` / `.html` | `SoftwareApplication` + `ItemList` |
| https://www.seomonitor.com/pricing | `pricing.jsonld` / `.html` | `SoftwareApplication` + `AggregateOffer` cu câte un `Offer` per plan |

Fișierele `.jsonld` conțin JSON-ul pur. Fișierele `.html` sunt aceleași date
împachetate în `<script type="application/ld+json">`, gata de lipit.

## Implementare

Inserează un singur bloc per pagină, în `<head>` (sau oriunde în `<body>` —
Google acceptă ambele). Site-ul rulează WordPress + Breakdance pe Kinsta, deci
cel mai simplu e un câmp „custom head code” per pagină sau un snippet
condiționat pe slug.

Nu duplica blocurile: fiecare pagină primește exact unul.

## Structură

Toate cele trei fișiere folosesc `@graph` cu aceleași noduri partajate,
identificate prin `@id` stabil, ca Google să le consolideze într-o singură
entitate la nivel de site:

- `https://www.seomonitor.com/#organization` — compania (fondată 2010, logo,
  profiluri sociale, `knowsAbout`)
- `https://www.seomonitor.com/#website` — site-ul
- `https://www.seomonitor.com/#software` — produsul-umbrelă SEOmonitor,
  referit prin `isPartOf` de fiecare modul

Peste ele, fiecare pagină definește:

- `WebPage` cu `about` / `mainEntity` către modulul ei, `breadcrumb`,
  `datePublished` și `dateModified` (preluate din `article:*_time` existente)
- `BreadcrumbList` cu două niveluri (Home → pagina). Nu am adăugat un nivel
  „Solutions” pentru că nu există un URL `/solutions/` real pe site, iar
  Google cere ca fiecare item din breadcrumb să fie o pagină existentă.
- `SoftwareApplication` pentru modulul specific, cu `featureList`, `audience`,
  `offers`, `aggregateRating`, `potentialAction` (RegisterAction către trial)

Toate valorile sunt luate din conținutul vizibil al paginilor. Nu am inventat
date — cerința Google e ca structured data să reflecte ce vede utilizatorul.

## Ce trebuie completat înainte de publicare

**`ratingCount` — obligatoriu.** În toate trei fișierele apare:

```json
"ratingCount": "REPLACE_WITH_G2_REVIEW_COUNT"
```

Paginile afișează „Rated 4.7/5 on G2”, dar nu și numărul de review-uri.
Google cere `ratingCount` sau `reviewCount` pentru `aggregateRating`; fără el
blocul e invalid. Ai două opțiuni:

1. Pune numărul real de review-uri din profilul G2 al SEOmonitor (și
   afișează-l și pe pagină, lângă „4.7/5”, ca marcajul să corespundă cu ce
   vede userul).
2. Șterge complet obiectul `aggregateRating`.

Placeholder-ul e lăsat ca string tocmai ca să pice zgomotos în Rich Results
Test dacă cineva publică fără să-l completeze.

## /pricing — ce trebuie completat

Pagina /pricing nu a putut fi citită direct când a fost scris blocul (domeniul
era blocat din mediul de lucru), așa că doar structura și valorile deja
confirmate sunt completate:

- `AggregateOffer` cu `lowPrice` 25, `highPrice` 299, `offerCount` 3, EUR —
  preluate din documentul „Seo Monitor Complete Schema” (blocul homepage)
- planurile **Starter** și **Pro** — numele apar pe paginile de produs
  („Included in Starter and Pro plans”); al treilea plan nu e confirmat
- `WebPage`, `BreadcrumbList`, `RegisterAction` către trial — la fel ca la
  celelalte pagini

Înlocuiește din pagina live toate valorile `REPLACE_WITH_*`:

| Placeholder | Ce pui |
|---|---|
| `REPLACE_WITH_PAGE_TITLE` / `REPLACE_WITH_META_DESCRIPTION` | `<title>` și meta description ale paginii |
| `REPLACE_WITH_STARTER_PRICE`, `..._PRO_PRICE`, `..._PLAN3_PRICE` | prețul afișat, doar cifra (`"99"`, nu `"€99"`) |
| `REPLACE_WITH_BILLING_UNIT` | unitatea afișată, ex. `per month`, `per 100 keywords / month` |
| `REPLACE_WITH_*_DESCRIPTION_FROM_PAGE` | textul scurt de sub numele planului |
| `REPLACE_WITH_THIRD_PLAN_NAME` | numele celui de-al treilea plan; dacă nu există, șterge oferta și pune `offerCount` 2 |
| `REPLACE_WITH_G2_REVIEW_COUNT` | ca mai sus |

După completare, `lowPrice` și `highPrice` trebuie să fie exact cel mai mic și
cel mai mare preț dintre oferte. Dacă prețul depinde de numărul de keywords
(calculator/slider), pune la fiecare plan prețul de pornire afișat și
`unitText` corespunzător.

Pe /pricing pune blocul acesta în locul nodului `#software` minimal folosit
pe celelalte pagini: e același `@id`, deci Google le consolidează.

## Note despre rich results

- **FAQPage** (pagina `/automation`): din august 2023 Google afișează rich
  results FAQ doar pentru site-uri guvernamentale și de sănătate. Marcajul
  rămâne valid și util pentru Bing, pentru AI Overviews și pentru parsarea de
  către LLM-uri — relevant dat fiind că SEOmonitor vinde chiar tracking de
  vizibilitate în AI search. Nu te aștepta la stele/acordeoane în Google.
- **HowTo**: rich results HowTo au fost scoase de Google în septembrie 2023.
  Le-am păstrat din același motiv semantic, nu pentru afișare în SERP.
- **SoftwareApplication**: rich result-ul de aplicație cere `name`, `offers`
  și `aggregateRating` sau `review` — toate sunt prezente (vezi mai sus
  despre `ratingCount`).
- **Review-ul lui Radu Marcusu** (paginile `/automation` și
  `/win-more-pitches`) nu are `reviewRating`, pentru că pe pagină nu apare o
  notă. Rămâne valid ca schema.org și e util ca context de entitate, dar nu
  va genera rich result de review de unul singur.

## Validare

```
https://search.google.com/test/rich-results
https://validator.schema.org/
```

Rulează ambele după publicare. `validator.schema.org` prinde erorile de
vocabular, Rich Results Test prinde ce e eligibil efectiv în Google.
