---
name: annotation-tool-output-aggregation
description: Use when an FBMN feature table (.csv quantitative export from MZmine
  >2.53) plus MGF spectra have parallel annotations from GNPS, Sirius 6, and ISDB-LOTUS
  and you must rank features by knownness/novelty.
when_to_use_negative:
- Only a single tool's annotations are available (e.g., a GNPS-only FBMN job) — the
  Tanimoto-based aggregation of GNPS/Sirius/ISDB-LOTUS outputs cannot be computed.
- The goal is de novo structure elucidation rather than prioritizing/ranking already
  multiannotated features in a molecular network.
- Input is a raw LC-MS/MS dataset with no feature-based molecular networking preprocessing
  (no quantitative .csv and MGF exported from MZmine).
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2DECIDE
  role: Core library whose K_estimation function aggregates the three tools' annotations
    via decision theory into the knownness score K ranking
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Provides FBMN spectral library annotations, either strict or via iterative
    weighted analog search (27 FBMN jobs)
- name: Sirius
  role: Provides Sirius 6 batch-mode annotations using the Confidence Approximate
    score as an aggregation input
- name: ISDB-LOTUS
  role: Provides in silico CFM-ID annotations via get_cfm_annotation(mgf, ISDBtol)
    matched with spectral_lib_matcher/MatchMS
- name: FERMO
  role: Baseline (novelty score) against which the MS2DECIDE k ranking is compared
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:3dbd0abb9cfa560b28576eedc8318db4a2dea835c494aa9c6f0d995d1e58a3ef
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# annotation-tool-output-aggregation

## Summary

Aggregates multiannotated tandem MS/MS spectra from GNPS, Sirius 6, and ISDB-LOTUS using MS2DECIDE's decision-theory knownness score K, enabling prioritization of novel natural products against chemodiverse libraries (e.g., the 95-NP benchmark).

## When to use

Use when an FBMN feature table (.csv quantitative export from MZmine >2.53) plus MGF spectra have parallel annotations from GNPS, Sirius 6, and ISDB-LOTUS and you must rank features by knownness/novelty.

## When NOT to use

- Only a single tool's annotations are available (e.g., a GNPS-only FBMN job) — the Tanimoto-based aggregation of GNPS/Sirius/ISDB-LOTUS outputs cannot be computed.
- The goal is de novo structure elucidation rather than prioritizing/ranking already multiannotated features in a molecular network.
- Input is a raw LC-MS/MS dataset with no feature-based molecular networking preprocessing (no quantitative .csv and MGF exported from MZmine).

## Inputs

- Quantitative feature table (.csv, MZmine >2.53 FBMN export)
- Tandem MS/MS spectra in MGF format
- GNPS account credentials (username, password, email)
- Sirius 6 annotation file (structure_identifications.tsv)
- ISDB-LOTUS spectral library (Zenodo record 8287341)

## Outputs

- Ranked .tsv file of features ordered by knownness score K (filterable for a FBMN continuous color-code overlay)
- Optional empty.tsv report of features with no GNPS annotations after iterative weighted analog search
- Regenerated baseline rank comparison table (e.g., table_4.csv) for GNPS O, GNPS 0.7, GNPS I, Sirius, ISDB-LOTUS, FERMO, and MS2DECIDE k

## How to apply

Generate GNPS FBMN library annotations — either 'strict' (0.02 Da mass difference tolerance, 0.001 cosine threshold) or 'iterative' weighted analog search, raising the minimum cosine (e.g., to 0.1) if FBMN jobs fail — and obtain Sirius 6 'structure_identifications.tsv' using the Confidence Approximate score, plus ISDB-LOTUS annotations via spectral_lib_matcher/MatchMS with a mass tolerance below 0.5 (default 0.02) and the correct ionization mode. Apply the documented fallback values when a tool fails: T_gs = T_gi = 0.7 when no GNPS or Sirius match exists (so Tanimoto cannot be computed), 0.5 as the Sirius matching score when Sirius proposes no annotation, and treat ISDB-LOTUS zero answers (strict 0.02 Da search) as meaningful novelty information. The K_estimation function compiles GNPS, Sirius, and ISDB-LOTUS annotations into a unified dataframe, filters and sorts by the expert-derived knownness score K, and exports a ranked .tsv. Correctness is judged by whether the K ranking recovers known novel compounds: in the 95-compound chemodiverse library benchmark, MS2DECIDE's k is the only method ranking all six unreported NPs (ids 90-95) in the top ten, outperforming GNPS O, default GNPS 0.7, GNPS I, Sirius, ISDB-LOTUS, and FERMO baselines.

## Related tools

- **MS2DECIDE** (Core library whose K_estimation function aggregates the three tools' annotations via decision theory into the knownness score K ranking) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Provides FBMN spectral library annotations, either strict or via iterative weighted analog search (27 FBMN jobs))
- **Sirius** (Provides Sirius 6 batch-mode annotations using the Confidence Approximate score as an aggregation input)
- **ISDB-LOTUS** (Provides in silico CFM-ID annotations via get_cfm_annotation(mgf, ISDBtol) matched with spectral_lib_matcher/MatchMS)
- **FERMO** (Baseline (novelty score) against which the MS2DECIDE k ranking is compared)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Spot-check against published Table 4: GNPS matches only 19 of 89 known features at cosine >= 0.7.
- MS2DECIDE k must place all six unreported compounds (ids 90-95) of the 95-compound chemodiverse library within the top ten ranks — the only method to do so among the evaluated baselines.
- Fallback invariants hold: T_gs and T_gi default to 0.7 when GNPS or Sirius matches are absent; Sirius matching score defaults to 0.5 when no annotation is proposed.
- Output file has a valid .tsv extension, contains a filtered dataframe sorted by K values, and unique mappingFeatureId entries in the Sirius input (no redundant features triggering a ValueError).

## Limitations

- The iterative weighted GNPS analog search launches 27 FBMN jobs and can take up to three hours; with the 0.001 library threshold some FBMN failures occur, so a higher minimum cosine (e.g., 0.1) is recommended.
- Redundant features in the Sirius structure_identifications.tsv (duplicate 'mappingFeatureId' values) raise 'ValueError: DataFrame index must be unique for orient=index'.
- When GNPS or Sirius return no match, the GNPS–Sirius Tanimoto cannot be calculated and a default of 0.7 is assigned to T_gs and T_gi, a built-in approximation in the K computation.

## Evidence

- [abstract] leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS): "leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS)"
- [readme] MS2DECIDE integrates GNPS (enhanced with a iterative weighted analog search), ISDB-LOTUS and Sirius6 annotations to deliver reliable recommendation even in uncertain scenarios.: [evidence span withheld — anchor status: external_doc]
- [readme] In scenarios where there is no match with GNPS or no match with Sirius, the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$: [evidence span withheld — anchor status: external_doc]
- [readme] Sirius annotations were performed in batch mode by using Sirius 6. we utilized the Confidence Approximate score.: [evidence span withheld — anchor status: external_doc]
- [readme] The dataframe is filtered and sorted by \( K \)-values.: [evidence span withheld — anchor status: external_doc]
