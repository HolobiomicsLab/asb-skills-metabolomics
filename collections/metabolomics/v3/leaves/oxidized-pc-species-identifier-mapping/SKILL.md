---
name: oxidized-pc-species-identifier-mapping
description: Use when a fixed sample of oxidized PC species retrieved from CMM 3.0
  (via its RESTful API) must be checked for presence/absence in a public reference
  like LIPID MAPS to quantify novel, non-overlapping oxidized lipids.
when_to_use_negative:
- When the task is annotating unknown features in LC-MS/MS data — use CMM 3.0's metabolite
  annotation, MS2 spectral comparison, or spectral quality-assessment services instead;
  this skill operates on enumerated lipid identifier lists, not spectra.
- When the public reference cannot be normalized to PC class / oxidation-level nomenclature,
  making identifier-level matching meaningless.
- When you need MS/MS spectra or structural properties of the oxidized lipids rather
  than library membership/overlap accounting.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: CEU Mass Mediator (CMM) 3.0
  role: Serves the in-house oxidized-lipid library; source of the sampled oxidized
    PC entries whose presence in public databases is being tested
  repo: https://github.com/albertogilf/ceuMassMediator
- name: CMM RESTful API
  role: Retrieval channel for the fixed sample of oxidized PC entries from CMM 3.0
- name: LIPID MAPS
  role: Public reference database; its oxidized phospholipid subset is the ground
    set against which each CMM entry is matched
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00720
    title: 'CEU Mass Mediator 3.0: A Metabolite Annotation Tool'
  source_content_hash: sha256:3eb10f317dd359dd98cf8b96c06b057dbec713c2b86f6de9431cdeb0eadf1fe1
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# oxidized-pc-species-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many oxidized glycerophosphocholine (PC) species from CMM 3.0's in-house oxidized-lipid library are absent from public lipid databases (e.g., LIPID MAPS) by normalized identifier matching. It turns the paper's claim that the library contains oxidized lipids 'not present in other sources' into a per-entry, countable overlap analysis.

## When to use

Use when a fixed sample of oxidized PC species retrieved from CMM 3.0 (via its RESTful API) must be checked for presence/absence in a public reference like LIPID MAPS to quantify novel, non-overlapping oxidized lipids.

## When NOT to use

- When the task is annotating unknown features in LC-MS/MS data — use CMM 3.0's metabolite annotation, MS2 spectral comparison, or spectral quality-assessment services instead; this skill operates on enumerated lipid identifier lists, not spectra.
- When the public reference cannot be normalized to PC class / oxidation-level nomenclature, making identifier-level matching meaningless.
- When you need MS/MS spectra or structural properties of the oxidized lipids rather than library membership/overlap accounting.

## Inputs

- Fixed sample of oxidized glycerophosphocholine (PC) species identifiers/abbreviated names from the CMM 3.0 in-house oxidized-lipid library, retrieved via the CMM RESTful API
- Oxidized phospholipid subset exported from LIPID MAPS (public reference entry list)

## Outputs

- Per-entry comparison table oxidized_lipid_overlap.csv with present/absent-in-public-source status for each sampled CMM oxidized PC entry
- Tally of overlapping vs non-overlapping oxidized PC entries

## How to apply

Retrieve a fixed sample of oxidized glycerophosphocholine (PC) entries served by CEU Mass Mediator (CMM) 3.0 through its RESTful API, which backs the dedicated service for annotation of oxidized glycerophosphocholines and the in-house library reported to hold oxidized lipids not present in other sources. Download the oxidized phospholipid subset from LIPID MAPS as the public reference set. Normalize both sides before matching: harmonize PC class designation and oxidation-level nomenclature in lipid identifiers/abbreviated names so entries are compared on identical nomenclature rather than raw strings. Match each sampled CMM entry against the LIPID MAPS set, recording present vs absent per entry, then tally overlapping vs non-overlapping counts. Write the per-entry comparison table to oxidized_lipid_overlap.csv; the non-overlapping tally is the quantitative evidence for (or against) the library-novelty claim. Treat 'absent' verdicts cautiously: they are only meaningful after the nomenclature normalization rules have been fixed and documented.

## Related tools

- **CEU Mass Mediator (CMM) 3.0** (Serves the in-house oxidized-lipid library; source of the sampled oxidized PC entries whose presence in public databases is being tested) — https://github.com/albertogilf/ceuMassMediator
- **CMM RESTful API** (Retrieval channel for the fixed sample of oxidized PC entries from CMM 3.0)
- **LIPID MAPS** (Public reference database; its oxidized phospholipid subset is the ground set against which each CMM entry is matched)

## Examples

```
python -c "import pandas as pd; pd.DataFrame(rows, columns=['cmm_id','lipidmaps_id','present_in_lipidmaps']).to_csv('oxidized_lipid_overlap.csv', index=False)"
```

## Evaluation signals

- oxidized_lipid_overlap.csv contains exactly one row per sampled CMM oxidized PC entry, each with a binary present/absent-in-LIPID-MAPS verdict.
- Overlapping + non-overlapping tallies sum to the total number of sampled entries (no unmatched or dropped rows).
- Spot-check matched pairs to confirm PC class and oxidation-level nomenclature normalization was applied — no entry should be called 'absent' purely due to formatting/naming variants.
- All sampled entries were actually retrieved from CMM 3.0's in-house oxidized-lipid library via the RESTful API, not drawn from other sources.
- The non-overlapping count is reproducible when the workflow is re-run on the same fixed sample of oxidized PC species.

## Limitations

- Self-deployment of CMM requires access to an internal CEU database (credentials requested by email) plus app-server data-source configuration and the javax.faces-2.2.12.jar dependency, so the public instance/API is the practical route for retrieval.
- The overlap verdict depends on the quality of lipid identifier/abbreviated-name normalization (PC class, oxidation-level nomenclature); superficial naming differences can inflate or deflate the non-overlap count.
- The 'not present in other sources' property is the authors' claim; this skill quantifies overlap only against the chosen public reference (e.g., LIPID MAPS), not against all possible sources.
- The article provides no discussion-section detail on the oxidized-lipid library's composition ('No changelog found.'), so library coverage must be verified empirically via the API.

## Evidence
