---
name: per-feature-indicator-table-construction
description: Use when FBMN/GNPS annotation coverage is incomplete (missing InChIs
  or no matches per feature) and you need per-feature GNPS cosine/tanimoto indicators
  to feed MS2DECIDE's K_estimation aggregation.
when_to_use_negative:
- When a strict GNPS library search suffices — the strict option launches only one
  FBMN job at 0.02 Da tolerance and does not require relaxation discounts
- When every feature already has a complete InChI-annotated GNPS match, so no fallback
  (0.7) or zero-default indicators are needed
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2DECIDE
  role: Library whose K_estimation function consumes the per-feature indicators and
    offers the 'iterative' GNPS weighted analog search option that generates them
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Provides the FBMN library-search jobs (strict q(6, 0.02) or the 27-job iterative
    analog search) whose match scores populate the indicator table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:cea874a758d4ba939f282d756fa9cb014ed6012a0e297bdd45da2a16db1d87ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - MS2DECIDE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# per-feature-indicator-table-construction

## Summary

Builds the per-feature GNPS indicator table (c_g, t_g,i, t_g,s) for MS2DECIDE by reconstructing the GNPS I iterative weighted analog search: progressively relaxed GNPS queries with discount factors, plus fallback defaults (0.7 for score-without-InChI matches, 0 for no match).

## When to use

Use when FBMN/GNPS annotation coverage is incomplete (missing InChIs or no matches per feature) and you need per-feature GNPS cosine/tanimoto indicators to feed MS2DECIDE's K_estimation aggregation.

## When NOT to use

- When a strict GNPS library search suffices — the strict option launches only one FBMN job at 0.02 Da tolerance and does not require relaxation discounts
- When every feature already has a complete InChI-annotated GNPS match, so no fallback (0.7) or zero-default indicators are needed

## Inputs

- Per-feature GNPS molecular networking (FBMN) annotation results: cosine scores, masses, shared-peak counts, InChI availability
- Table S4 discount factors for relaxation degrees
- GNPS account credentials (username, password, email) when running the iterative search via the ms2decide library
- Quantitative feature table (.csv) and tandem MS data (.mgf) exported from MZmine for the FBMN jobs

## Outputs

- gnps_i_indicators.csv with a complete set of indicator values (c_g, t_g,i, t_g,s) per feature
- gnps_i_procedure.py implementing the iterative weighted analog search
- Optionally empty.tsv, a report of empty annotations from the iterative weighted GNPS analog search to complete missing InChIs/SMILES

## How to apply

Load the per-feature GNPS molecular networking annotation results (cosine scores, masses, shared-peak counts, InChI availability) into a pandas DataFrame. For each feature, issue the strict GNPS query q(6, 0.02) — 6 shared peaks, 0.02 Da mass tolerance — and record the top match score; the README notes the strict workflow corresponds to 'a typical mass difference tolerance of 0.02 Da and threshold value of 0.001'. If no strict match is returned, progressively relax the query (0.1 Da, then 10 Da mass difference, then shared-peak requirements), keeping the first returned match score discounted by the relaxation degree per Table S4; in the full library implementation this iterative search launches 27 FBMN jobs from the user's GNPS account. Apply fallback defaults: when a match is found with score-without-InChI only, set t_g,i = t_g,s = 0.7 (the README assigns 0.7 to T_gs and T_gi when a match is missing); when no match is found at all, set c_g = t_g,i = t_g,s = 0. Assemble the per-feature indicators (c_g, t_g,i, t_g,s) into gnps_i_indicators.csv and save the implementation as gnps_i_procedure.py.

## Related tools

- **MS2DECIDE** (Library whose K_estimation function consumes the per-feature indicators and offers the 'iterative' GNPS weighted analog search option that generates them) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Provides the FBMN library-search jobs (strict q(6, 0.02) or the 27-job iterative analog search) whose match scores populate the indicator table)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Every feature row in gnps_i_indicators.csv carries a complete set of indicator values (c_g, t_g,i, t_g,s) consistent with the Table S4 discount rules
- Spot-check strict-query features: their scores reproduce the plain q(6, 0.02) FBMN results with no discount applied
- Fallback invariants: score-without-InChI matches show t_g,i = t_g,s = 0.7; unmatched features show c_g = t_g,i = t_g,s = 0
- Relaxed-query matches show discounted scores monotonically decreasing with the relaxation degree reached (0.02 → 0.1 → 10 Da → shared-peak relaxation)
- If the library-level iterative option is used, 27 FBMN jobs should appear in the user's GNPS account (vs 1 for strict)

## Limitations

- The iterative workflow can take up to three hours and launches 27 FBMN jobs, so failures can occur with the FBMN GNPS workflow; the README recommends raising the library score threshold from 0.001 to e.g. 0.1
- A match found only without InChI yields no true tanimoto values, so defaults of 0.7 (t_g,i, t_g,s) are substitutes, not measured similarities
- When no match is found at all, all indicators default to 0, conflating 'no GNPS evidence' with genuine novelty information (unlike ISDB-LOTUS, where zero answers under strict 0.02 Da search are treated as informative)

## Evidence

- [readme] MS2DECIDE integrates GNPS (enhanced with a iterative weighted analog search), ISDB-LOTUS and Sirius6 annotations: [evidence span withheld — anchor status: external_doc]
- [readme] fallback default of 0.7: [evidence span withheld — anchor status: external_doc]
- [readme] iterative workflow launches 27 FBMN jobs: [evidence span withheld — anchor status: external_doc]
- [methods] GNPS I progressive relaxation from strict query: [evidence span withheld — anchor status: not_found]
