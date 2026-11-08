---
name: scoring-term-table-export
description: Use when a candidate list retrieved from PubChem/ChemSpider must be ranked
  with reference and patent information, i.e. when S_C_PCR and S_C_CCR scoring terms
  must be computed, max-normalized, and exported as a TSV for MetFrag scoring.
when_to_use_negative:
- The candidate list comes from a user-supplied SDF or a local database with no patent/PubMed
  or ChemSpider reference counts retrievable via web services — the terms cannot be
  computed.
- The analysis goal is purely fragment-based ranking from HR-MS/MS peak matching (MetFrag
  in silico fragmentation alone) with no reference/retention information included.
- Only a single candidate is under evaluation — max-normalization over the candidate
  list is undefined/trivial and the terms lose their comparative meaning.
tools:
- name: MetFrag2.2
  role: Computes and combines the normalized reference scoring terms S_C_PCR and S_C_CCR
    into the final candidate score and exports the results table; usable via command
    line or the metfRag R package
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: PubChem
  role: Source of the per-candidate patent counts (PNP) and PubMed reference counts
    (PPC) retrieved via PubChem web services for the PCR term
- name: ChemSpider
  role: Source of the five per-candidate reference counts retrieved via ChemSpider
    web services for the CCR term; access requires a ChemSpider token
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:ee9ac39f76d21446635bc5c0daf5b961bb54c4a3855fdcce2f4f0b281c7c25ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - ChemSpider
  - MetFrag2.2
  - PubChem
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scoring-term-table-export

## Summary

Computes MetFrag2.2's combined reference scoring terms for each candidate structure — the PubChem patent/PubMed score S_C_PCR and the ChemSpider five-count score S_C_CCR — normalizes both across the candidate list, and writes per-candidate scores plus contributing counts to a tab-separated results table.

## When to use

Use when a candidate list retrieved from PubChem/ChemSpider must be ranked with reference and patent information, i.e. when S_C_PCR and S_C_CCR scoring terms must be computed, max-normalized, and exported as a TSV for MetFrag scoring.

## When NOT to use

- The candidate list comes from a user-supplied SDF or a local database with no patent/PubMed or ChemSpider reference counts retrievable via web services — the terms cannot be computed.
- The analysis goal is purely fragment-based ranking from HR-MS/MS peak matching (MetFrag in silico fragmentation alone) with no reference/retention information included.
- Only a single candidate is under evaluation — max-normalization over the candidate list is undefined/trivial and the terms lose their comparative meaning.

## Inputs

- Candidate list L (e.g. structures retrieved from PubChem and ChemSpider in June 2015, or a user-supplied SDF) with per-candidate identifiers
- Per-candidate PubChem patent counts (PNP_C) and PubMed reference counts (PPC_C) from PubChem web services
- Per-candidate ChemSpider counts for the five reference/patent/data-source terms from ChemSpider web services (ChemSpider token required)
- Binary PubChem term weights a1, a2 ∈ {0,1} and MetFrag scoring-term weights (ωFrag, ωRT, ωRefs)

## Outputs

- Per-candidate normalized PubChem combined reference score S_C_PCR
- Per-candidate normalized ChemSpider combined reference score S_C_CCR
- Tab-separated results table containing S_C_PCR, S_C_CCR and the contributing patent/reference counts for every candidate

## How to apply

For each candidate C in the list L, retrieve the PubChem patent count (PNP_C) and PubMed reference count (PPC_C) via the PubChem web services, and build the cumulative reference term N_C_PCR = a1·PNP_C + a2·PPC_C with the binary weights a1, a2 ∈ {0,1}. Normalize over the candidate list as S_C_PCR = N_C_PCR / max over all candidates of N_PCR so the term is comparable within the candidate set. Similarly load the five ChemSpider reference counts per candidate retrieved via the ChemSpider web services (requires a ChemSpider token, e.g. set as ChemSpiderToken in settings.properties), combine them into S_C_CCR, and normalize S_C_CCR across the same candidate list. The reference score is then one of the terms weighted into the final MetFrag score, where the weights were optimized by randomly drawing 1000 combinations such that ωFrag+ωRT+ωRefs=1. Write the per-candidate S_C_PCR and S_C_CCR values together with the contributing PNP, PPC and ChemSpider counts to a tab-separated results table for downstream ranking.

## Related tools

- **MetFrag2.2** (Computes and combines the normalized reference scoring terms S_C_PCR and S_C_CCR into the final candidate score and exports the results table; usable via command line or the metfRag R package) — https://github.com/ipb-halle/MetFragRelaunched
- **PubChem** (Source of the per-candidate patent counts (PNP) and PubMed reference counts (PPC) retrieved via PubChem web services for the PCR term)
- **ChemSpider** (Source of the five per-candidate reference counts retrieved via ChemSpider web services for the CCR term; access requires a ChemSpider token)

## Examples

```
library(devtools); install_github("ipb-halle/MetFragR/metfRag"); library(metfRag)
```

## Evaluation signals

- Both S_C_PCR and S_C_CCR are max-normalized, so every value lies in [0,1] and at least one candidate (the one with the highest raw combined count) equals 1.0 for each term.
- Recomputing N_C_PCR = a1·PNP_C + a2·PPC_C from the exported counts reproduces the exported normalized scores exactly.
- The output is a tab-separated table whose rows cover the full candidate list and whose columns include the contributing PNP, PPC and ChemSpider counts alongside S_C_PCR and S_C_CCR.
- The scoring-term weights used to combine the reference term with fragment and retention terms satisfy ωFrag + ωRT + ωRefs = 1 (drawn from the 1000-combination optimization).
- Adding reference and retention information should measurably improve ranking relative to fragment-only scoring (e.g. MetFrag2.2 reached 420 and 336 Top 1 ranks with ChemSpider and PubChem, 89% and 71%).

## Limitations

- Reference, data source and patent counts are retrieved live from PubChem and ChemSpider web services, so scores are time-dependent (the paper's candidates were retrieved in June 2015 and PubChem has since doubled in size, making ranking more challenging).
- A ChemSpider token must be configured for the ChemSpider queries; without it the five CCR counts cannot be retrieved.
- If a candidate has no retrievable patent/PubMed/ChemSpider references, its combined reference term is zero and it is disadvantaged relative to well-documented candidates regardless of spectral fit.

## Evidence

- [abstract] retrieval of reference, data source and patent information via ChemSpider and PubChem web services: "retrieval of reference, data source and patent information via ChemSpider and PubChem web services"
- [methods] Write the per-candidate S_C_PCR and S_C_CCR values, together with the contributing counts, to a tab-separated results table.: [evidence span withheld — anchor status: not_found]
- [results] the combination of each scoring term was assessed by randomly drawing 1000 different weight combinations such that (ωFrag+ωRT+ωRefs=1): [evidence span withheld — anchor status: not_found]
- [readme] ## define chemspider token to query ChemSpider database: "## define chemspider token to query ChemSpider database"
