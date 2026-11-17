# CSA Cyber Trust Mark (2025) Requirements – OSCAL Catalogs

Machine-readable [OSCAL](https://pages.nist.gov/OSCAL/) 1.1.2 catalogs for the cybersecurity requirements of the CSA Cyber Trust Mark, issued by the Cyber Security Agency of Singapore (CSA). The requirements are split by security context (Classical cybersecurity, Cloud security, OT security and AI security). Each context has one catalog per preparedness tier plus a combined catalog covering all tiers.

> **Unofficial transcription.** This is not published or endorsed by CSA. The official PDF is authoritative; check any clause you rely on against it.

**Acknowledgment** The README of this repository is adapted from Baden Hughes’ [sg-csa-ccop-cii-oscal](https://github.com/badenh/sg-csa-ccop-cii-oscal), which provides an OSCAL catalog of the Cybersecurity Code of Practice for Critical Information Infrastructure. Thanks to Baden for publishing that work openly and for the approach it established.

## What's in the catalogs

| | |
|---|---|
| Source | CSA Cybersecurity Certification: Cyber Trust Mark, v202504 (April 2025) |
| Scope | Annex B – Cybersecurity preparedness domains and descriptions, printed pp. 30–111 (PDF pp. 31–112): domains B.1 (Governance) to B.22 (Business continuity/Disaster recovery) |
| Columns used | "Classical cybersecurity", "Cloud security", "OT security" and "AI security", each in its own folder |
| Not included | Annex A (Cyber Essentials mark, which the document only points to externally); clauses that only state "Domain is not assessable for this tier" (25 clauses, e.g. B.1.1, B.1.2, B.4.1–B.4.4); clauses marked "#" in the cloud, OT or AI column (see below) |
| Format | OSCAL 1.1.2 catalog, JSON |

## Repository layout

```
.
├── classical-cybersecurity/   Classical cybersecurity catalogs
├── cloud-security/            Cloud security catalogs
├── ot-cybersecurity/          OT security catalogs
├── ai-cybersecurity/          AI security catalogs
├── examples/                  Example code
├── LICENSE
├── NOTICE
└── README.md
```

Each of the four catalog folders holds one catalog per preparedness tier plus an `all-tiers` catalog. In `classical-cybersecurity/` the files are `catalog-<tier>.json` and `catalog-all-tiers.json`. The other folders put the security context in the name, for example `catalog-cloud-advocate.json` and `catalog-ot-all-tiers.json`.

### Catalogs and control counts

| Folder | Supporter | Practitioner | Promoter | Performer | Advocate | All tiers |
|---|---|---|---|---|---|---|
| `classical-cybersecurity/` | 13 | 25 | 37 | 56 | 58 | 189 |
| `cloud-security/` | 2 | 5 | 14 | 17 | 11 | 49 |
| `ot-cybersecurity/` | 4 | 5 | 16 | 21 | 13 | 59 |
| `ai-cybersecurity/` | 2 | 0 (placeholder) | 14 | 11 | 10 | 37 |

The five tier catalogs in a folder together contain exactly the controls in that folder's `all-tiers` catalog. Each tier catalog lists only the clauses introduced at that tier, so assessing a higher tier means combining the catalogs for it and the tiers below, if the certification scheme applies tiers cumulatively (check this against the document).

**How the columns relate.** In Annex B, a "#" in the cloud, OT or AI column means the Classical cybersecurity statement also applies in that context. The cloud, OT and AI catalogs include only clauses that have their own statement in that column. For the full requirements of an environment, use the Classical catalog together with the catalog for that environment; they share clause numbers.

**AI Practitioner.** Annex B has no AI-specific statements at the Practitioner tier: every Practitioner row is "#" in the AI column, except B.10.3, whose AI cell is blank. The AI Practitioner catalog therefore holds a single placeholder control (`ai-practitioner-none`, class `note`) that explains this. It is not source text and is not part of the AI all-tiers catalog.

### Structure

```
catalog
└── group  b.1                  Domain B.1  Governance
    └── control  b.1.7          Clause B.1.7  (tier: Advocate)
        └── part  statement     Clause text (id b.1.7_smt)
```

- **One group per domain, one control per clause.** Group IDs are the lower-cased domain number (`b.1`) and control IDs the lower-cased clause number (`b.1.7`). Only domains that have at least one clause in the catalog appear. Control IDs repeat across folders, so `b.12.11` can exist in both the Classical and the Cloud catalog. Do not merge catalogs from different folders into one document without prefixing the IDs.
- **Titles are generated.** The document gives clauses no titles, so each is built as `<Domain> – <clause> (<Tier>)`, for example `System security – B.12.12 (Advocate)`.
- **Properties on every control.**
  - `label` – the clause number, e.g. `B.12.12`.
  - `preparedness-tier` – `Supporter`, `Practitioner`, `Promoter`, `Performer` or `Advocate`.
  - `security-context` – `Classical cybersecurity`, `Cloud security`, `OT security` or `AI security`.

  The last two use the namespace `https://www.csa.gov.sg/cyber-trust`, so they are not mistaken for OSCAL-registry names. Domain groups carry a `label` property with the domain number.
- **Flat statements.** Each clause is a single `statement` part. Lists and `NOTE`s inside a cell (for example the examples under B.14.3, or the CIS note under B.12.12) are kept inline in the prose rather than split into sub-parts.
- **Tier assignment.** The document names the tier only on the first row of each tier within a domain; later rows inherit it. The same rule is applied here.
- **Metadata.** `version` is `202504`; `last-modified` is the date the catalogs were generated; `remarks` states the column and tier covered; a `source` link in `metadata` and a `back-matter` resource point to the CSA Cyber Trust page.

## Quick start

Python 3.8+ and the standard library only:

```python
import json

with open("classical-cybersecurity/catalog-all-tiers.json", encoding="utf-8") as f:
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

To see the Classical statement next to the cloud-specific one for each clause:

```python
def load(path):
    with open(path, encoding="utf-8") as f:
        return {prop(c, "label"): c for c in controls(json.load(f)["catalog"]["groups"])}

classical = load("classical-cybersecurity/catalog-all-tiers.json")
cloud = load("cloud-security/catalog-cloud-all-tiers.json")

for label, c in cloud.items():
    print(label)
    print("  classical:", classical[label]["parts"][0]["prose"] if label in classical else "(none)")
    print("  cloud:    ", c["parts"][0]["prose"])
```

A cloud, OT or AI clause can be missing from the Classical catalog when Annex B marks the Classical cell "not assessable for this tier", which is why the lookup above checks `label in classical`.

## Working with the catalogs in other tools

These are standard OSCAL catalogs, so they should work with OSCAL tooling, for example as the source of a profile, component definition or assessment plan. For a profile that selects a given tier, import the matching tier catalog(s) and include controls by ID.

```bash
pip install compliance-trestle
trestle import -f classical-cybersecurity/catalog-all-tiers.json -o cybertrust2025
trestle validate -f classical-cybersecurity/catalog-all-tiers.json
```

The structure was checked for unique IDs and clause counts, but the files have **not** yet been run through an OSCAL schema validator. Please validate them with your preferred tool (for example `oscal-cli` or `compliance-trestle`) before depending on them.

## Notes and limitations

- The text was extracted from the PDF programmatically, using column positions to isolate each security-context column. Page headers that break into the middle of a clause across pages ("Cyber Trust Mark") and the printed page numbers that fall inside the cloud column were removed, and line-break hyphenation was repaired. The curly quotes and British spelling of the original are kept.
- The one footnote in the Classical cybersecurity column (footnote 11 on B.1.3, "Cybersecurity includes cloud security, OT security and AI security if the organisation has included these areas in the scope of certification") is dropped, and its marker removed from the prose. Clause B.1.3 is therefore worded slightly differently from the printed page.
- The "#" cross-reference is not represented as a control in the cloud, OT and AI catalogs; it is expressed by those clauses being absent from them.
- In B.10.3 the AI security cell is blank in the source, so the clause is not in any AI catalog.
- Clauses were spot-checked against the PDF, but not every word was independently verified. Please report any discrepancy.

## Source and licence

The regulatory content belongs to CSA and the Government of Singapore. See the official publication on the [CSA Cyber Trust page](https://www.csa.gov.sg/cyber-trust) for the authoritative text and its terms of use.

Licence for the tooling and structure: see `LICENSE` and `NOTICE`.
