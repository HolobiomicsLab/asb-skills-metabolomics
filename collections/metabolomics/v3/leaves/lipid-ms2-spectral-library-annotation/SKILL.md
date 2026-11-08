---
name: lipid-ms2-spectral-library-annotation
description: Use when annotating lipid MS/MS spectra from LC-MS/MS lipidomics data
  (Thermo .raw, SCIEX .wiff, Bruker/Agilent .d, .mzML) and benchmarking annotation
  recall against tools like MS-DIAL or LipidMatch.
when_to_use_negative:
- Input is already an annotated feature/quantification table — this skill operates
  on raw MS/MS spectra, not post-annotation matrices
- Data are non-lipid small-molecule metabolomics spectra outside LipidIN's 121-subclass
  lipid scope
- Waters raw files with LipidMatch-style comparator workflows are needed — LipidMatch
  README states it 'does not currently support Waters files', so comparator runs must
  use a different acquisition format
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidIN
  role: 'Primary annotation framework: EQ module performs the MS2 spectral library
    search; LCI module applies the three relative RT rules (ECN, IUP, ESCN); WMYn
    supplies 5th-level regenerated fingerprints'
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Baseline annotation tool and public spectral database used for comparator
    recall@Top-20 runs on identical benchmark spectra
- name: LipidSearch
  role: Comparator lipid annotation tool run on benchmark spectra under recommended
    default settings
- name: LipidMatch
  role: Comparator tool matching experimental fragments against in-silico fragmentation
    libraries of >500,000 lipid species across >60 lipid types
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: Spectral Entropy
  role: Comparator similarity-scoring tool (entropy similarity) benchmarked in recall@Top-20
    comparisons
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:ce2c8745056cdde6981cdbd93e68a8eb54e102a1a452bbb1a97281ec83e6e97b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - LipidMatch
  - LipidSearch
  - MS-DIAL
  - Spectral Entropy
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lipid-ms2-spectral-library-annotation

## Summary

Annotate LC-MS/MS lipid spectra by searching MS2 peaks against a hierarchical lipid fragmentation library (EQ module) and suppressing false positives with relative retention-time rules (LCI module), as implemented in LipidIN.

## When to use

Use when annotating lipid MS/MS spectra from LC-MS/MS lipidomics data (Thermo .raw, SCIEX .wiff, Bruker/Agilent .d, .mzML) and benchmarking annotation recall against tools like MS-DIAL or LipidMatch.

## When NOT to use

- Input is already an annotated feature/quantification table — this skill operates on raw MS/MS spectra, not post-annotation matrices
- Data are non-lipid small-molecule metabolomics spectra outside LipidIN's 121-subclass lipid scope
- Waters raw files with LipidMatch-style comparator workflows are needed — LipidMatch README states it 'does not currently support Waters files', so comparator runs must use a different acquisition format

## Inputs

- MS/MS lipid spectra from LC-MS/MS runs (.mzML, or vendor formats Thermo .raw, SCIEX .wiff, Bruker/Agilent .d)
- LipidIN hierarchical spectral library (.rda: pos_ALL.rda / neg_ALL.rda; or Python .pkl: pos_common.pkl / neg_common.pkl)
- MS-DIAL public library for baseline EQ-only searches
- Reference lipid identity list for recall computation (e.g. 2704 reported lipids in the aging atlas)

## Outputs

- Per-spectrum lipid annotations ranked by score, with chain compositions and C=C double-bond positions
- recall@Top-20 values, overall and stratified by lipid subclass (recall_comparison.csv, per-subclass plots)
- Filtered annotation table after LCI relative-RT rule suppression of false positives

## How to apply

Load a lipid MS/MS spectral dataset into the LipidIN framework together with the chosen spectral library — the R-native hierarchical library (pos_ALL.rda / neg_ALL.rda) from Zenodo record 13350719, or the Python .pkl libraries (pos_common.pkl / neg_common.pkl, 24,631,702 spectra; PB_PX_H.pkl, 2,289,913 spectra) from Zenodo record 21421866. Run the EQ module's MS2 library search, which queries chain compositions and C=C double-bond positions against the theoretical library (168.6 million fragmentation entries) using hash tables and bisection for linear-time lookup. Then apply the LCI module's three relative retention time rules (ECN, IUP and ESCN) to filter false-positive annotations; this combination reported ~5.7% estimated FDR across 8,923 lipids and >90% recall@Top-20 versus ~70% for EQ alone against the MS-DIAL library. Optionally truncate the hierarchical library to levels 1–4, or use the full 1st–5th-level library where level 5 comprises WMYn-regenerated lipid fragment fingerprints (~+20% recall). Evaluate by computing recall@Top-20 per lipid subclass (glycerophospholipids, glycerolipids, sphingolipids, fatty acids) and comparing against reported values.

## Related tools

- **LipidIN** (Primary annotation framework: EQ module performs the MS2 spectral library search; LCI module applies the three relative RT rules (ECN, IUP, ESCN); WMYn supplies 5th-level regenerated fingerprints) — https://github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Baseline annotation tool and public spectral database used for comparator recall@Top-20 runs on identical benchmark spectra)
- **LipidSearch** (Comparator lipid annotation tool run on benchmark spectra under recommended default settings)
- **LipidMatch** (Comparator tool matching experimental fragments against in-silico fragmentation libraries of >500,000 lipid species across >60 lipid types) — https://github.com/GarrettLab-UF/LipidMatch
- **Spectral Entropy** (Comparator similarity-scoring tool (entropy similarity) benchmarked in recall@Top-20 comparisons) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
R: load('pos_ALL.rda'); results <- LipidIN::eq_search(spectra, library = pos_ALL, levels = 1:5); lci_filter(results, rt_rules = c('ECN','IUP','ESCN'))
```

## Evaluation signals

- EQ+LCI on the 1st–4th-level hierarchical library should reach >90% recall@Top-20 on published benchmarks, vs ~70% for EQ alone against the MS-DIAL library
- On the aging mouse lipidome atlas, full 1st–5th-level library recall must reproduce the reported 93.64% of 2704 reported lipids, including the ~20% WMYn-driven recall gain over level-4 search
- Per-subclass ordering (glycerophospholipids, glycerolipids, sphingolipids, fatty acids) should match the published Supplementary Fig. 9a–b pattern
- Estimated false discovery rate of LCI-filtered annotations should approximate the reported 5.7% across 8923 lipids
- Library integrity checks: pos/neg .rda files load without error and the general Python library contains 24,631,702 spectra

## Limitations

- The R version of LipidIN 'is kept for reproducibility only and will no longer be updated' — new work should use the Python 'LipidIN × Agent' track
- Minimum 16 GB RAM is required to host the multi-million-spectrum libraries
- Similarity-only annotation approaches (MS-DIAL, LipidSearch, MS entropy) 'might cause feature redundancy'; RT-based score combination and deeper structure–RT relationships remain deferred future work
- Extending reverse lipidomics requires pre-training on larger, more diverse datasets

## Evidence

- [results] EQ combined with the LCI module, using the 1st- to 4th-level hierarchical library, achieved over 90% recall@Top-20, outperforming the other tested annotation tools: [evidence span withheld — anchor status: not_found]
- [abstract] developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
- [methods] R-native | `.rda` (`pos_ALL.rda`, `neg_ALL.rda`): "R-native | `.rda` (`pos_ALL.rda`, `neg_ALL.rda`)"
- [readme] The earlier **R version is kept for reproducibility only and will no longer be updated.** New users should use the Python track below.: "The earlier **R version is kept for reproducibility only and will no longer be updated.** New users should use the Python track below."
- [discussion] data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
