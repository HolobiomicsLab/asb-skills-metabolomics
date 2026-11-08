---
name: openswath-targeted-proteomics-quantification
description: Use when you have SWATH-MS (DIA) LC-MS/MS maps (e.g. plasma or cell-lysed
  twin-cohort runs) and an assay/transition library, and you need targeted per-peptide/per-protein
  quantification plus cross-run comparability.
when_to_use_negative:
- Data are already reduced to a feature/abundance table, so targeted extraction and
  RT alignment would be redundant.
- The acquisition is classic DDA with a single-shot database search and no DIA/SWATH
  windowing, so transition-library-guided extraction does not apply.
- No assay/transition library is available and only global (untargeted) peptide detection
  is needed rather than targeted quantification.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: OpenMS
  role: framework supplying the processing chain (file conversion, centroiding, database
    searching, feature detection, RT alignment, cross-run feature linking, protein
    inference, statistical error control) used to turn DIA maps into a quantified
    protein table
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS 2.0
  role: the specific released version cited by the article when the OpenSWATH module
    was applied to the twin-cohort plasma maps
  repo: https://github.com/OpenMS/OpenMS
- name: Python
  role: used to fit the downstream per-protein linear mixed-effects variance-component
    model (heritable, environmental, longitudinal fractions)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:6d18932ee3ead85f543d33b53768bc3506bae607a3c03d81bc89c4c8d2f7bf2c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS
  - OpenMS 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# openswath-targeted-proteomics-quantification

## Summary

Quantify peptides and proteins from SWATH-MS data-interdependent-acquisition (DIA) LC-MS/MS maps by running the OpenMS/OpenSWATH targeted extraction pipeline — transition-library-guided peak picking, decoy-based scoring, RT alignment, and cross-run feature linking — to build a per-protein abundance matrix ready for downstream statistics.

## When to use

Use when you have SWATH-MS (DIA) LC-MS/MS maps (e.g. plasma or cell-lysed twin-cohort runs) and an assay/transition library, and you need targeted per-peptide/per-protein quantification plus cross-run comparability.

## When NOT to use

- Data are already reduced to a feature/abundance table, so targeted extraction and RT alignment would be redundant.
- The acquisition is classic DDA with a single-shot database search and no DIA/SWATH windowing, so transition-library-guided extraction does not apply.
- No assay/transition library is available and only global (untargeted) peptide detection is needed rather than targeted quantification.

## Inputs

- SWATH-MS DIA LC-MS/MS maps in mzML or equivalent open format
- assay/transition library (targeted peptide transition list with decoys)
- protein database for database search (used in the OpenMS processing chain)

## Outputs

- per-peptide quantity matrix (RT-aligned, cross-run linked)
- per-protein abundance matrix (after protein inference and statistical error control)
- optional downstream decomposition, e.g. variance_components.csv with heritable / environmental / longitudinal fractions per protein

## How to apply

Load the DIA maps (mzML) and run the OpenMS 2.0 processing chain — file conversion, centroiding, database searching, and feature detection — before OpenSWATH targeted extraction. Feed an assay/transition library into OpenSWATH to extract peptide transitions from each map, scoring targets against decoys for FDR control. Perform retention-time alignment across all runs, then cross-run feature linking so the same peptide is consistently identified in every map. Aggregate linked peptide quantities via protein inference and apply statistical error control to produce a per-protein abundance matrix, which can then feed downstream models such as linear mixed-effects variance-component decomposition (heritable vs. environmental vs. longitudinal fractions, summing to ~1 per protein).

## Related tools

- **OpenMS** (framework supplying the processing chain (file conversion, centroiding, database searching, feature detection, RT alignment, cross-run feature linking, protein inference, statistical error control) used to turn DIA maps into a quantified protein table) — https://github.com/OpenMS/OpenMS
- **OpenMS 2.0** (the specific released version cited by the article when the OpenSWATH module was applied to the twin-cohort plasma maps) — https://github.com/OpenMS/OpenMS
- **Python** (used to fit the downstream per-protein linear mixed-effects variance-component model (heritable, environmental, longitudinal fractions))

## Examples

```
OpenSWATH -in plasma_map.mzML -traML assays.TraML -tr_ichrom_models lib.rtmodel -out features.mzML -min_rsq 0.95
```

## Evaluation signals

- The three downstream variance components (heritable, environmental, longitudinal) sum to approximately 1 for every protein.
- Per-protein quantification counts exceed the paper's reported >300 plasma proteins for the twin-cohort maps.
- RT alignment produces a consistent peptide elution order across all runs — the same peptide maps to comparable retention times in every twin-pair file.
- Cross-run feature linking yields one linked feature per peptide per run (no missing values for peptides that should be detectable).
- OpenSWATH decoy-based scoring keeps FDR-controlled identifications at the expected level (decoy/target ratio matches the chosen threshold).

## Limitations

- Quality depends on the assay/transition library — a sparse or poorly curated library will miss peptides.
- DIA maps must be centroided and pre-processed (file conversion, centroiding, database searching, feature detection) before OpenSWATH extraction; raw vendor files are not directly consumable.
- Variance-component decomposition assumes the twin-cohort design (heritable vs. environmental vs. longitudinal) and does not generalize to non-paired cohorts.

## Evidence

- [full_text] Using the OpenSWATH module developed with OpenMS, Liu et al. quantified over 300 plasma proteins and decomposed the observed variance for each protein into heritable, environmental (common and individual) and longitudinal components.: [evidence span withheld — anchor status: not_found]
- [readme] OpenMS offers analyses for various quantitation protocols, including label-free quantitation, SILAC, iTRAQ, TMT, SRM, SWATH, etc.: [evidence span withheld — anchor status: external_doc]
