---
name: spectral-library-annotation-scoring
description: Use when a feature-based molecular networking job has multiannotated
  MS/MS spectra (GNPS FBMN quantitative table, MGF, Sirius6 structure_identifications.tsv)
  and you need to prioritize features by annotation reliability or benchmark annotation
  ranks.
when_to_use_negative:
- Input is a single-tool annotation with no multiannotation sources (no Sirius6 or
  ISDB-LOTUS outputs), since the knownness score k requires aggregating all three
  tools
- The analysis target is de novo structure elucidation rather than ranking/prioritizing
  library-based annotations of MS/MS features
- The MGF/quantitative pair does not come from a GNPS FBMN workflow with MZmine export
  format, as required file formats will not validate
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2DECIDE
  role: Aggregates GNPS, Sirius, and ISDB-LOTUS annotations with decision theory to
    compute the knownness score k for feature prioritization
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Provides FBMN library spectral matching (strict or iterative weighted analog
    search) used as one annotation source for k aggregation and as baseline methods
    GNPS O / GNPS I
- name: Sirius
  role: Supplies structure identifications with Confidence Approximate score (defaulting
    to 0.5 matching score when no annotation is proposed) as an annotation source
- name: ISDB-LOTUS
  role: Provides in silico CFM-ID-based spectral annotations via get_cfm_annotation
    with 0.02 Da mass tolerance against ISDB-LOTUS spectra
- name: FERMO
  role: Baseline method providing a novelty score used in the rank comparison against
    MS2DECIDE k
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:20033ee1162cfe56282ca22d7b1425770e4938673efaf4cd77fddf5ff9d0e126
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - FERMO
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectral-library-annotation-scoring

## Summary

Score and rank spectral library annotations of tandem MS/MS features by aggregating GNPS, Sirius, and ISDB-LOTUS outputs with MS2DECIDE's decision-theory-based knownness score k, enabling benchmarking against baseline annotation methods on a 95-natural-product chemodiverse library.

## When to use

Use when a feature-based molecular networking job has multiannotated MS/MS spectra (GNPS FBMN quantitative table, MGF, Sirius6 structure_identifications.tsv) and you need to prioritize features by annotation reliability or benchmark annotation ranks.

## When NOT to use

- Input is a single-tool annotation with no multiannotation sources (no Sirius6 or ISDB-LOTUS outputs), since the knownness score k requires aggregating all three tools
- The analysis target is de novo structure elucidation rather than ranking/prioritizing library-based annotations of MS/MS features
- The MGF/quantitative pair does not come from a GNPS FBMN workflow with MZmine export format, as required file formats will not validate

## Inputs

- Quantitative feature table exported from MZmine (>2.53) FBMN (.csv)
- Tandem mass spectrometry data in MGF format (.mgf)
- Sirius 6 annotation file (structure_identifications.tsv)
- GNPS account credentials (username, password, email) for FBMN job setup
- ISDB-LOTUS spectral library (Zenodo record 8287341) via spectral_lib_matcher/MatchMS

## Outputs

- Filtered and k-ranked multiannotation table (.tsv) aggregating GNPS, Sirius, and ISDB-LOTUS annotations
- Optional empty-annotation report (empty.tsv) for iterative weighted GNPS analog search
- Regenerated baseline rank comparison table (e.g. table_4.csv) covering GNPS O, default GNPS 0.7, GNPS I, Sirius, ISDB-LOTUS, FERMO, and MS2DECIDE k

## How to apply

Load the per-compound annotation outputs from the MejriY/MS2DECIDE repository for the 95-NP chemodiverse library. Run ms2decide.K_estimation, providing the quantitative table (.csv), the MGF file, GNPS credentials, the Sirius6 structure_identifications.tsv (using the Confidence Approximate score, defaulting Sirius matching score to 0.5 when Sirius proposes no annotation), and ISDB-LOTUS annotations via get_cfm_annotation with mass tolerance 0.02 Da (< 0.5). For GNPS, choose the iterative weighted analog search or strict library search; where no GNPS–Sirius match exists, assign the default value 0.7 to T_gs and T_gi. The function filters and sorts features by the knownness score k and exports a ranked .tsv. To reproduce the baseline comparison (Table 4), recompute ranks for compounds 90–95 under GNPS O (q(6,0.02) filtering, 1e-4 threshold), default GNPS cosine at the 0.7 cutoff, GNPS I, Sirius confidence score, ISDB-LOTUS cosine, and FERMO novelty score, then verify that MS2DECIDE k is the only method ranking all six unreported compounds within the top ten.

## Related tools

- **MS2DECIDE** (Aggregates GNPS, Sirius, and ISDB-LOTUS annotations with decision theory to compute the knownness score k for feature prioritization) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Provides FBMN library spectral matching (strict or iterative weighted analog search) used as one annotation source for k aggregation and as baseline methods GNPS O / GNPS I)
- **Sirius** (Supplies structure identifications with Confidence Approximate score (defaulting to 0.5 matching score when no annotation is proposed) as an annotation source)
- **ISDB-LOTUS** (Provides in silico CFM-ID-based spectral annotations via get_cfm_annotation with 0.02 Da mass tolerance against ISDB-LOTUS spectra)
- **FERMO** (Baseline method providing a novelty score used in the rank comparison against MS2DECIDE k)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- MS2DECIDE k is the only evaluated method that ranks all six unreported natural products (compound ids 90–95) within the top ten of the 95-compound chemodiverse library
- Spot-check that default GNPS matches only 19 of 89 known features at cosine >= 0.7, matching published Table 4
- Output .tsv has a valid extension, unique Sirius mappingFeatureId entries (no redundant features), and is sorted by descending k values
- Default-fill invariants hold: T_gs = T_gi = 0.7 when GNPS–Sirius Tanimoto cannot be computed, and Sirius matching score = 0.5 when Sirius proposes no annotation
- The iterative GNPS workflow produces 27 FBMN jobs; the strict workflow produces 1, consistent with the configured library-search mode

## Limitations

- With a library score threshold of 0.001 some FBMN GNPS workflow failures can occur; the README recommends raising the minimum cosine threshold (e.g., 0.1)
- Sirius sometimes cannot propose an annotation; a default matching score of 0.5 is associated in these cases, and redundant features in structure_identifications.tsv (mappingFeatureId column) can raise 'ValueError: DataFrame index must be unique for orient="index".'
- The iterative weighted analog search can take up to three hours and launches 27 FBMN jobs from the user's GNPS account
- GNPS credentials and internet connectivity are required to run K_estimation, since it must access GNPS and related services

## Evidence

- [results] MS2DECIDE's knownness score k was the only method among the evaluated baselines to place all six unreported NPs (ids 90-95) within the top ten ranks of the 95-compound chemodiverse library.: [evidence span withheld — anchor status: not_found]
- [abstract] k aggregates three annotation tools: "leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools"
- [readme] default T_gs/T_gi = 0.7 and Sirius 0.5 fallback: [evidence span withheld — anchor status: external_doc]
- [readme] Sirius 0.5 fallback and ISDB zero-answer rationale: [evidence span withheld — anchor status: external_doc]
- [full_text] GNPS 0.7 cosine baseline spot-check: [evidence span withheld — anchor status: not_found]
- [readme] iterative workflow cost: [evidence span withheld — anchor status: external_doc]
