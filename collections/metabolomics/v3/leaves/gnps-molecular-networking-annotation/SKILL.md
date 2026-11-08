---
name: gnps-molecular-networking-annotation
description: Use when you have an MGF file of tandem MS spectra plus a MZmine (>2.53)
  quantitative feature table and need per-node spectral-library annotation scores
  (cosine scores) for every ion feature before k-aggregation.
when_to_use_negative:
- You only have the raw vendor LC-MS/MS files and no MZmine FBMN quantitative table
  (.csv) and MGF export — the K_estimation input format will not be accepted
- You need de novo structure elucidation rather than spectral-library annotation —
  Sirius/ISDB-LOTUS streams, not the GNPS library search, address unknowns (ISDB zero
  answers are treated as novelty information)
- You lack internet access or a GNPS account, since the workflow uploads files to
  and queries the GNPS platform
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Runs FBMN library search (strict or iterative weighted analog search) to produce
    per-node spectral annotation scores aggregated by MS2DECIDE
  repo: https://github.com/MejriY/MS2DECIDE
- name: MS2DECIDE
  role: Python library whose K_estimation function launches the GNPS FBMN job and
    aggregates GNPS, Sirius and ISDB-LOTUS annotations into a K-ranked .tsv
  repo: https://github.com/MejriY/MS2DECIDE
- name: Sirius
  role: Sirius 6 batch annotations (structure_identifications.tsv, Confidence Approximate
    score) are compiled alongside GNPS results; missing matches default to a 0.5 score
- name: ISDB-LOTUS
  role: ISDB-LOTUS spectral matching (via cfm-id/spectral_lib_matcher on MatchMS,
    strict 0.02 Da search) supplies the third indicator stream for K aggregation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:23ce9e868f56545ae74b91e691c2ac0903b93d1e10108122ce6b1d8a3494e313
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
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gnps-molecular-networking-annotation

## Summary

Performs GNPS molecular networking / feature-based molecular networking (FBMN) annotation of LC-MS/MS ion features, one of the three annotation streams (with Sirius and ISDB-LOTUS) that MS2DECIDE aggregates via its knownness score K for natural products prioritization.

## When to use

Use when you have an MGF file of tandem MS spectra plus a MZmine (>2.53) quantitative feature table and need per-node spectral-library annotation scores (cosine scores) for every ion feature before k-aggregation.

## When NOT to use

- You only have the raw vendor LC-MS/MS files and no MZmine FBMN quantitative table (.csv) and MGF export — the K_estimation input format will not be accepted
- You need de novo structure elucidation rather than spectral-library annotation — Sirius/ISDB-LOTUS streams, not the GNPS library search, address unknowns (ISDB zero answers are treated as novelty information)
- You lack internet access or a GNPS account, since the workflow uploads files to and queries the GNPS platform

## Inputs

- Quantitative feature table in .csv (MZmine >2.53 FBMN export format)
- Tandem mass spectrometry data in MGF format (.mgf)
- GNPS account credentials (username, password, email)
- Optional: Sirius 6 structure_identifications.tsv and ISDB-LOTUS annotation settings (ionization mode POS/NEG, mass tolerance <=0.5, default 0.02)

## Outputs

- Per-node GNPS FBMN annotation results (spectral-library/analog matching scores for all ion features)
- Ranked K.tsv from MS2DECIDE K_estimation, aggregating GNPS, Sirius and ISDB-LOTUS annotations
- Optional empty.tsv report of empty annotations when the iterative weighted GNPS analog search was used

## How to apply

Export the quantitative table as .csv and the tandem MS data as .mgf from MZmine (>2.53) FBMN format, then run the K_estimation function, which authenticates with GNPS (username, password, email) and launches the FBMN job. Choose the 'strict' library search (mass difference tolerance 0.02 Da, threshold 0.001, one FBMN job) or the 'iterative' weighted analog search (up to three hours, launching 27 FBMN jobs from your GNPS account). Because the workflows default to a library score threshold of 0.001, which can cause FBMN failures, increase the minimum cosine score prompt value (e.g. to 0.1). The resulting per-node GNPS annotation scores feed MS2DECIDE's K_estimation together with Sirius6 structure_identifications.tsv and ISDB-LOTUS cfm-id annotations (mass tolerance <=0.5, default 0.02 Da) to rank features by knownness K; when GNPS yields no match the T_gs and T_gi tanimoto defaults of 0.7 apply.

## Related tools

- **GNPS** (Runs FBMN library search (strict or iterative weighted analog search) to produce per-node spectral annotation scores aggregated by MS2DECIDE) — https://github.com/MejriY/MS2DECIDE
- **MS2DECIDE** (Python library whose K_estimation function launches the GNPS FBMN job and aggregates GNPS, Sirius and ISDB-LOTUS annotations into a K-ranked .tsv) — https://github.com/MejriY/MS2DECIDE
- **Sirius** (Sirius 6 batch annotations (structure_identifications.tsv, Confidence Approximate score) are compiled alongside GNPS results; missing matches default to a 0.5 score)
- **ISDB-LOTUS** (ISDB-LOTUS spectral matching (via cfm-id/spectral_lib_matcher on MatchMS, strict 0.02 Da search) supplies the third indicator stream for K aggregation)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Output K .tsv exists, is loadable, and contains a ranked row per ion feature of the FBMN job
- For the P. mutica dataset (MassIVE MSV95596), the 754-feature ranking reproduces the reported reference ranks, e.g. pleiomutine (m/z 631.3986) ranked 525/754
- Import sanity check succeeds: python -c "import ms2decide; print('MS2DECIDE installed successfully!')"
- With the iterative analog search, empty.tsv correctly reports features with empty annotations so missing InChIs/SMILES can be completed
- Library search used the requested cosine threshold (0.001 default, raised e.g. to 0.1) without FBMN job failures

## Limitations

- Library score threshold of 0.001 can cause FBMN workflow failures; increasing it (e.g. 0.1) is recommended
- The iterative weighted analog search can take up to three hours and launches 27 FBMN jobs from the user's GNPS account
- When there is no GNPS match, tanimoto between GNPS and Sirius cannot be calculated and defaults of 0.7 are assigned to T_gs and T_gi
- Sirius6 annotation files may contain redundant features in the mappingFeatureId column, raising "ValueError: DataFrame index must be unique for orient='index'." and requiring manual deduplication
- ISDB-LOTUS mass tolerance must be below 0.5 (default 0.02); a zero ISDB answer is treated as meaningful information about novelty rather than as missing data

## Evidence

- [readme] MS2DECIDE integrates GNPS (enhanced with a iterative weighted analog search), ISDB-LOTUS and Sirius6 annotations to deliver reliable recommendation: [evidence span withheld — anchor status: external_doc]
- [abstract] aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS): "tandem mass spectrum to decision (MS2DECIDE) is presented which leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and"
- [readme] strict: Uses a typical mass difference tolerance of 0.02 Da and threshold value of 0.001: [evidence span withheld — anchor status: external_doc]
- [readme] these workflows use a library score threshold of 0.001 some failures can occur with the FBMN GNPS workflow: [evidence span withheld — anchor status: external_doc]
- [readme] a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$: [evidence span withheld — anchor status: external_doc]
- [readme] the iterative workflow launches 27 FBMN jobs from your GNPS account: [evidence span withheld — anchor status: external_doc]
