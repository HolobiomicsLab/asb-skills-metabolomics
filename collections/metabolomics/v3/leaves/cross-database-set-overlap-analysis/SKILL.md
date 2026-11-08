---
name: cross-database-set-overlap-analysis
description: Use when you need to quantify how many entries of an in-house metabolite/lipid
  library overlap with a public reference database, e.g., verifying claims that CMM
  3.0's oxidized lipids are 'not present in other sources'.
when_to_use_negative:
- The goal is annotating experimental spectra from m/z or MS2 data rather than auditing
  library contents — use CMM 3.0's annotation, spectral-comparison, or spectral quality-assessment
  services instead.
- The two databases cannot be reconciled to a shared identifier/abbreviated-name nomenclature
  (no PC-class or oxidation-level normalization possible), making set membership meaningless.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: 'Source library: its in-house oxidized lipid entries (served for sampled oxidized
    glycerophosphocholines) form the set whose overlap with public sources is tested'
  repo: albertogilf/ceuMassMediator
- name: CMM RESTful API
  role: Programmatic retrieval endpoint used to fetch the sampled oxidized lipid entries
    from CMM 3.0
  repo: albertogilf/ceuMassMediator
- name: LIPID MAPS
  role: 'Public reference set: its downloaded oxidized phospholipid subset defines
    the ''present in other sources'' side of the comparison'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:29121ef39c6b263c46a9f514fa75629808549af20d3302be9adc4652b2e94aa6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/albertogilf/ceuMassMediator
  repo_url: https://github.com/albertogilf/ceuMassMediator
  tools:
  - CEU Mass Mediator (CMM) 3.0
  - CMM RESTful API
  - LIPID MAPS
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-database-set-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Set-membership comparison that quantifies how many entries of CMM 3.0's in-house oxidized lipid library overlap a public reference (LIPID MAPS), yielding per-entry presence/absence records that substantiate the claim that CMM 3.0 added oxidized lipids 'not present in other sources'.

## When to use

Use when you need to quantify how many entries of an in-house metabolite/lipid library overlap with a public reference database, e.g., verifying claims that CMM 3.0's oxidized lipids are 'not present in other sources'.

## When NOT to use

- The goal is annotating experimental spectra from m/z or MS2 data rather than auditing library contents — use CMM 3.0's annotation, spectral-comparison, or spectral quality-assessment services instead.
- The two databases cannot be reconciled to a shared identifier/abbreviated-name nomenclature (no PC-class or oxidation-level normalization possible), making set membership meaningless.

## Inputs

- Fixed sample of oxidized glycerophosphocholine (PC) species identifiers
- Oxidized lipid library entries retrieved from CMM 3.0 via its RESTful API
- Oxidized phospholipid subset downloaded from LIPID MAPS

## Outputs

- Per-entry presence/absence comparison table written to oxidized_lipid_overlap.csv
- Tallies of overlapping vs non-overlapping entries (count of CMM-only oxidized lipids)

## How to apply

Fix a sample of oxidized glycerophosphocholine (PC) species and pull each species' library entry from CMM 3.0 through its RESTful API; the rationale is the article's claim that CMM 3.0 added an in-house library of oxidized lipids reported to be absent from other sources. Download the oxidized phospholipid subset from LIPID MAPS as the public reference set. Before any membership test, normalize lipid identifiers and abbreviated names on both sides — resolving the PC class prefix and the oxidation-level nomenclature — because raw name strings differ across databases and would inflate false 'novel' calls. Label each sampled CMM entry as present or absent in LIPID MAPS and tally overlapping vs non-overlapping counts. Persist the per-entry comparison table to oxidized_lipid_overlap.csv so the database-novelty claim can be audited entry by entry.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Source library: its in-house oxidized lipid entries (served for sampled oxidized glycerophosphocholines) form the set whose overlap with public sources is tested) — albertogilf/ceuMassMediator
- **CMM RESTful API** (Programmatic retrieval endpoint used to fetch the sampled oxidized lipid entries from CMM 3.0) — albertogilf/ceuMassMediator
- **LIPID MAPS** (Public reference set: its downloaded oxidized phospholipid subset defines the 'present in other sources' side of the comparison)

## Examples

```
python -c "import pandas as pd; print(pd.read_csv('oxidized_lipid_overlap.csv')['present_in_lipid_maps'].value_counts())"
```

## Evaluation signals

- oxidized_lipid_overlap.csv exists and contains exactly one row per sampled CMM oxidized PC entry, each with a present/absent verdict.
- Overlapping + non-overlapping tallies sum to the number of sampled entries, with no unresolved or NA matches.
- Matches are made on normalized PC-class / oxidation-level identifiers, not raw name strings — spot-check a few matches by hand against both sources.
- The non-overlapping count is consistent with the article's claim that CMM 3.0's in-house oxidized lipids are 'not present in other sources'.
- Re-running the same fixed PC-species sample produces identical counts (deterministic retrieval and matching).

## Limitations

- Overlap calls depend entirely on identifier/abbreviated-name normalization (PC class, oxidation-level nomenclature); unhandled naming variants would inflate the 'novel entry' count.
- The comparison covers a fixed sample of oxidized glycerophosphocholine species against LIPID MAPS only — it cannot certify absence from all other sources or across the full library.
- Self-hosting CMM requires access to CEU's internal database (requested by email), so fully local reproduction of library retrieval is gated on external credentials.

## Evidence
