---
name: msms-library-matching
description: Use when raw LC-MS/MS feature tables from lipidomics experiments (e.g.
  Thermo .raw, SCIEX .wiff, .mzML inputs) need MS2 library-search annotation against
  a theoretical spectral library covering lipid chain compositions and C=C double-bond
  positions.
when_to_use_negative:
- Input lacks MS/MS fragmentation data (MS1-only features) — library search requires
  MS2 spectra to match against theoretical fragmentation entries
- The target analytes are small molecules outside lipid classes — the LipidIN library
  covers 121 lipid subclasses and glycerophospholipid/lysophospholipid PB subclasses,
  not general metabolomes
- Only oxidized lipids are targeted without PB derivatization data — oxidized-lipid
  spectra reside in a separate Zenodo record (13735639) and PB localization requires
  Paternò–Büchi derivatized samples
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: LipidIN
  role: Framework providing the theoretical spectral library (168.6 million fragmentation
    entries) and the EQ querying module for MS2 library search
  repo: github.com/LinShuhaiLAB/LipidIN
- name: LipidIN EQ module
  role: Expeditious querying module performing hash-table/bisection MS2 library search
    with linear time complexity for sub-second annotation
  repo: github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Baseline similarity-based annotation tool compared against LipidIN; combined
    EQ + MS-DIAL public database outperformed MS-DIAL alone
- name: Flash Entropy Search
  role: Alternative spectral library search tool; LipidIN's EQ linear-time hashing
    outperforms its squared time complexity
  repo: https://github.com/YuanyueLi/EntropySearch/releases
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:d94e508840b26ee1767968cd9a9517f890dbe4ba6ad9a71f96ca80180dd2c2a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flash Entropy Search
  - LipidIN
  - LipidIN EQ module
  - LipidIN LCI module
  - LipidIN WMYn module
  - MS-DIAL
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# msms-library-matching

## Summary

Matches experimental LC-MS/MS feature spectra against a large theoretical lipid MS/MS library (chain compositions + C=C double-bond positions) to annotate lipids, as implemented in LipidIN's EQ module with hash-table/bisection querying for sub-second searches.

## When to use

Use when raw LC-MS/MS feature tables from lipidomics experiments (e.g. Thermo .raw, SCIEX .wiff, .mzML inputs) need MS2 library-search annotation against a theoretical spectral library covering lipid chain compositions and C=C double-bond positions.

## When NOT to use

- Input lacks MS/MS fragmentation data (MS1-only features) — library search requires MS2 spectra to match against theoretical fragmentation entries
- The target analytes are small molecules outside lipid classes — the LipidIN library covers 121 lipid subclasses and glycerophospholipid/lysophospholipid PB subclasses, not general metabolomes
- Only oxidized lipids are targeted without PB derivatization data — oxidized-lipid spectra reside in a separate Zenodo record (13735639) and PB localization requires Paternò–Büchi derivatized samples

## Inputs

- Raw MS/MS feature tables from LC-MS/MS lipidomics runs (Thermo .raw, SCIEX .wiff, Bruker/Agilent .d, .mzML)
- LipidIN theoretical spectral library in R-native format (pos_ALL.rda, neg_ALL.rda)
- LipidIN general theoretical spectral library in Python .pkl format (pos_common.pkl, neg_common.pkl; 24,631,702 spectra)
- LipidIN PB spectral library for Paternò–Büchi double-bond localization (PB_PX_H.pkl, common_PX_CH3COO.pkl + MS1 index; 2,289,913 spectra)

## Outputs

- Per-feature lipid annotation table mapping MS2 spectra to lipid species with chain compositions and C=C double-bond positions
- Annotation report with estimated false discovery rate per dataset (e.g. annotation_report.csv matching the reported 5.69% average FDR and 471 unique lipids in the mixture dataset)

## How to apply

Load the raw MS/MS feature tables and the LipidIN theoretical spectral library — either the R-native files (pos_ALL.rda, neg_ALL.rda) or the Python .pkl libraries (pos_common.pkl, neg_common.pkl; 24,631,702 spectra) downloaded from Zenodo record 21421866. Run the EQ (expeditious querying) module to perform MS2 library search against the theoretical library covering all chain compositions and C=C double-bond positions for each feature. EQ uses data-structure optimization based on linear time complexity via hash tables and bisection methods, enabling sub-second querying across the 24.6-million-spectrum library (~70 billion spectral comparisons in <1 s). Note that LipidIN's linear-time hashing outperforms the squared-time complexity of flash entropy search, and similarity-algorithm approaches (MS-DIAL, LipidSearch, MS entropy) can cause feature redundancy. Filter the resulting annotations downstream (e.g. with LCI retention-time rules and ECN-rule criteria) to control the false discovery rate, which reached an estimated 5.7% across 8,923 lipids.

## Related tools

- **LipidIN** (Framework providing the theoretical spectral library (168.6 million fragmentation entries) and the EQ querying module for MS2 library search) — github.com/LinShuhaiLAB/LipidIN
- **LipidIN EQ module** (Expeditious querying module performing hash-table/bisection MS2 library search with linear time complexity for sub-second annotation) — github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Baseline similarity-based annotation tool compared against LipidIN; combined EQ + MS-DIAL public database outperformed MS-DIAL alone)
- **Flash Entropy Search** (Alternative spectral library search tool; LipidIN's EQ linear-time hashing outperforms its squared time complexity) — https://github.com/YuanyueLi/EntropySearch/releases

## Evaluation signals

- Estimated false discovery rate across datasets falls near the reported ~5.7% (average FDR of 5.69% at cutoff threshold 2.4 under strict criteria)
- Annotation counts are consistent with reported scale: 8,923 lipids annotated across RBL-2H3 cells, mixture dataset, human sera, and zebrafish tissues; 471 unique lipids in the mixture dataset
- Query throughput reaches billions of comparisons per second — e.g. ~70 billion spectral comparisons complete in <1 s on the 24.6-million-spectrum general library
- Annotated lipid coverage spans the library's 121 subclasses, and subclass×adduct combinations align with the reported ~66 (pos) / ~95 (neg) subclasses and 91 / 108 combinations
- Output annotation_report.csv reproduces the per-dataset and aggregated FDR summary consistent with Fig. 3e-h and Supplementary Data 6-7

## Limitations

- Minimum 16 GB RAM required; R version is kept for reproducibility only and will no longer be updated — new users should use the Python track
- Similarity-based annotation algorithms (MS-DIAL, LipidSearch, MS entropy) may cause feature redundancy compared to LipidIN's EQ approach
- PB double-bond localization requires Paternò–Büchi derivatization and a separate PB library; oxidized-lipid spectra are held in an additional Zenodo record

## Evidence

- [methods] Run the EQ (expeditious querying) module to perform MS2 library search against the theoretical library (chain compositions + C=C double-bond positions) for each feature, using hash tables and bisection methods for sub-second querying: [evidence span withheld — anchor status: not_found]
- [results] EQ+LCI achieved an average FDR of 5.69% at a cutoff threshold of 2.4 under strict criteria, annotating 8923 lipids in four datasets: [evidence span withheld — anchor status: not_found]
- [discussion] data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [readme] Annotation — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions): "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)."
- [discussion] Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy.: "Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy."
