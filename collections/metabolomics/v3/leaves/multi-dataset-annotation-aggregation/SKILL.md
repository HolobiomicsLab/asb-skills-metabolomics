---
name: multi-dataset-annotation-aggregation
description: Use when you have raw MS/MS feature tables from multiple LC-MS/MS lipidomics
  datasets and need standardized lipid annotations with estimated FDR aggregated across
  datasets.
when_to_use_negative:
- Input is already a curated, filtered annotation table — re-running EQ+LCI library
  search and ECN filtering would duplicate existing work
- Target analytes are non-lipid metabolites — the LCI retention-time rules and LipidIN
  library are lipid-specific (121 subclasses)
- The task requires double-bond localization via Paternò–Büchi derivatization — use
  the separate PB spectral library workflow, not the general EQ+LCI pipeline
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Framework providing the EQ querying, LCI retention-time filtering, and WMYn
    fingerprint-regeneration modules used for annotation and FDR estimation
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: LipidIN EQ module
  role: Performs sub-second MS2 library search against the 24.6-million-spectrum theoretical
    library using hash tables and bisection methods
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: LipidIN LCI module
  role: Applies three relative retention time rules with BFS heuristic to predict
    lipid subclass and remove false positives
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: LipidIN WMYn module
  role: Regenerates lipid fragment fingerprints as the 5th-level library to boost
    recall (~20%) and confirm low-confidence annotations
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Baseline peak-picking/annotation tool whose public database results are compared
    against EQ outputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:c037a6b9649d0721571b27e9ce3fc0c333b20910b16a6918cc52f67da299a4cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - LipidIN EQ module
  - LipidIN LCI module
  - LipidIN WMYn module
  - MS-DIAL
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-dataset-annotation-aggregation

## Summary

Runs the LipidIN EQ+LCI annotation workflow across four LC-MS/MS datasets (RBL-2H3 cells, lipid mixture, human sera, zebrafish tissue), computes per-dataset false discovery rates, and aggregates them to a reported average FDR of 5.69% at the cutoff threshold of 2.4.

## When to use

Use when you have raw MS/MS feature tables from multiple LC-MS/MS lipidomics datasets and need standardized lipid annotations with estimated FDR aggregated across datasets.

## When NOT to use

- Input is already a curated, filtered annotation table — re-running EQ+LCI library search and ECN filtering would duplicate existing work
- Target analytes are non-lipid metabolites — the LCI retention-time rules and LipidIN library are lipid-specific (121 subclasses)
- The task requires double-bond localization via Paternò–Büchi derivatization — use the separate PB spectral library workflow, not the general EQ+LCI pipeline

## Inputs

- Raw MS/MS feature tables from four datasets: RBL-2H3 cells, lipid mixture, human sera, zebrafish tissues
- LipidIN theoretical spectral library (.rda: pos_ALL.rda, neg_ALL.rda; or .pkl: pos_common.pkl, neg_common.pkl) from Zenodo record 21421866
- WMYn regenerated 5th-level fingerprint library (optional, for recall boosting)

## Outputs

- Per-dataset annotation tables with lipid subclass predictions
- Aggregated annotation_report.csv containing per-dataset and average FDR summary
- Unique lipid counts per dataset (e.g., 471 unique lipids in the mixture dataset)

## How to apply

Load raw MS/MS feature tables from each dataset and the LipidIN theoretical spectral libraries (pos_ALL.rda / neg_ALL.rda for R, or pos_common.pkl / neg_common.pkl for Python, downloaded from Zenodo). Run the EQ (expeditious querying) module to perform MS2 library search against the theoretical library covering chain compositions and C=C double-bond positions, exploiting hash tables and bisection methods for sub-second querying of the 24.6-million-spectrum library. Apply the LCI module's three relative retention time rules with a best-first search (BFS) heuristic to predict lipid subclass and remove false positive annotations, then enforce strict ECN-rule and feature-peak criteria. Optionally use the WMYn-regenerated 5th-level fingerprint library to boost recall (~20%) and confirm low-confidence lipids. Compute the estimated FDR per dataset, aggregate across datasets, and save per-dataset annotation tables plus an FDR summary as annotation_report.csv; correctness is judged against the reported 5.69% average FDR, 8923 annotated lipids, and 471 unique lipids in the mixture dataset (Fig. 3e-h, Supplementary Data 6-7).

## Related tools

- **LipidIN** (Framework providing the EQ querying, LCI retention-time filtering, and WMYn fingerprint-regeneration modules used for annotation and FDR estimation) — https://github.com/LinShuhaiLAB/LipidIN
- **LipidIN EQ module** (Performs sub-second MS2 library search against the 24.6-million-spectrum theoretical library using hash tables and bisection methods) — https://github.com/LinShuhaiLAB/LipidIN
- **LipidIN LCI module** (Applies three relative retention time rules with BFS heuristic to predict lipid subclass and remove false positives) — https://github.com/LinShuhaiLAB/LipidIN
- **LipidIN WMYn module** (Regenerates lipid fragment fingerprints as the 5th-level library to boost recall (~20%) and confirm low-confidence annotations) — https://github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Baseline peak-picking/annotation tool whose public database results are compared against EQ outputs)

## Examples

```
python run_lipidin.py --datasets RBL-2H3 mixture human_sera zebrafish --library pos_common.pkl neg_common.pkl --strict --cutoff 2.4 --output annotation_report.csv
```

## Evaluation signals

- Average estimated FDR across the four datasets should be ~5.69% at the cutoff threshold of 2.4 under strict criteria
- Total annotated lipids across datasets should approximate 8923, with 471 unique lipids in the mixture dataset
- annotation_report.csv should contain both per-dataset and aggregated FDR values matching Supplementary Data 6-7
- Annotations surviving strict ECN-rule and feature-peak filtering should show removal of inconsistent subclass predictions
- QC of quantified features (CV / PCA / PLS-DA) should be applied downstream per the article's annotation → quantification → QC workflow

## Limitations

- Estimated FDR (~5.7%) is coverage-dependent; extending recall via pre-training on larger datasets is stated as future work
- EQ querying must be paired with MS-DIAL's public database carefully — similarity algorithms like MS-DIAL, LipidSearch, and MS entropy may cause feature redundancy
- Deeper relationship between metabolites and relative RT is uninvestigated, limiting RT-rule generalization beyond the tested lipid subclasses

## Evidence

- [results] EQ plus LCI achieved an average FDR of 5.69% at a cutoff threshold of 2.4 under strict criteria, annotating 8923 lipids in four datasets: [evidence span withheld — anchor status: not_found]
- [abstract] LCI three RRT rules with 5.7% FDR: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
- [methods] Zenodo spectral library download instructions: "Download the spectral libraries from **[Zenodo → https://zenodo.org/records/21421866](https://zenodo.org/records/21421866)**"
- [discussion] Regeneration of lipid fingerprints as 5th-level library: "regeneration of lipid fingerprints as the 5th-level library for high-confidence and high-coverage annotation"
- [readme] README workflow metrics table: "Annotation false discovery rate | 5.7 % (estimated), across 8,923 lipids"
