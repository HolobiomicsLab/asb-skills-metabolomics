---
name: reference-table-regeneration-verification
description: Use when a paper publishes a scored/ranked table (e.g. Table 3 of MS2DECIDE)
  whose inputs (per-compound annotation indicators and the scoring function) are deposited
  in a public repo, and the goal is to reproduce and verify it.
when_to_use_negative:
- Input is raw MS/MS data (quantitative .csv + .mgf files) rather than precomputed
  annotation indicators — use the MS2DECIDE K_estimation pipeline instead to generate
  k values from scratch
- The goal is de novo annotation or prioritization of new spectra rather than reproducing/verifying
  an already-published ranking table
- The deposited repository does not contain the reconstructed indicators or scoring
  function needed to recompute the published scores
tools:
- name: MS2DECIDE
  role: Python library whose decision-theory k function aggregates GNPS, ISDB-LOTUS,
    and Sirius annotations to produce the k scores that are sorted to regenerate the
    reference ranking table
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: One of the three annotation tools whose outputs (via FBMN, optionally iterative
    weighted analog search) feed the k-score computation underlying the table
- name: Sirius
  role: Annotation tool (Sirius 6, batch mode, Confidence Approximate score) whose
    structure_identifications.tsv outputs feed the k-score computation
- name: ISDB-LOTUS
  role: Annotation tool (strict 0.02 Da library search via spectral_lib_matcher/MatchMS)
    whose outputs feed the k-score computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:b503b7d0f5293b36a48c63f67a3c0d39e1967c964de09d7cb7041dd4ac1c6432
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Reference table regeneration and verification

## Summary

A reproducibility skill for regenerating a published ranking table (here, the MS2DECIDE k-score ranking of the 95-compound chemodiverse NP library, Table 3/Table S7) from deposited data, then validating it by row count, rank-order checks, and spot-checks of known entries against the reference.

## When to use

Use when a paper publishes a scored/ranked table (e.g. Table 3 of MS2DECIDE) whose inputs (per-compound annotation indicators and the scoring function) are deposited in a public repo, and the goal is to reproduce and verify it.

## When NOT to use

- Input is raw MS/MS data (quantitative .csv + .mgf files) rather than precomputed annotation indicators — use the MS2DECIDE K_estimation pipeline instead to generate k values from scratch
- The goal is de novo annotation or prioritization of new spectra rather than reproducing/verifying an already-published ranking table
- The deposited repository does not contain the reconstructed indicators or scoring function needed to recompute the published scores

## Inputs

- Deposited article data from the MejriY/MS2DECIDE_article_data GitHub repository (reconstructed annotation-tool indicators and decision-theory k function)
- Reconstructed GNPS, ISDB-LOTUS, and Sirius annotation indicators for the 95-compound chemodiverse spectral library
- Reference ranking table (Table 3 / Table S7) from the article

## Outputs

- table3_ranking.csv containing NP identifier, k value, and rank columns for all 95 NPs sorted by increasing k
- Validation report confirming row count = 95, unreported NPs (ids 90-95) within top 10 ranks, and spot-checked entries matching the reference

## How to apply

Load the deposited article data from GitHub repo MejriY/MS2DECIDE_article_data, which contains the reconstructed annotation-tool indicators (GNPS, ISDB-LOTUS, Sirius) and the decision-theory k function for the 95 library natural products. Compute the knownness score k for each of the 95 NPs, sort in order of increasing k, and assign ranks 1 through 95; note that the reconstructed indicators embed the paper's defaults (T_gs = T_gi = 0.7 when a GNPS/Sirius match is absent, Sirius matching score = 0.5 when Sirius fails). Assemble the regenerated table with NP identifier, k value, and rank columns and save it as table3_ranking.csv. Validate against the reference by checking the row count equals 95, confirming all six previously unreported NPs (ids 90-95) fall within the top 10 ranks, and spot-checking specific entries (e.g. Swietenin with k=-4.5 at rank 3 and Caffeic acid with k=115.5 at rank 94).

## Related tools

- **MS2DECIDE** (Python library whose decision-theory k function aggregates GNPS, ISDB-LOTUS, and Sirius annotations to produce the k scores that are sorted to regenerate the reference ranking table) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (One of the three annotation tools whose outputs (via FBMN, optionally iterative weighted analog search) feed the k-score computation underlying the table)
- **Sirius** (Annotation tool (Sirius 6, batch mode, Confidence Approximate score) whose structure_identifications.tsv outputs feed the k-score computation)
- **ISDB-LOTUS** (Annotation tool (strict 0.02 Da library search via spectral_lib_matcher/MatchMS) whose outputs feed the k-score computation)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Row count of the regenerated table equals 95 (one row per chemodiverse library NP)
- All six previously unreported NPs (ids 90-95) appear within the top 10 positions of the increasing-k ranking
- Spot-check entry: Swietenin has k = -4.5 at rank 3
- Spot-check entry: Caffeic acid has k = 115.5 at rank 94
- Ranks 1 through 95 are uniquely assigned after sorting by increasing k with no ties-induced rank duplication

## Limitations

- When there is no match with GNPS or Sirius, the tanimoto between GNPS and Sirius cannot be calculated, so default values of 0.7 were assigned to T_gs and T_gi — regeneration must replicate these defaults
- Sirius sometimes fails to propose an annotation; a value of 0.5 was associated to the Sirius matching score in those cases
- For ISDB-LOTUS, a zero answer under the strict 0.02 Da library search was considered important information regarding the paper's definition of novelty, not a missing value
- GNPS workflows use a library score threshold of 0.001 which can cause FBMN failures; the README recommends increasing it (e.g. to 0.1), which may affect reproduction of raw annotations

## Evidence

- [abstract] tandem mass spectrum to decision (MS2DECIDE) is presented which leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools: "tandem mass spectrum to decision (MS2DECIDE) is presented which leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools"
- [readme] Annotations from GNPS, Sirius, and ISDB-LOTUS are compiled into a unified dataframe.: [evidence span withheld — anchor status: external_doc]
- [readme] The dataframe is filtered and sorted by \( K \)-values.: [evidence span withheld — anchor status: external_doc]
- [readme] In scenarios where there is no match with GNPS or no match with Sirius, the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7: [evidence span withheld — anchor status: external_doc]
- [readme] Unfortunately, in some cases, Sirius was not able to propose an annotation. To remedy, we associated a value of 0.5 to Sirius matching score.: [evidence span withheld — anchor status: external_doc]
