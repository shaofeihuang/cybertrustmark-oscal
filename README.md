# CSA Cyber Trust mark (v202504) Classical Cybersecurity Requirements – OSCAL Catalogs

Machine-readable [OSCAL](https://pages.nist.gov/OSCAL/) 1.1.2 catalogs for the "Classical cybersecurity" requirements of the CSA Cyber Trust mark, issued by the Cyber Security Agency of Singapore (CSA). One catalog is provided per preparedness tier, plus a combined catalog covering all tiers.

> **Unofficial transcription.** This is not published or endorsed by CSA. The official PDF is authoritative; check any clause you rely on against it.

## What's in the catalogs

| | |
|---|---|
| Source | CSA Cybersecurity Certification: Cyber Trust mark, v202504 (April 2025) |
| Scope | Annex B – Cybersecurity preparedness domains and descriptions, printed pp. 30–111 (PDF pp. 31–112): domains B.1 (Governance) to B.22 (Business continuity/Disaster recovery) |
| Column used | "Classical cybersecurity" only |
| Not included | Annex A (Cyber Essentials mark, which the document only points to externally); the Cloud security, OT security and AI security columns; clauses that only state "Domain is not assessable for this tier" (25 clauses, e.g. B.1.1, B.1.2, B.4.1–B.4.4) |
| Format | OSCAL 1.1.2 catalog, JSON |

### Files

| File | Tier | Domains | Controls |
|---|---|---|---|
| `catalog-supporter.json` | Supporter | 10 | 13 |
| `catalog-practitioner.json` | Practitioner | 13 | 25 |
| `catalog-promoter.json` | Promoter | 19 | 37 |
| `catalog-performer.json` | Performer | 21 | 56 |
| `catalog-advocate.json` | Advocate | 22 | 58 |
| `catalog-all-tiers.json` | All five tiers | 22 | 189 |

The five tier catalogs together contain exactly the controls in `catalog-all-tiers.json`. Tiers are cumulative in the certification scheme, but each tier catalog lists only the clauses introduced at that tier, so assessing a higher tier means combining the catalogs for it and the tiers below.

### Structure

```
catalog
└── group  b.1                  Domain B.1  Governance
    └── control  b.1.7          Clause B.1.7  (tier: Advocate)
        └── part  statement     Clause text (id b.1.7_smt)
```

- **One group per domain, one control per clause.** Group IDs are the lower-cased domain number (`b.1`) and control IDs the lower-cased clause number (`b.1.7`). Only domains that have at least one clause in the tier appear in a tier catalog.
- **Titles are generated.** The document gives clauses no titles, so each is built as `<Domain> – <clause> (<Tier>)`, for example `System security – B.12.12 (Advocate)`.
- **Properties on every control.**
  - `label` – the clause number, e.g. `B.12.12`.
  - `preparedness-tier` – `Supporter`, `Practitioner`, `Promoter`, `Performer` or `Advocate`.
  - `security-context` – always `Classical cybersecurity`.

  The last two use the namespace `https://www.csa.gov.sg/cyber-trust`, so they are not mistaken for OSCAL-registry names. Domain groups carry a `label` property with the domain number.
- **Flat statements.** Each clause is a single `statement` part. Lists and `NOTE`s that sit inside a clause's Classical cybersecurity cell (for example the examples under B.14.3, or the CIS note under B.12.12) are kept inline in the prose rather than split into sub-parts.
- **Tier assignment.** The document names the tier only on the first row of each tier within a domain; later rows inherit it. The same rule is applied here.
- **Metadata.** `version` is `202504`; `last-modified` is the date the catalogs were generated; a `source` link in `metadata` and a `back-matter` resource point to the CSA Cyber Trust page.

## Quick start

Python 3.8+ and the standard library only:

```python
import json

with open("catalog-all-tiers.json", encoding="utf-8") as f:
    catalog = json.load(f)["catalog"]

def controls(groups):
    for g in groups:
        yield from g.get("controls", [])
        yield from controls(g.get("groups", []))

def prop(control, name):
    return next(p["value"] for p in control["props"] if p["name"] == name)

# Print every Advocate-tier clause
for c in controls(catalog["groups"]):
    if prop(c, "preparedness-tier") == "Advocate":
        stmt = next(p for p in c["parts"] if p["name"] == "statement")
        print(prop(c, "label"), stmt["prose"])
```

To count controls per tier:

```python
from collections import Counter
print(Counter(prop(c, "preparedness-tier") for c in controls(catalog["groups"])))
```

## Working with the catalogs in other tools

These are standard OSCAL catalogs, so they should work with OSCAL tooling, for example as the source of a profile, component definition or assessment plan. For a profile that selects a given tier, import the matching tier catalog(s) and include controls by ID.

```bash
pip install compliance-trestle
trestle import -f catalog-all-tiers.json -o cybertrust2025
trestle validate -f catalog-all-tiers.json
```

The structure was checked for unique IDs and clause counts, but the files have **not** yet been run through an OSCAL schema validator. Please validate them with your preferred tool (for example `oscal-cli` or `compliance-trestle`) before depending on them.

## Notes and limitations

- The text was extracted from the PDF programmatically, using column positions to isolate the Classical cybersecurity column. Page headers that break into the middle of a clause across pages ("Cyber Trust mark") were removed, and line-break hyphenation was repaired. The curly quotes and British spelling of the original are kept.
- The one footnote in the Classical cybersecurity column (footnote 11 on B.1.3, "Cybersecurity includes cloud security, OT security and AI security if the organisation has included these areas in the scope of certification") is dropped, and its marker removed from the prose. Clause B.1.3 is therefore worded slightly differently from the printed page.
- Many Classical statements are marked "#" in the Cloud, OT and AI columns, meaning the Classical statement also applies in that context. That cross-reference is not represented here.
- Clauses were spot-checked against the PDF, but not every word was independently verified. Please report any discrepancy.

## Source and licence

The regulatory content belongs to CSA and the Government of Singapore. See the official publication on the [CSA Cyber Trust page](https://www.csa.gov.sg/cyber-trust) for the authoritative text and its terms of use.

Licence for the tooling and structure: see `LICENSE`.
