---
name: mass-spectrometry-novelty-prioritization
description: Use when a feature-based molecular networking (FBMN) experiment produces
  multiannotated MS/MS features and you need to prioritize putatively novel natural
  products by ranking features with a knownness score instead of relying on single-tool
  annotation ranks.
when_to_use_negative:
- When only a single annotation tool output is available — K_estimation requires GNPS,
  ISDB-LOTUS, and Sirius 6 inputs to aggregate
- When the goal is identity confirmation of known compounds rather than novelty prioritization
  of unreported natural products
- When features lack MS/MS spectra (no MGF entries), since the score is derived from
  tandem mass spectra
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2DECIDE
  role: Core library whose K_estimation function aggregates the three annotations
    and ranks features by the knownness score K
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Provides FBMN library-search annotations (strict or iterative weighted analog
    search) whose cosine scores feed the aggregation
- name: Sirius
  role: Batch-mode Sirius 6 structure annotations (Confidence Approximate score) supplied
    as structure_identifications.tsv
- name: ISDB-LOTUS
  role: In silico annotations via get_cfm_annotation against the ISDB-LOTUS Zenodo
    spectral library using spectral_lib_matcher (MatchMS)
- name: FERMO
  role: Baseline prioritization method (novelty score) used for rank comparison against
    K
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:d6c72ee34e8172424ec4513b24a765134ff35d91d82103537bd69773c25f5cf7
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# mass-spectrometry-novelty-prioritization

## Summary

Aggregates multiannotated tandem MS/MS spectra (GNPS, Sirius 6, ISDB-LOTUS) with decision theory to compute MS2DECIDE's knownness score K, which ranks features by chemical novelty for natural products prioritization and outperforms baseline annotation methods on unreported compounds.

## When to use

Use when a feature-based molecular networking (FBMN) experiment produces multiannotated MS/MS features and you need to prioritize putatively novel natural products by ranking features with a knownness score instead of relying on single-tool annotation ranks.

## When NOT to use

- When only a single annotation tool output is available — K_estimation requires GNPS, ISDB-LOTUS, and Sirius 6 inputs to aggregate
- When the goal is identity confirmation of known compounds rather than novelty prioritization of unreported natural products
- When features lack MS/MS spectra (no MGF entries), since the score is derived from tandem mass spectra

## Inputs

- Quantitative feature table from MZmine (>2.53) FBMN export (.csv)
- Tandem mass spectra in MGF format (.mgf)
- Sirius 6 annotation file (structure_identifications.tsv)
- GNPS account credentials (username, password, email) and FBMN job parameters (strict or iterative weighted analog search)
- ISDB-LOTUS spectral library (Zenodo record 8287341) accessed via spectral_lib_matcher

## Outputs

- Ranked .tsv file of features ordered by the knownness score K (K.tsv) with aggregated GNPS/Sirius/ISDB-LOTUS annotations
- Optional empty.tsv report of features left unannotated by the iterative weighted GNPS analog search
- Regenerated baseline-vs-K rank comparison table (e.g., table_4.csv) when benchmarking against GNPS O, GNPS 0.7, GNPS I, Sirius, ISDB-LOTUS, and FERMO

## How to apply

Run the ms2decide K_estimation function on the MZmine-exported quantitative table (.csv), the MGF of tandem mass spectra, the Sirius 6 structure_identifications.tsv (using the 'approximate' confidence score mode), and GNPS FBMN results (strict 0.02 Da library search or iterative weighted analog search, with a raised minimum cosine threshold such as 0.1 to avoid FBMN failures). ISDB-LOTUS annotations are computed via get_cfm_annotation(mgf, ISDBtol) with a mass tolerance below 0.5 (default 0.02 Da) against the ISDB-LOTUS Zenodo spectral library using spectral_lib_matcher/MatchMS. Missing matches are imputed by expert-modeled defaults: Tanimoto values T_gs and T_gi set to 0.7 when GNPS/Sirius agreement cannot be computed, and 0.5 for Sirius when no annotation is proposed, while a zero ISDB-LOTUS answer under the strict 0.02 Da search is treated as informative for novelty. The function compiles a unified dataframe, filters and sorts features by the knownness score K, and exports a ranked .tsv that can be mapped as a continuous color code onto the FBMN graph; to benchmark against baselines, recompute ranks under GNPS O (q(6,0.02), 1e-4 threshold), default GNPS 0.7 cosine, GNPS I, Sirius confidence, ISDB-LOTUS cosine, and FERMO novelty, as in the 95-NP chemodiverse library comparison where only K placed all six unreported compounds (ids 90-95) in the top ten.

## Related tools

- **MS2DECIDE** (Core library whose K_estimation function aggregates the three annotations and ranks features by the knownness score K) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Provides FBMN library-search annotations (strict or iterative weighted analog search) whose cosine scores feed the aggregation)
- **Sirius** (Batch-mode Sirius 6 structure annotations (Confidence Approximate score) supplied as structure_identifications.tsv)
- **ISDB-LOTUS** (In silico annotations via get_cfm_annotation against the ISDB-LOTUS Zenodo spectral library using spectral_lib_matcher (MatchMS))
- **FERMO** (Baseline prioritization method (novelty score) used for rank comparison against K)

## Examples

```
python -c "from ms2decide.K_estimation import K_estimation; K_estimation()"
```

## Evaluation signals

- On the 95-NP chemodiverse library benchmark, verify that MS2DECIDE k is the only method placing all six unreported compounds (ids 90-95) within the top ten ranks
- Spot-check that default GNPS matches only 19 of 89 known features at cosine >= 0.7 against published Table 4
- Output file is a valid .tsv with a sorted rank column derived from K, and unique mappingFeatureId entries (redundant Sirius features raise a ValueError)
- Imputation invariants hold: T_gs = T_gi = 0.7 when GNPS-Sirius Tanimoto is not computable, and Sirius matching score = 0.5 when Sirius proposes no annotation
- Iterative GNPS mode yields the expected 27 FBMN jobs; any features with empty annotations appear in the empty.tsv report

## Limitations

- GNPS library score threshold of 0.001 can cause FBMN job failures; a higher threshold (e.g., 0.1) is recommended
- The iterative weighted analog search can take up to three hours and launches 27 FBMN jobs from the user's GNPS account
- Redundant features in Sirius structure_identifications.tsv (duplicate 'mappingFeatureId' values) trigger "ValueError: DataFrame index must be unique for orient='index'." and must be resolved manually
- Missing-match defaults (Tanimoto 0.7, Sirius score 0.5) are expert-imputed values, not measured scores

## Evidence

- [abstract] aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS): "aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS)"
- [abstract] case studies novelty: "We demonstrate, through two case studies, that MS2DECIDE reliably captures the novelty of natural products from their tandem mass spectra."
- [readme] default Tanimoto imputation: [evidence span withheld — anchor status: external_doc]
- [readme] ISDB zero-answer novelty semantics: [evidence span withheld — anchor status: external_doc]
- [readme] Sirius confidence mode and fallback: [evidence span withheld — anchor status: external_doc]
