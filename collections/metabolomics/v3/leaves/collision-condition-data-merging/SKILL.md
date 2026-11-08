---
name: collision-condition-data-merging
description: Use when building or querying an ESI-MS2 reference library for spectral
  search where multiple CID collision energies exist per compound, and identification
  precision at a fixed score cutoff (e.g. 0.6) must be improved without losing recall.
when_to_use_negative:
- When only a single collision-energy spectrum exists per compound — there is nothing
  to merge and the unmerged baseline already applies
- When the analysis targets EI-MS or other non-ESI-MS2 data — the similarity-score
  weighting exponents and the 21-23% precision gain are specific to ESI-MS2 data
- When recall, not precision, is the binding constraint — merged targets slightly
  lowered recall in one pairing (0.307 vs. 0.299) and merging primarily trades for
  precision
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassBank3
  role: Next-generation MassBank web app and REST API hosting the spectral reference
    library from which merged ESI-MS2 target records are served and searched
  repo: https://github.com/MassBank/MassBank3
- name: MassBank-cli-tools
  role: Command-line tools (including the validator) used to check MassBank record
    files conform to the MassBank record format before import/merging
  repo: https://github.com/MassBank/MassBank-cli-tools
- name: MassBank-web
  role: Deprecated MassBank software repository; contains the legacy MassBank software
    and the MassBank record format documentation
  repo: https://github.com/MassBank/MassBank-web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:38af5c1af275866c5c38354d2065dda551f304e3783021b61fdfbc0e4255628c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank-cli-tools
  - MassBank-web
  - MassBank3
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# collision-condition-data-merging

## Summary

Merges ESI-MS2 spectra of identical compounds recorded at different collision-induced dissociation energies into single MassBank target records, improving spectral-search identification precision by 21-23% at a similarity score cutoff of 0.6 without reducing recall.

## When to use

Use when building or querying an ESI-MS2 reference library for spectral search where multiple CID collision energies exist per compound, and identification precision at a fixed score cutoff (e.g. 0.6) must be improved without losing recall.

## When NOT to use

- When only a single collision-energy spectrum exists per compound — there is nothing to merge and the unmerged baseline already applies
- When the analysis targets EI-MS or other non-ESI-MS2 data — the similarity-score weighting exponents and the 21-23% precision gain are specific to ESI-MS2 data
- When recall, not precision, is the binding constraint — merged targets slightly lowered recall in one pairing (0.307 vs. 0.299) and merging primarily trades for precision

## Inputs

- MassBank ESI-MS2 spectral record sets (MassBank record file format, e.g. MSBNK-*.txt with accession IDs and peak lists) containing multiple collision-energy entries per compound
- Query spectra sets acquired on QqQ or QqTOF instruments

## Outputs

- Merged (superposed) ESI-MS2 target record sets per compound
- Compound identification calls per query at similarity score cutoff 0.6
- Precision, recall, and F-measure values for each query-target pairing (expected: TS=[Merged QqQ] = [0.454, 0.307, 0.366]; TS=[Merged QqTOF] = [0.490, 0.299, 0.371])

## How to apply

Load ESI-MS2 records from MassBank for identical compounds acquired under different collision-induced dissociation conditions and merge them into single target entries (superposed spectra) per compound. Run the MassBank spectral search using the weighted cosine correlation similarity score, with weighting exponents on peak intensity and m/z optimized for ESI-MS2 data, testing cross-instrument pairings such as QS=[QqQ] queries against TS=[Merged QqTOF] and vice versa. Apply a similarity score cutoff of 0.6 to call compound identifications, then compute precision, recall, and F-measure. Benchmark against unmerged-baseline searches: merged targets raised precision from baseline to 0.454/0.366 (TS=[Merged QqQ]) and 0.490/0.371 (TS=[Merged QqTOF]), i.e. 21-23% precision gain at unchanged recall (~0.30).

## Related tools

- **MassBank3** (Next-generation MassBank web app and REST API hosting the spectral reference library from which merged ESI-MS2 target records are served and searched) — https://github.com/MassBank/MassBank3
- **MassBank-cli-tools** (Command-line tools (including the validator) used to check MassBank record files conform to the MassBank record format before import/merging) — https://github.com/MassBank/MassBank-cli-tools
- **MassBank-web** (Deprecated MassBank software repository; contains the legacy MassBank software and the MassBank record format documentation) — https://github.com/MassBank/MassBank-web

## Examples

```
git clone https://github.com/MassBank/MassBank3.git && cd MassBank3/compose && cp env.dist .env && docker compose up -d
```

## Evaluation signals

- Computed precision/recall/F at score cutoff 0.6 reproduce [0.454, 0.307, 0.366] for TS=[Merged QqQ] and [0.490, 0.299, 0.371] for TS=[Merged QqTOF]
- Precision improvement over the corresponding unmerged-baseline searches falls in the 21-23% range (23% for QqQ, 21% for QqTOF)
- Recall remains essentially unchanged relative to the unmerged baseline (~0.30 in both pairings)
- Every merged record file passes the MassBank format validator and carries its own accession ID with peaks listed per the MassBank Record Format

## Limitations

- The 21-23% precision improvement was measured at a similarity score of 0.6; gains at other cutoffs are not established by this evidence
- Merging is specific to ESI-MS2 data of identical compounds under different collision-induced dissociation conditions; the weighted-cosine exponents are optimized for ESI-MS2 and may not transfer to other ionization modes
- The MassBank-web repository is deprecated — use MassBank3 for the web app and MassBank-cli-tools for the validator and command line tools

## Evidence

- [abstract] Merging ESI-MS2 data of identical compounds under different collision-induced dissociation conditions improved the precision of compound identification by 21-23% at a similarity score of 0.6.: [evidence span withheld — anchor status: not_found]
- [abstract] Weighted cosine correlation scoring optimized for ESI-MS2: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [readme] MassBank record format requirements for merged/custom records: [evidence span withheld — anchor status: not_found]
- [readme] MassBank-web deprecation notice: "This repo is deprecated. Please do not use it! The new MassBank web app is maintained in repo https://github.com/MassBank/MassBank3."
