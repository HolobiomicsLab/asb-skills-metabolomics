---
name: tandem-ms-peak-matching
description: Use when feature-level MS/MS spectra (e.g. an MGF from MZmine FBMN export)
  fail to yield GNPS library annotations under strict parameters, and you need per-feature
  GNPS scores (c_g, t_g,i, t_g,s) for downstream aggregation as in MS2DECIDE.
when_to_use_negative:
- A strict GNPS library search already returns matches for most features — the single
  strict job (0.02 Da, threshold 0.001) is faster and the iterative workflow is unnecessary
- The goal is ISDB-LOTUS or Sirius annotation — those tools use their own matching
  procedures (ISDB-LOTUS strict 0.02 Da search; Sirius 6 batch mode with Confidence
  Approximate score), not the GNPS analog search
- You need real-time annotation — the iterative weighted analog search can take up
  to three hours and launches 27 FBMN jobs
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MS2DECIDE
  role: Host library implementing K_estimation, which invokes the iterative weighted
    GNPS analog search and aggregates the resulting annotations
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: External annotation service queried via FBMN; strict search uses 0.02 Da tolerance,
    iterative search launches 27 relaxed jobs to maximize analog matching
  repo: https://doi.org/10.1038/nbt.3597
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:fd574d80387e67152853c43199d69215b4ee9a5e9ec65b289a5c70781a56ded9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - MS2DECIDE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tandem-ms-peak-matching

## Summary

Reconstructs the GNPS I iterative weighted analog search used by MS2DECIDE: instead of a single strict GNPS library match, spectra are queried against GNPS with progressively relaxed shared-peak and mass-difference constraints, with match scores discounted by the relaxation degree reached. This increases the availability of GNPS annotation indicators (cosine score and Tanimoto values) for feature prioritization.

## When to use

Use when feature-level MS/MS spectra (e.g. an MGF from MZmine FBMN export) fail to yield GNPS library annotations under strict parameters, and you need per-feature GNPS scores (c_g, t_g,i, t_g,s) for downstream aggregation as in MS2DECIDE.

## When NOT to use

- A strict GNPS library search already returns matches for most features — the single strict job (0.02 Da, threshold 0.001) is faster and the iterative workflow is unnecessary
- The goal is ISDB-LOTUS or Sirius annotation — those tools use their own matching procedures (ISDB-LOTUS strict 0.02 Da search; Sirius 6 batch mode with Confidence Approximate score), not the GNPS analog search
- You need real-time annotation — the iterative weighted analog search can take up to three hours and launches 27 FBMN jobs

## Inputs

- Quantitative feature table (.csv, e.g. MZmine >2.53 FBMN export)
- MGF file containing tandem mass spectra
- Per-feature GNPS FBMN annotation results (cosine scores, masses, shared-peak counts, InChI availability)
- GNPS account credentials (username, password, email) for launching FBMN jobs

## Outputs

- Per-feature GNPS I indicator table (gnps_i_indicators.csv with c_g, t_g,i, t_g,s)
- Implementation script (gnps_i_procedure.py)
- Report of empty annotations (empty.tsv) listing features with missing GNPS annotations

## How to apply

Start from the per-feature GNPS molecular networking annotation results (cosine scores, masses, shared-peak counts, InChI availability) loaded into a pandas DataFrame. For each feature, first issue the strict GNPS query q(6, 0.02) — 6 shared peaks and 0.02 Da mass tolerance, matching the GNPS strict library search defaults — and record the top match score. If no strict match is returned, progressively relax the query by widening the mass difference tolerance to 0.1 Da and then 10 Da and relaxing the shared-peak requirement, keeping the first returned match score discounted by the relaxation degree reached according to Table S4. Apply the fallback rules: when a match is found with score-without-InChI only, set t_g,i = t_g,s = 0.7; when no match at all is found, set c_g = t_g,i = t_g,s = 0. Assemble the per-feature indicator values into gnps_i_indicators.csv and validate that every row carries a complete indicator set consistent with the discount and fallback rules. In the MS2DECIDE implementation, choosing the 'iterative' GNPS library search option launches 27 FBMN jobs (versus one for 'strict') and can take up to three hours.

## Related tools

- **MS2DECIDE** (Host library implementing K_estimation, which invokes the iterative weighted GNPS analog search and aggregates the resulting annotations) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (External annotation service queried via FBMN; strict search uses 0.02 Da tolerance, iterative search launches 27 relaxed jobs to maximize analog matching) — https://doi.org/10.1038/nbt.3597

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()  # choose 'iterative' at the GNPS library search prompt
```

## Evaluation signals

- Every feature row in gnps_i_indicators.csv carries a complete set of indicator values (c_g, t_g,i, t_g,s) consistent with the Table S4 discount rules
- Features matched under the strict query q(6, 0.02) spot-check identically against direct strict-search results, with no discount applied
- Features with score-without-InChI only show t_g,i = t_g,s = 0.7, and completely unmatched features show c_g = t_g,i = t_g,s = 0
- The iterative option produces 27 FBMN jobs in the user's GNPS account (vs. 1 for strict), and the final K_estimation output is a valid .tsv with ranked features

## Limitations

- The iterative weighted analog search can take up to three hours and launches 27 FBMN jobs from the user's GNPS account
- Because the workflows use a library score threshold of 0.001, some FBMN GNPS workflow failures can occur; increasing the threshold (e.g. to 0.1) is recommended
- When no GNPS match (or no Sirius match) exists, the Tanimoto between GNPS and Sirius cannot be calculated, so defaults of 0.7 are assigned to T_gs and T_gi, which attenuates true novelty signals

## Evidence

- [supplementary] GNPS I queries GNPS starting with the strict query q(6, 0.02) and progressively relaxes constraints (0.1 Da then 10 Da mass difference, then shared-peak requirements), keeping the first match score returned by GNPS: [evidence span withheld — anchor status: not_found]
- [readme] MS2DECIDE integrates GNPS (enhanced with a iterative weighted analog search), ISDB-LOTUS and Sirius6 annotations to deliver reliable recommendation: [evidence span withheld — anchor status: external_doc]
- [readme] the iterative workflow launches 27 FBMN jobs from your GNPS account. In the case of `strict`, only one job will be launched.: [evidence span withheld — anchor status: external_doc]
- [readme] a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances.: [evidence span withheld — anchor status: external_doc]
- [readme] If requested (`yes`), the function generates a report of empty annotations and saves it as `empty.tsv`.: [evidence span withheld — anchor status: external_doc]
