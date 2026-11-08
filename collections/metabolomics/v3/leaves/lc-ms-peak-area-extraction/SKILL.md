---
name: lc-ms-peak-area-extraction
description: Use when the analysis goal is to obtain quantitative peak-area values
  for specific metabolites (e.g. phenylacetyl-carnitine) from raw LC-MS/MS untargeted
  metabolomics files rather than relying on pre-computed feature tables.
when_to_use_negative:
- Input is already a processed feature table (e.g. from MZmine/MS-DIAL export) — skip
  reprocessing and proceed directly to statistical modeling.
- Study design requires fully validated (Level 1 MSI) structures for all metabolites
  — spectral matching alone yields Level 2 or 3 annotations.
- The instrument acquisition method lacks adequate MS/MS fragmentation for the target
  analyte — feature detection and annotation may be unreliable.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: tissueMASST
  role: Used to track molecular features (e.g. phenylacetyl-carnitine) across public
    metabolomics repositories and to contextualize their detection in tissues, complementing
    peak-area extraction from the primary data.
- name: GNPS
  role: Public spectral reference libraries used for MS/MS spectral matching to assign
    annotation levels (Level 1/2/3) to extracted features.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:3b65e0541a984bd4b0022f6801cd4d10a6bb97a9944678e3c124454da6b7a224
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lc-ms-peak-area-extraction

## Summary

Reprocess untargeted LC-MS/MS metabolomics data to extract metabolite peak areas and annotate them by MS/MS spectral matching against the GNPS reference libraries, enabling quantification of features such as phenylacetyl-carnitine (M+H, m/z 280.153) for downstream statistical modeling.

## When to use

Use when the analysis goal is to obtain quantitative peak-area values for specific metabolites (e.g. phenylacetyl-carnitine) from raw LC-MS/MS untargeted metabolomics files rather than relying on pre-computed feature tables.

## When NOT to use

- Input is already a processed feature table (e.g. from MZmine/MS-DIAL export) — skip reprocessing and proceed directly to statistical modeling.
- Study design requires fully validated (Level 1 MSI) structures for all metabolites — spectral matching alone yields Level 2 or 3 annotations.
- The instrument acquisition method lacks adequate MS/MS fragmentation for the target analyte — feature detection and annotation may be unreliable.

## Inputs

- raw untargeted LC-MS/MS metabolomics data files
- subject metadata table (e.g. age, sex, BMI)
- GNPS spectral library reference files

## Outputs

- metabolite peak-area table (features x samples)
- MS/MS spectral annotation table with annotation level (Level 1/2/3) per feature
- merged data table combining peak areas with subject metadata

## How to apply

Reprocess the untargeted LC-MS/MS data files to extract peak areas for the target metabolite(s), confirming annotation level by MS/MS spectral matching against the public GNPS reference libraries (e.g. phenylacetyl-carnitine confirmed as a Level 1 annotation via M+H, m/z 280.153). Note that spectral matching against GNPS libraries yields Level 2 or 3 annotations under the Metabolomics Standards Initiative, and cross-tissue molecular networking based on MS/MS spectral matching can be used to support feature assignment. Merge the extracted peak-area table with subject metadata (e.g. age, sex, BMI) before downstream partial correlation analysis. Validate that the feature detection is consistent across sample groups and that the metabolite of interest is detected in the expected tissues.

## Related tools

- **tissueMASST** (Used to track molecular features (e.g. phenylacetyl-carnitine) across public metabolomics repositories and to contextualize their detection in tissues, complementing peak-area extraction from the primary data.)
- **GNPS** (Public spectral reference libraries used for MS/MS spectral matching to assign annotation levels (Level 1/2/3) to extracted features.)

## Examples

```
python scripts/extract_peak_area.py --input raw_mzml/ --library GNPS/ --output peak_areas.csv
```

## Evaluation signals

- The metabolite of interest is detected with the expected precursor m/z and adduct (e.g. phenylacetyl-carnitine M+H, m/z 280.153).
- The MS/MS spectral match against GNPS libraries confirms the expected annotation level (Level 1 or Level 2/3 as appropriate).
- Peak-area table dimensions match the number of input LC-MS/MS samples and expected feature count.
- The peak-area distribution is not dominated by blank/injection artifacts and shows detectable signal across biological replicates.
- Downstream partial correlation signs (e.g. phenylacetyl-carnitine positively associated with phosphorylated tau and α-synuclein, negatively with Aβ42/Aβ40 ratio) match the reported directions.

## Limitations

- MS/MS annotation via spectral matching against GNPS reference libraries represents only Level 2 or 3 annotation under the Metabolomics Standards Initiative, not fully validated structures.
- Feature detection can be influenced by study-specific extraction methods, workflows, and instrumentation; lack of detection does not necessarily mean the compound is absent.
- Some spectral pairs may be missing from molecular networks due to poor fragmentation or missing fragments during MS/MS acquisition.
- Germ-free animals received an antibiotic cocktail during behavioral testing, which could have altered organ metabolic profiles and thus peak-area values.

## Evidence

- [results] phenylacetyl-carnitine was positively correlated with phosphorylated tau protein and α-synuclein levels and negatively correlated with the Aβ42/Aβ40 ratio: [evidence span withheld — anchor status: not_found]
- [discussion] MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative"
- [discussion] feature detection ... can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily"
- [methods] confirming a Level 1 annotation of PAC (M+H, m/z 280.153) via spectral matching against the GNPS library: [evidence span withheld — anchor status: not_found]
