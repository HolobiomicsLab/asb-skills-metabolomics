---
name: rank-reproducibility-verification-against-published-values
description: Use when a reimplementation or rerun of an aggregation/prioritization
  pipeline (GNPS + Sirius + ISDB-LOTUS indicators) must be validated by reproducing
  specific per-feature ranks reported in a publication.
when_to_use_negative:
- The dataset lacks the required FBMN export pair (quantitative .csv + .mgf) or the
  Sirius structure_identifications.tsv, so the three-tool aggregation cannot be run
  as published
- No published/reference rank values exist for the dataset, in which case a direct
  k-ranking run is appropriate rather than a reproducibility verification
- Only raw MS/MS spectra are available without GNPS, Sirius, and ISDB-LOTUS annotation
  outputs — the k pipeline aggregates these three, not raw spectra
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2DECIDE
  role: Provides the K_estimation decision-theory aggregation that produces the k-ranked
    feature table being reproduced
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: FBMN library search (strict or iterative weighted analog search) supplying
    per-node annotation scores to the aggregation
- name: Sirius
  role: Batch-mode Sirius 6 structure identification (structure_identifications.tsv,
    Confidence Approximate) feeding the k aggregation
- name: ISDB-LOTUS
  role: Strict (0.02 Da) spectral-library annotation via get_cfm_annotation providing
    the third per-node indicator
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:84789295543bf5f15365d2fdadcca2d83c9a091f287ce443e44e90e07598b0f9
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# rank-reproducibility-verification-against-published-values

## Summary

Reproduce a published feature-prioritization ranking (here the MS2DECIDE k-pipeline over 754 LC-MS/MS ion features of the Pleiocarpa mutica alkaloid extract) and verify that node-level ranks match the values reported in the article, e.g. pleiomutine (m/z 631.3986) at rank 525/754.

## When to use

Use when a reimplementation or rerun of an aggregation/prioritization pipeline (GNPS + Sirius + ISDB-LOTUS indicators) must be validated by reproducing specific per-feature ranks reported in a publication.

## When NOT to use

- The dataset lacks the required FBMN export pair (quantitative .csv + .mgf) or the Sirius structure_identifications.tsv, so the three-tool aggregation cannot be run as published
- No published/reference rank values exist for the dataset, in which case a direct k-ranking run is appropriate rather than a reproducibility verification
- Only raw MS/MS spectra are available without GNPS, Sirius, and ISDB-LOTUS annotation outputs — the k pipeline aggregates these three, not raw spectra

## Inputs

- P. mutica real-case LC-MS/MS dataset from MassIVE accession MSV95596
- MZmine (>2.53) FBMN export quantitative feature table (.csv)
- MGF file with tandem mass spectra
- Sirius 6 annotation file (structure_identifications.tsv)
- ISDB-LOTUS spectral library (Zenodo record 8287341) via spectral_lib_matcher/MatchMS
- MejriY/MS2DECIDE repository (k pipeline reconstruction)
- Published reference rank values for validation (pleiomutine 525, id 92 at 326, m/z 817.4137 at 92, pleiokomenine A at 741)

## Outputs

- ms2decide_ranking.csv with exactly 754 rows, one per ion feature, containing aggregated k scores and ranks
- Validation report confirming the four reported node ranks match the article's Results values
- Optional empty.tsv report of empty annotations when iterative weighted GNPS analog search is used

## How to apply

Reconstruct the pipeline end-to-end before checking ranks: download the P. mutica tandem MS dataset from MassIVE accession MSV95596, export the MZmine (>2.53) FBMN quantitative table (.csv) and MGF, run GNPS FBMN library search, Sirius 6 batch mode (structure_identifications.tsv, Confidence Approximate score), and ISDB-LOTUS annotation via get_cfm_annotation with strict 0.02 Da mass tolerance. Apply the MS2DECIDE k aggregation of the three tools' outputs, honoring the paper's fallback values (T_gs and T_gi = 0.7 when GNPS/Sirius matches are absent; Sirius matching score 0.5 when Sirius proposes no annotation), then export the full ranked table (ms2decide_ranking.csv / .tsv, one row per feature). Validation is by schema and value checks: the output must contain exactly 754 feature rows, be sorted by k-value, and the reported nodes must fall at their reference ranks — pleiomutine (m/z 631.3986) at 525/754, the node with id 92 at 326, m/z 817.4137 at 92, and pleiokomenine A at 741. Any rank mismatch should trigger review of thresholds (e.g. GNPS minimum cosine score) and fallback assignments before claiming reproducibility.

## Related tools

- **MS2DECIDE** (Provides the K_estimation decision-theory aggregation that produces the k-ranked feature table being reproduced) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (FBMN library search (strict or iterative weighted analog search) supplying per-node annotation scores to the aggregation)
- **Sirius** (Batch-mode Sirius 6 structure identification (structure_identifications.tsv, Confidence Approximate) feeding the k aggregation)
- **ISDB-LOTUS** (Strict (0.02 Da) spectral-library annotation via get_cfm_annotation providing the third per-node indicator)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Output file contains exactly 754 rows (one per ion feature of the P. mutica dataset)
- Known MIA pleiomutine (node at m/z 631.3986) is ranked 525/754, matching the reference value in the Results section
- Node with id 92 is ranked 326, m/z 817.4137 is ranked 92, and pleiokomenine A is ranked 741, all matching reference values
- Ranking is sorted by k-values and fallback defaults (T_gs/T_gi = 0.7; Sirius score 0.5 for missing annotations) are applied where annotations are absent
- GNPS FBMN jobs completed above the minimum cosine-score threshold (raise from 0.001 to e.g. 0.1 if failures occur)

## Limitations

- The iterative weighted analog search launches 27 FBMN jobs and can take up to three hours; strict mode launches one job
- Library score threshold 0.001 can cause FBMN workflow failures; a higher threshold (e.g. 0.1) is recommended
- Redundant features in the 'mappingFeatureId' column of structure_identifications.tsv raise "ValueError: DataFrame index must be unique for orient='index'" and must be deduplicated first
- ISDB-LOTUS mass tolerance must be < 0.5 (default 0.02); a zero ISDB answer is treated as informative for novelty rather than as missing data

## Evidence

- [results] P. mutica finding: [evidence span withheld — anchor status: not_found]
- [readme] compilation and sorting of annotations: [evidence span withheld — anchor status: external_doc]
- [readme] fallback values for missing matches: [evidence span withheld — anchor status: external_doc]
- [other] validation step of workflow: [evidence span withheld — anchor status: not_found]
- [readme] GNPS threshold failure mode: [evidence span withheld — anchor status: external_doc]
