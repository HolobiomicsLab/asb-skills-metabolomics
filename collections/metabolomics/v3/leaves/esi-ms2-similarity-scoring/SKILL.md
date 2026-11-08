---
name: esi-ms2-similarity-scoring
description: Use when identifying small chemical compounds (<3000 Da) from ESI-MS2
  query spectra by spectral search against MassBank reference records, especially
  when comparing QqQ and QqTOF query/target instrument pairings or tuning precision
  vs recall.
when_to_use_negative:
- The query data are EI-MS or FAB-MS spectra rather than ESI-MS2 — the weighting exponents
  on peak intensity and m/z are optimized specifically to ESI-MS2 data
- The goal is de novo or structural elucidation rather than identification by similarity
  search against an authenticated reference library
- Compounds are outside MassBank's scope of small chemical compounds for life sciences
  (<3000 Da) or absent from the reference record set
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassBank
  role: Public reference repository of mass spectra of small chemical compounds providing
    the ESI-MS2 target records (merged and unmerged) used for spectral search
  repo: https://github.com/MassBank/MassBank3
- name: MassBank-web
  role: Deprecated MassBank software containing the spectral search functionality;
    superseded by MassBank3 for the web app
  repo: https://github.com/MassBank/MassBank-web
- name: MassBank-cli-tools
  role: Java command line tools including the validator for MassBank record files
    used in the search workflow
  repo: https://github.com/MassBank/MassBank-cli-tools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:15b396a781e3541e363ac1e977b63cef93d8d5dc5c603291fd32e60359aff218
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank3
  tools:
  - MassBank
  - MassBank-cli-tools
  - MassBank-web
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# esi-ms2-similarity-scoring

## Summary

Compound identification by spectral search against MassBank ESI-MS2 reference records using a weighted cosine correlation similarity score (weighting exponents on peak intensity and m/z optimized for ESI-MS2 data), with merged-spectrum target sets improving precision by 21-23% at score cutoff 0.6.

## When to use

Use when identifying small chemical compounds (<3000 Da) from ESI-MS2 query spectra by spectral search against MassBank reference records, especially when comparing QqQ and QqTOF query/target instrument pairings or tuning precision vs recall.

## When NOT to use

- The query data are EI-MS or FAB-MS spectra rather than ESI-MS2 — the weighting exponents on peak intensity and m/z are optimized specifically to ESI-MS2 data
- The goal is de novo or structural elucidation rather than identification by similarity search against an authenticated reference library
- Compounds are outside MassBank's scope of small chemical compounds for life sciences (<3000 Da) or absent from the reference record set

## Inputs

- MassBank ESI-MS2 spectral record sets (merged and unmerged target sets, e.g. TS=[Merged QqQ], TS=[Merged QqTOF])
- Query ESI-MS2 spectra from QqQ and QqTOF instruments (QS=[QqQ], QS=[QqTOF])
- MassBank record files in MassBank Record Format with accession IDs and peak lists

## Outputs

- Ranked spectral-search matches per query with weighted cosine correlation similarity scores
- Compound identification calls at similarity score cutoff 0.6
- Precision, recall, and F-measure per query-target pairing (e.g. [0.454, 0.307, 0.366] for TS=[Merged QqQ])
- Precision improvement comparison vs unmerged baseline (21-23%)

## How to apply

Load the ESI-MS2 spectral record sets from MassBank, using merged target sets in which data for identical compounds acquired under different collision-induced dissociation conditions are superposed into single target entries. Run the MassBank spectral search with the weighted cosine correlation similarity score, whose weighting exponents on peak intensity and m/z are optimized for ESI-MS2 data, pairing QS=[QqQ] queries against TS=[Merged QqTOF] and QS=[QqTOF] queries against TS=[Merged QqQ]. Apply a similarity score cutoff of 0.6 to call compound identifications for each query-target pairing. Compute precision, recall, and F-measure per pairing; merged targets should yield precision/recall/F of [0.454, 0.307, 0.366] for TS=[Merged QqQ] and [0.490, 0.299, 0.371] for TS=[Merged QqTOF]. Finally, compare against the corresponding unmerged-baseline searches to confirm the 23% (QqQ) and 21% (QqTOF) precision improvements at essentially unchanged recall.

## Related tools

- **MassBank** (Public reference repository of mass spectra of small chemical compounds providing the ESI-MS2 target records (merged and unmerged) used for spectral search) — https://github.com/MassBank/MassBank3
- **MassBank-web** (Deprecated MassBank software containing the spectral search functionality; superseded by MassBank3 for the web app) — https://github.com/MassBank/MassBank-web
- **MassBank-cli-tools** (Java command line tools including the validator for MassBank record files used in the search workflow) — https://github.com/MassBank/MassBank-cli-tools

## Examples

```
cd MassBank3/compose && cp env.dist .env && docker compose up -d && curl http://localhost:8081/MassBank-api/records/count
```

## Evaluation signals

- At similarity score cutoff 0.6, computed precision/recall/F-value reproduce [0.454, 0.307, 0.366] for TS=[Merged QqQ] and [0.490, 0.299, 0.371] for TS=[Merged QqTOF]
- Precision improvement over the unmerged baseline is 23% (QqQ) and 21% (QqTOF), with recall not significantly affected
- Similarity scores are computed as a weighted cosine correlation with exponents on peak intensity and m/z tuned for ESI-MS2 data (not a plain unweighted cosine)
- Merged target entries superpose ESI-MS2 spectra of identical compounds recorded at different collision energies into single records before search
- All record files validate against the MassBank Record Format with matching accession IDs and .txt file names

## Limitations

- MassBank-web is deprecated; use MassBank3 for the web app and MassBank-cli-tools for the validator/command line tools
- Precision at score 0.6 with merged targets is still only ~0.45-0.49, so a substantial fraction of identifications remain false positives
- Recall drops slightly (0.307 to 0.307 vs 0.299) with merged QqTOF targets; merging improves precision without significantly affecting recall but does not increase it
- Self-hosted MassBank3 instances require populating the database on first start (~30 minutes on a 4-CPU, 8GB Docker environment) before searches can run

## Evidence

- [abstract] Weighted cosine correlation with ESI-MS2 optimized exponents: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [abstract] Merging improves precision 21-23% at score 0.6: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [abstract] MassBank scope: small compounds for life sciences: "MassBank is the first public repository of mass spectra of small chemical compounds for life sciences (<3000 Da)."
- [intro] MassBank-web deprecated in favor of MassBank3 and cli-tools: "This repo is deprecated. Please do not use it! The new MassBank web app is maintained in repo https://github.com/MassBank/MassBank3. The validator and other command line tools are maintained in"
- [readme] MassBank record files must contain accession ID and peaks: [evidence span withheld — anchor status: not_found]
