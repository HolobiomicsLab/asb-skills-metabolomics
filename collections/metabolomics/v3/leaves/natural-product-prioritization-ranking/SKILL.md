---
name: natural-product-prioritization-ranking
description: Use when you have an MZmine FBMN quantitative table (.csv) plus an MGF
  of tandem mass spectra and need to prioritize features by novelty — e.g., to surface
  unreported NPs or rank candidate annotations in an alkaloid/plant extract.
when_to_use_negative:
- You have no tandem MS/MS spectra (no .mgf) — the k score aggregates MS/MS-based
  annotation indicators and cannot be computed from a feature table alone
- You need a single-compound structural elucidation rather than prioritization of
  many features across a sample
- Your annotation tools are not GNPS, Sirius, and ISDB-LOTUS — the decision-theory
  k function is calibrated to these three indicators
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2DECIDE
  role: Core library whose K_estimation function aggregates GNPS, ISDB-LOTUS, and
    Sirius indicators via decision theory to compute and rank k scores
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: FBMN library search (strict or iterative weighted analog search) producing
    per-node spectral-library annotation scores
- name: Sirius
  role: Sirius 6 batch-mode structural annotation (structure_identifications.tsv,
    approximate confidence score) feeding the k aggregation
- name: ISDB-LOTUS
  role: In silico spectral annotation via get_cfm_annotation matched against ISDB-LOTUS
    spectral data with spectral_lib_matcher
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:8a3c44806c1885c50fda3f030efa073444cd769e6e9824e977810468faa57116
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# natural-product-prioritization-ranking

## Summary

Rank LC-MS/MS ion features by the MS2DECIDE knownness score k, computed by aggregating GNPS (FBMN), Sirius 6, and ISDB-LOTUS annotation indicators with a decision-theory function that models expert preference elicitation for natural product novelty prioritization.

## When to use

Use when you have an MZmine FBMN quantitative table (.csv) plus an MGF of tandem mass spectra and need to prioritize features by novelty — e.g., to surface unreported NPs or rank candidate annotations in an alkaloid/plant extract.

## When NOT to use

- You have no tandem MS/MS spectra (no .mgf) — the k score aggregates MS/MS-based annotation indicators and cannot be computed from a feature table alone
- You need a single-compound structural elucidation rather than prioritization of many features across a sample
- Your annotation tools are not GNPS, Sirius, and ISDB-LOTUS — the decision-theory k function is calibrated to these three indicators

## Inputs

- Quantitative feature table (.csv, MZmine >2.53 FBMN export format)
- Tandem mass spectrometry file (.mgf)
- Sirius 6 annotation file (structure_identifications.tsv)
- GNPS platform credentials (username, password, email)
- ISDB-LOTUS spectral data (zenodo.org/records/8287341, matched via spectral_lib_matcher/MatchMS)

## Outputs

- Ranked .tsv of ion features sorted by increasing knownness score k (GNPS/Sirius/ISDB-LOTUS indicators compiled, filtered, and ordered)
- Optional empty.tsv report of empty annotations (iterative weighted GNPS analog search)

## How to apply

Run the K_estimation function from the ms2decide Python package (Python ≥3.8), supplying a quantitative .csv (MZmine >2.53 FBMN export) and an .mgf of MS/MS spectra. Launch a GNPS FBMN job — either 'strict' (0.02 Da mass difference tolerance) or 'iterative' weighted analog search (launches 27 FBMN jobs, up to ~3 h) — then compute ISDB-LOTUS annotations via get_cfm_annotation(mgf, ISDBtol) with ionization mode POS/NEG and mass tolerance <0.5 (default 0.02), and load the Sirius 6 structure_identifications.tsv choosing 'exact' or 'approximate' confidence scoring. Handle missing annotations per the paper's conventions: T_gs and T_gi default to 0.7 when GNPS–Sirius/ISDB Tanimoto cannot be computed, Sirius matching score defaults to 0.5 when no annotation is proposed, while a zero ISDB-LOTUS answer at 0.02 Da is treated as informative for novelty. The function compiles the three tools' indicators, applies the decision-theory k function, sorts features by increasing k, and writes a ranked .tsv.

## Related tools

- **MS2DECIDE** (Core library whose K_estimation function aggregates GNPS, ISDB-LOTUS, and Sirius indicators via decision theory to compute and rank k scores) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (FBMN library search (strict or iterative weighted analog search) producing per-node spectral-library annotation scores)
- **Sirius** (Sirius 6 batch-mode structural annotation (structure_identifications.tsv, approximate confidence score) feeding the k aggregation)
- **ISDB-LOTUS** (In silico spectral annotation via get_cfm_annotation matched against ISDB-LOTUS spectral data with spectral_lib_matcher)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Output .tsv exists, is sorted by increasing k, and each ion feature receives exactly one rank (754 rows for the P. mutica real-case dataset)
- On the 95-NP chemodiverse library validation, the six unreported NPs (ids 90-95) all fall within the top 10 ranks (Table 3 / Table S7)
- Spot-check reference entries: Swietenin at k=-4.5 rank 3 and Caffeic acid at k=115.5 rank 94
- On the P. mutica dataset (MassIVE MSV95596), pleiomutine (m/z 631.3986) is ranked 525/754, matching the article's Results section
- Missing-annotation defaults are respected: T_gs=T_gi=0.7 when Tanimoto is undefined and Sirius score 0.5 when no annotation is proposed

## Limitations

- The iterative weighted analog GNPS search launches 27 FBMN jobs and can take up to three hours
- GNPS workflows use a library score threshold of 0.001, which can cause FBMN failures; the README recommends increasing it (e.g., 0.1)
- Sirius structure_identifications.tsv can contain redundant mappingFeatureId values, raising 'ValueError: DataFrame index must be unique for orient='index'' — deduplicate before loading
- Missing annotations force default imputations (T_gs/T_gi=0.7, Sirius score 0.5), which can mask true annotation uncertainty in the k ranking

## Evidence

- [readme] MS2DECIDE is a Python library that leverages decision theory to assist chemists in natural products multiannotation and prioritization.: [evidence span withheld — anchor status: external_doc]
- [readme] a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances.: [evidence span withheld — anchor status: external_doc]
- [readme] we associated a value of 0.5 to Sirius matching score.: [evidence span withheld — anchor status: external_doc]
- [abstract] aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS): "tandem mass spectrum to decision (MS2DECIDE) is presented which leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and"
- [abstract] MS2DECIDE reliably captures the novelty of natural products from their tandem mass spectra.: "We demonstrate, through two case studies, that MS2DECIDE reliably captures the novelty of natural products from their tandem mass spectra."
