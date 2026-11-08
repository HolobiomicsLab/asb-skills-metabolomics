---
name: weighted-cosine-correlation-search
description: Use when querying tandem mass spectra (e.g., ESI-MS2 from QqQ or QqTOF
  instruments) against a spectral reference library and you need to quantify identification
  performance (precision/recall/F) at a fixed similarity-score cutoff.
when_to_use_negative:
- The input is EI-MS or other non-ESI-MS2 data — the weighting exponents are optimized
  specifically to ESI-MS2 data
- The goal is to evaluate merged (collision-energy-merged) reference spectra — this
  skill reproduces the unmerged baseline only
- Only pairwise spectrum similarity is needed without any identification-performance
  evaluation (no precision/recall/F computation required)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassBank-cli-tools
  role: MassBank command line tools (validator and related CLI utilities) referenced
    for working with MassBank records and spectral search workflows
  repo: https://github.com/MassBank/MassBank-cli-tools
- name: MassBank3
  role: Next-generation MassBank spectral reference library providing the REST API
    and web app from which reference spectra can be retrieved
  repo: https://github.com/MassBank/MassBank3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/jms.1777
    title: 'MassBank: a public repository for sharing mass spectral data for life
      sciences'
  source_content_hash: sha256:4159f48cab3338066d37ef2bc3b50ccd8ae79a74f29da4ef82af172653885bdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank-cli-tools
  tools:
  - MassBank-cli-tools
  - MassBank3
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# weighted-cosine-correlation-search

## Summary

Cross-search ESI-MS2 query spectra against target reference spectra using the MassBank weighted cosine correlation similarity score (with intensity and m/z weighting exponents optimized for ESI-MS2), then evaluate precision, recall, and F-value at a similarity-score threshold of 0.6.

## When to use

Use when querying tandem mass spectra (e.g., ESI-MS2 from QqQ or QqTOF instruments) against a spectral reference library and you need to quantify identification performance (precision/recall/F) at a fixed similarity-score cutoff.

## When NOT to use

- The input is EI-MS or other non-ESI-MS2 data — the weighting exponents are optimized specifically to ESI-MS2 data
- The goal is to evaluate merged (collision-energy-merged) reference spectra — this skill reproduces the unmerged baseline only
- Only pairwise spectrum similarity is needed without any identification-performance evaluation (no precision/recall/F computation required)

## Inputs

- Query ESI-MS2 peak lists (e.g., 4205 spectra from the QqQ subset)
- Target ESI-MS2 peak lists (e.g., 4431 spectra from the QqTOF subset)
- Compound identity mapping for the 856 compounds common to both instrument subsets
- Similarity-score threshold (0.6) and MassBank weighted cosine correlation parameters (intensity and m/z weighting exponents)

## Outputs

- similarity_search_metrics.csv with precision, recall, and F-value for each search direction (TS=[QqQ] and TS=[QqTOF])
- Per-query top-hit similarity scores and true/false match labels

## How to apply

Load the query and target peak lists (e.g., 856 compounds common to the QqQ and QqTOF subsets, yielding 4205 spectra for QS=[QqQ]/TS=[QqTOF] and 4431 for the reverse direction) with pandas. Score every query-target spectrum pair using the weighted cosine correlation function whose weighting exponents on peak intensity and m/z are those optimized for ESI-MS2 data in MassBank's spectral search. For each search direction, label a query a true match when the top-scoring target is the same compound and its similarity score is at least 0.6, otherwise a false match; compute precision, recall, and F-value from the true/false counts. Write the six metrics (two directions x three metrics) to a CSV and compare them against reference values such as TS=[QqQ]: precision 0.222, recall 0.327, F 0.264. Note that unmerged spectra perform poorly; merging ESI-MS2 records taken under different collision-induced dissociation conditions improved precision by 21-23% at similarity score 0.6, so this workflow can also serve as the baseline against which merged-reference searches are judged.

## Related tools

- **MassBank-cli-tools** (MassBank command line tools (validator and related CLI utilities) referenced for working with MassBank records and spectral search workflows) — https://github.com/MassBank/MassBank-cli-tools
- **MassBank3** (Next-generation MassBank spectral reference library providing the REST API and web app from which reference spectra can be retrieved) — https://github.com/MassBank/MassBank3

## Examples

```
python spectral_search_eval.py --query QS_QqQ --target TS_QqTOF --threshold 0.6 --output similarity_search_metrics.csv
```

## Evaluation signals

- TS=[QqQ] direction reproduces precision 0.222, recall 0.327, F-value 0.264 at similarity score 0.6
- TS=[QqTOF] direction reproduces precision 0.276, recall 0.292, F-value 0.284 at similarity score 0.6
- similarity_search_metrics.csv contains exactly six metric values (2 directions x 3 metrics) in the expected ranges (all between 0.2 and 0.33)
- Query/target spectrum counts match 4205 (QS=[QqQ]/TS=[QqTOF]) and 4431 (QS=[QqTOF]/TS=[QqQ]) for the 856 common compounds
- Every reported true match must have a top-hit similarity score >= 0.6 and an identical compound ID to the query

## Limitations

- Unmerged ESI-MS2 data perform poorly as reference spectra: precision, recall, and F-values at threshold 0.6 are roughly 0.22-0.33
- Results are specific to the similarity-score threshold of 0.6 and to cross-instrument searches (QqQ vs QqTOF); other thresholds or instruments are not covered
- Merging ESI-MS2 data of identical compounds under different collision-induced dissociation conditions is needed to gain the reported 21-23% precision improvement at similarity score 0.6

## Evidence

- [abstract] Weighted cosine correlation score definition: "the similarity score is calculated by a weighted cosine correlation in which weighting exponents on peak intensity and the mass-to-charge ratio are optimized to the ESI-MS(2) data"
- [abstract] Merging improves precision: "Data merging has significantly improved the precision of the identification of a chemical compound by 21-23% at a similarity score of 0.6."
- [intro] CLI tools repository: "The validator and other command line tools are maintained in https://github.com/MassBank/MassBank-cli-tools"
- [abstract] Database scope: "MassBank is the first public repository of mass spectra of small chemical compounds for life sciences (<3000 Da)."
