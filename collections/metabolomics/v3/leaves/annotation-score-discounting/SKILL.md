---
name: annotation-score-discounting
description: Use when per-feature GNPS/FBMN annotations contain missing or InChI-less
  matches that would otherwise be dropped, and MS2DECIDE needs complete c_g, t_g,i,
  t_g,s indicator values per feature.
when_to_use_negative:
- The feature already matched under the strict q(6, 0.02) query with InChI available
  — no relaxation or discounting is needed
- The zero annotation itself is informative (as with the strict 0.02 Da ISDB-LOTUS
  search, where a zero answer reflects the novelty definition and must not be defaulted)
- You are scoring Sirius failures, which use a different default (0.5 for the Sirius
  matching score), not the GNPS 0.7 fallback
tools:
- name: MS2DECIDE
  role: Python library implementing the K_estimation workflow that launches the strict
    or iterative weighted GNPS analog search jobs and aggregates the resulting scores
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Source of the annotation cosine scores; queried via 27 successive FBMN jobs
    with progressively relaxed parameters in the iterative mode
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:71194e4e5722e361859e12aa516f05bcbd9f0d2a934b49c867b471f02d057366
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - MS2DECIDE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# annotation-score-discounting

## Summary

A progressive-relaxation procedure (GNPS iterative weighted analog search, GNPS I) that salvages annotation scores for features failing the strict GNPS library search: the strict query q(6, 0.02) is relaxed stepwise (0.02→0.1→10 Da mass tolerance, relaxed shared-peak minimum) and the first returned cosine score is discounted by the relaxation degree, with 0.7/0 fallback defaults.

## When to use

Use when per-feature GNPS/FBMN annotations contain missing or InChI-less matches that would otherwise be dropped, and MS2DECIDE needs complete c_g, t_g,i, t_g,s indicator values per feature.

## When NOT to use

- The feature already matched under the strict q(6, 0.02) query with InChI available — no relaxation or discounting is needed
- The zero annotation itself is informative (as with the strict 0.02 Da ISDB-LOTUS search, where a zero answer reflects the novelty definition and must not be defaulted)
- You are scoring Sirius failures, which use a different default (0.5 for the Sirius matching score), not the GNPS 0.7 fallback

## Inputs

- Per-feature GNPS/FBMN molecular networking annotation results (cosine scores, precursor masses, shared-peak counts, InChI availability)
- Discount schedule from Table S4 (relaxation degree → score discount)
- Quantitative feature table (.csv from MZmine >2.53 FBMN export)
- Tandem MS spectra file (.mgf)

## Outputs

- gnps_i_indicators.csv — per-feature table of c_g, t_g,i, t_g,s values
- gnps_i_procedure.py — saved implementation script
- empty.tsv — optional report of empty annotations from the iterative search

## How to apply

Load the per-feature GNPS molecular networking (FBMN) annotation results — cosine scores, masses, shared-peak counts, InChI availability — into a pandas DataFrame. For each feature, issue the strict GNPS query q(6, 0.02) (6 shared peaks, 0.02 Da mass tolerance, library score threshold 0.001) and record the top match score. If no strict match returns, progressively relax the mass tolerance to 0.1 Da then 10 Da and relax the shared-peak constraint, keeping the first returned match score but discounting it according to the relaxation degree specified in Table S4 — the rationale is that scores from looser analog searches carry less evidence weight than strict matches. Apply fallback defaults: when only a score-without-InChI match exists, set t_g,i = t_g,s = 0.7; when no match is found at all, set c_g = t_g,i = t_g,s = 0. Assemble the per-feature indicator values into gnps_i_indicators.csv and save the implementation as gnps_i_procedure.py.

## Related tools

- **MS2DECIDE** (Python library implementing the K_estimation workflow that launches the strict or iterative weighted GNPS analog search jobs and aggregates the resulting scores) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Source of the annotation cosine scores; queried via 27 successive FBMN jobs with progressively relaxed parameters in the iterative mode)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Every feature row in gnps_i_indicators.csv carries a complete set of indicator values (c_g, t_g,i, t_g,s) consistent with the Table S4 discount rules
- Spot-check features matched under the strict query against the q(6, 0.02) FBMN results — scores must match exactly
- Discounted scores decrease monotonically as the relaxation degree increases (0.02 → 0.1 → 10 Da)
- Rows with only a score-without-InChI match have t_g,i = t_g,s = 0.7; rows with no match have c_g = t_g,i = t_g,s = 0
- The empty.tsv report of empty annotations is consistent with rows lacking a complete indicator set

## Limitations

- The iterative workflow launches 27 FBMN jobs from the GNPS account and can take up to three hours
- At the default library score threshold of 0.001, FBMN GNPS workflow failures can occur; increasing the threshold (e.g., to 0.1) is recommended
- When no GNPS or Sirius match exists, the GNPS–Sirius Tanimoto cannot be calculated, so 0.7 is a fixed default rather than a measured value
- Empty annotations in the iterative search may require manual completion of missing InChIs or SMILES for existing MS/MS spectra on GNPS

## Evidence

- [methods] GNPS I queries GNPS starting with the strict query q(6, 0.02) and progressively relaxes constraints (0.1 Da then 10 Da mass difference, then shared-peak requirements), keeping the first match score: [evidence span withheld — anchor status: not_found]
- [readme] the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances.: [evidence span withheld — anchor status: external_doc]
- [readme] the iterative workflow launches 27 FBMN jobs from your GNPS account. In the case of `strict`, only one job will be launched.: [evidence span withheld — anchor status: external_doc]
- [readme] Since, these workflows use a library score threshold of `0.001` some failures can occur with the FBMN GNPS workflow. We recommend to increase the threshold value (for e.g., 0.1).: [evidence span withheld — anchor status: external_doc]
- [readme] MS2DECIDE integrates GNPS (enhanced with a iterative weighted analog search), ISDB-LOTUS and Sirius6 annotations to deliver reliable recommendation: [evidence span withheld — anchor status: external_doc]
