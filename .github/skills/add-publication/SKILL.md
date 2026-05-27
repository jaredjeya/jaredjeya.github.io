---
name: add-publication
description: "Use when adding a new publication to the academic website. Given a paper URL (arXiv, DOI, journal page), fetches metadata and creates a fully-drafted _publications/ markdown file with frontmatter, abstract, and an image placeholder."
argument-hint: "Paste an arXiv URL, DOI URL, or journal page URL for the paper to add."
---

# Add Publication Skill

Create a fully-drafted publication markdown file for this academic website from a URL.

## Workflow

### Step 1 — Fetch metadata from the URL

Fetch the provided URL. Extract:
- **Title** — full paper title
- **Authors** — full ordered author list
- **Abstract** — complete abstract text
- **Date** — submission/publication date (YYYY-MM-DD)
- **Venue** — journal or conference name, or `arXiv` if preprint-only
- **arXiv ID** — if present (e.g. `2603.26870`)
- **DOI** — if present
- **Journal reference** — volume, pages, year if published

For arXiv URLs, also fetch the abstract page (`https://arxiv.org/abs/<id>`) to check for a journal reference. Look for:
- **"Journal-ref:"** — gives the human-readable journal reference (volume, pages, year)
- **"Related DOI:"** — gives the journal DOI (distinct from the arXiv DOI; only use a DOI found under this heading, not any other DOI on the page)

**Author list truncation**: The arXiv abstract page truncates long author lists with "...". If the list appears cut off, fetch the HTML paper (`https://arxiv.org/html/<id>v1`) to retrieve the full author list from the title page. Use that complete list for both the frontmatter citation and the body citation string.

### Step 2 — Determine category and reference

**Category logic** (choose first that applies):

| Condition | Category |
|---|---|
| Has journal reference / DOI and is a journal article | `manuscript` |
| Has journal reference / DOI and is conference proceedings | `conference` |
| arXiv only (no journal ref) | `preprint` |
| PhD thesis | `thesis` |
| Book | `book` |

If the category is ambiguous, ask the user to confirm before proceeding.

**Reference and URL logic:**
- If published in a journal: use DOI URL (`https://doi.org/<doi>`) as `paperurl`, journal name as `venue`
- If preprint only: use arXiv abstract URL (`https://arxiv.org/abs/<id>`) as `paperurl`, `arXiv` as `venue`
- If both exist: prefer the DOI/journal for `paperurl` and `venue`, but mention arXiv in the citation

### Step 3 — Construct the citation string

Format the citation as:
```
Author One, Author Two, Author Three, "Title." Journal Volume, Pages (Year)
```
Or for preprints:
```
Author One, Author Two, "Title." arXiv:XXXX.XXXXX [subject]
```
Use `&quot;` for quotation marks around the title (HTML entity, required for YAML safety).

For arXiv citations, include the subject classification if available (e.g. `[cond-mat.str-el]`).

### Step 4 — Generate the permalink slug

Construct as: `/publication/YYYY-MM-DD-short-slug`

For the slug: lowercase the title, replace spaces with hyphens, remove special characters. Target 4–6 words; aim for slugs under ~50 characters. To shorten:
- Drop articles (a, an, the) and prepositions first
- Abbreviate well-known compound terms (e.g. "matrix-product-state" → "mps", "many-body localization" → "mbl", "quantum chromodynamics" → "qcd")
- Use fewer words if the remaining ones unambiguously identify the paper

### Step 5 — Create the file

Filename: `_publications/YYYY-MM-DD-short-slug.md` (same slug as permalink, without leading `/publication/`)

```markdown
---
title: "TITLE"
collection: publications
category: "CATEGORY"
permalink: /publication/YYYY-MM-DD-short-slug
date: YYYY-MM-DD
venue: 'VENUE'
paperurl: 'URL'
citation: 'CITATION STRING'
---
<!-- TODO: Replace placeholder with actual figure. Suggested: graphical abstract or key result figure. -->
<img src="/images/placeholder.png" alt="Graphical abstract" style="width: 600px; display: block; margin: auto;">

ABSTRACT TEXT
```

Write the abstract verbatim from the source. Do not summarise or paraphrase.

### Step 6 — Confirm and report

After creating the file:
1. Print the frontmatter for the user to review
2. Note the image placeholder — remind the user to replace `/images/placeholder.png` with the actual figure and update the `alt` text
3. Further remind the user to update `/_pages/about.md` and `/_data/cv.json`.
4. If the paper was found to be published (has a journal ref) but was added as a preprint (or vice versa), flag this for the user to verify

## Reference: existing publication example

See [`_publications/Ergodicity-breaking-in-matrix-product-state-effective-Hamiltonians.md`](../../../_publications/Ergodicity-breaking-in-matrix-product-state-effective-Hamiltonians.md) for a working example of the expected format.

## Reference: valid categories

Defined in [`_config.yml`](../../../_config.yml) under `publication_category`:

| Key | Display Title |
|---|---|
| `preprint` | Preprints |
| `manuscript` | Journal Articles |
| `conference` | Conference Papers |
| `thesis` | Phd Thesis |
| `book` | Books |

Adding a new category requires editing `_config.yml` and restarting the Jekyll server.
