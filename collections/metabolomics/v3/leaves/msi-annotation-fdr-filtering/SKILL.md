---
name: msi-annotation-fdr-filtering
description: Use when selecting usable public METASPACE imaging MS datasets for annotation
  or benchmarking, or when comparing METASPACE-ML vs rule-based MSM annotations against
  MS/MS-validated LC-MS/MS ground truth.
when_to_use_negative:
- Input data are profile-mode rather than centroided, since the new METASPACE-ML features
  (absolute and relative m/z error) were developed for centroid data
- The analysis requires instrument/technology types absent from the training data,
  e.g. QTOF datasets, which are underrepresented due to lack of publicly available
  data for reliable model evaluation
- The goal is annotation of non-imaging bulk LC-MS/MS data rather than imaging MS
  datasets in the METASPACE context
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0091
tools:
- name: METASPACE
  role: Platform and knowledge base from which public imaging MS datasets, annotations,
    and the GraphQL/python-client interfaces are accessed
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE-ML
  role: Machine-learning FDR-controlled metabolite annotation engine applied to the
    filtered datasets and compared against rule-based MSM
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:a5fcf82baac9fb6731dcc6a01d1376354f3d7e7b44dcd3ecba73b1f278f87edf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# msi-annotation-fdr-filtering

## Summary

Quality-filter and FDR-control metabolite annotations for imaging mass spectrometry datasets using the METASPACE/METASPACE-ML pipeline, applying dataset-level heuristics (proportion_overlap, annotation count, peak count) and FDR-controlled scoring instead of the rule-based MSM approach.

## When to use

Use when selecting usable public METASPACE imaging MS datasets for annotation or benchmarking, or when comparing METASPACE-ML vs rule-based MSM annotations against MS/MS-validated LC-MS/MS ground truth.

## When NOT to use

- Input data are profile-mode rather than centroided, since the new METASPACE-ML features (absolute and relative m/z error) were developed for centroid data
- The analysis requires instrument/technology types absent from the training data, e.g. QTOF datasets, which are underrepresented due to lack of publicly available data for reliable model evaluation
- The goal is annotation of non-imaging bulk LC-MS/MS data rather than imaging MS datasets in the METASPACE context

## Inputs

- Metadata and annotation records for 9251 public METASPACE datasets from the METASPACE knowledge base
- Formula+adduct annotations at FDR < 10% from METASPACE-ML and MSM (Supplementary Data 8, 10 mouse brain MALDI-FTICR imaging datasets)
- MS/MS-validated bulk LC-MS/MS metabolite identifications as reference ground truth (Supplementary Data 9)

## Outputs

- retained_datasets.csv with the identifiers of datasets passing all quality filters (expected 7713 of 9251)
- validation_metrics.csv with per-approach median MAP and overall/exclusive-ion TPR values (MAP 0.57 ML vs 0.43 MSM; overall TPR 91%; exclusive-ion TPR 72% vs 11%)

## How to apply

For dataset selection, download metadata and annotation records from the METASPACE knowledge base, then per dataset select the top 50 highest-intensity pixels and compute the proportion_overlap score with a 3 ppm m/z tolerance; exclude datasets with proportion_overlap > 0.5, fewer than 10 annotations at FDR 20%, or median peak count > 50,000, and verify the retained count against the expected ~7713 of 9251 datasets (downloaded September 2023). For annotation comparison, load formula+adduct annotations at FDR < 10% (e.g. Supplementary Data 8) for METASPACE-ML and MSM across MALDI-FTICR imaging datasets, intersect them with MS/MS-validated bulk LC-MS/MS identifications (Supplementary Data 9) using set-based pandas operations, then compute median annotation power (MAP) and true positive rates overall and for exclusively captured ions (expected: 91% overall TPR for METASPACE-ML; 72% vs 11% for exclusive ions). Rationale: the rule-based MSM scoring assigns equal weights to features and lacks adaptability, so ML-based FDR-controlled scoring recovers more low-intensity, biologically relevant metabolites.

## Related tools

- **METASPACE** (Platform and knowledge base from which public imaging MS datasets, annotations, and the GraphQL/python-client interfaces are accessed) — https://github.com/metaspace2020/metaspace
- **METASPACE-ML** (Machine-learning FDR-controlled metabolite annotation engine applied to the filtered datasets and compared against rule-based MSM) — https://github.com/metaspace2020/metaspace

## Examples

```
python filter_datasets.py --input metaspace_metadata.csv --output retained_datasets.csv --max-proportion-overlap 0.5 --min-annotations 10 --fdr 0.20 --max-median-peaks 50000
```

## Evaluation signals

- retained_datasets.csv exists and contains exactly 7713 dataset identifiers out of 9251 downloaded datasets
- Reproduced overall LC-MS/MS validation TPR of 91% for METASPACE-ML annotations
- Reproduced exclusive-ion TPR of 72% (METASPACE-ML) vs 11% (rule-based MSM)
- Reproduced median METASPACE annotation power across the 10 datasets: 0.57 (METASPACE-ML) vs 0.43 (MSM)
- All filtering decisions traceable to the three thresholds: proportion_overlap > 0.5, < 10 annotations at FDR 20%, median peak count > 50,000

## Limitations

- QTOF-based datasets are underrepresented in the model due to the lack of publicly available data needed for statistically reliable model evaluation
- Training data selection is biased towards public datasets represented in METASPACE (from 47 labs)
- Less common data types, e.g. single-cell metabolomics from cultured cells or industrial lab data, are underrepresented and warrant further evaluation
- MSM's sensitivity to the rho_spatial score can cause it to capture annotations missed by METASPACE-ML, so exclusive-ion results differ per approach

## Evidence

- [abstract] METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites.: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites"
- [intro] a limitation of METASPACE is its rule-based scoring system, namely Metabolite Signal Matching (MSM), which assigns equal weights to the features and lacks adaptability to data variations: "a limitation of METASPACE is its rule-based scoring system, namely Metabolite Signal Matching (MSM), which assigns equal weights to the features and lacks adaptability to data variations"
- [intro] We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity.: "We demonstrate that METASPACE-ML outperforms the traditional MSM rule-based approach by delivering more annotations, especially at low FDR thresholds and for ions of low intensity."
- [readme] hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase of the metabolites from thousands of public datasets: "hosts an engine for
 metabolite annotation of imaging mass spectrometry data as well as a
 spatial metabolite knowledgebase of the metabolites from thousands of
 public datasets provided by the"
- [discussion] CoreMetabolome, which resulted in higher MAP scores compared to more general databases: "CoreMetabolome, which resulted in higher MAP scores compared to more general databases"
