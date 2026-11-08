---
name: csf-biomarker-association-analysis
description: Use when a metabolite peak-area table has been extracted from LC-MS/MS
  data and needs to be tested for association with paired CSF AD biomarkers in a subject
  subset with covariate adjustment.
when_to_use_negative:
- When the metabolite of interest lacks a confirmed Level 1 annotation and only a
  Level 2 or 3 MSI assignment is available from public GNPS library matching — the
  article notes that such annotations 'represent a Level 2 or 3 annotation according
  to the Metabolomics Standards Initiative'
- When the cohort lacks paired CSF measurements or covariate data (age, sex, BMI),
  making partial correlation adjustment impossible
- When the metabolite peak areas are not yet extracted from the raw LC-MS/MS files
  — first run feature extraction and spectral annotation (e.g. via tissueMASST/GNPS)
  before applying this association skill
tools:
- name: tissueMASST
  role: Used to map the murine metabolite feature (phenylacetyl-carnitine) to human
    AD cohort datasets, bridging animal-model findings with human data prior to the
    CSF correlation analysis
- name: GNPS
  role: Public spectral reference library used to confirm the metabolite annotation
    level (Level 1 for PAC, M+H, m/z 280.153) before correlating with CSF biomarkers
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:fbfcc698a06dcd8aa8527f6d480a4cc6c9110ce4fa33788b0dd3888c50e0d53d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# csf-biomarker-association-analysis

## Summary

Quantifies partial correlations between a metabolite feature (e.g. serum phenylacetyl-carnitine) and cerebrospinal fluid Alzheimer's biomarkers (phosphorylated tau, α-synuclein, Aβ42/Aβ40 ratio) in a human cohort subset, adjusting for age, sex, and BMI.

## When to use

Use when a metabolite peak-area table has been extracted from LC-MS/MS data and needs to be tested for association with paired CSF AD biomarkers in a subject subset with covariate adjustment.

## When NOT to use

- When the metabolite of interest lacks a confirmed Level 1 annotation and only a Level 2 or 3 MSI assignment is available from public GNPS library matching — the article notes that such annotations 'represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative'
- When the cohort lacks paired CSF measurements or covariate data (age, sex, BMI), making partial correlation adjustment impossible
- When the metabolite peak areas are not yet extracted from the raw LC-MS/MS files — first run feature extraction and spectral annotation (e.g. via tissueMASST/GNPS) before applying this association skill

## Inputs

- Untargeted LC-MS/MS metabolomics files from a human cohort (e.g. the MARS cohort from the AD Knowledge Portal)
- Subject metadata: age, sex, BMI
- CSF biomarker measurements (phosphorylated tau, α-synuclein, Aβ42/Aβ40 ratio) for the subset of individuals with available CSF values (n = 77)
- Extracted metabolite peak-area table (e.g. phenylacetyl-carnitine, M+H, m/z 280.153, confirmed Level 1 via GNPS library spectral matching)

## Outputs

- CSV file (e.g. mars_pac_csf_correlations.csv) containing the merged metabolite peak-area table and the three adjusted partial correlation estimates (coefficients and p-values) for phosphorylated tau, α-synuclein, and Aβ42/Aβ40 ratio
- Directionality check: positive correlations with phosphorylated tau and α-synuclein; negative correlation with Aβ42/Aβ40 ratio

## How to apply

First, reprocess the LC-MS/MS data to extract the metabolite's peak areas and confirm annotation level — the article annotates phenylacetyl-carnitine (PAC, M+H, m/z 280.153) as a Level 1 hit via spectral matching against the GNPS library (Figure S9), while MS/MS spectra matched only through the public GNPS reference libraries remain Level 2 or 3 under the Metabolomics Standards Initiative. Second, merge the peak-area table with the subject metadata (age, sex, BMI) and the CSF biomarker measurements for the subset of individuals with available CSF values (n = 77 in the MARS cohort). Third, log-transform the metabolite intensities to stabilize variance, then compute partial correlations of the log-transformed peak areas with each CSF biomarker (phosphorylated tau, α-synuclein, and the Aβ42/Aβ40 ratio) adjusted for age, sex, and BMI — expecting positive coefficients with phosphorylated tau and α-synuclein and a negative coefficient with Aβ42/Aβ40 ratio, as shown in Figure 6G. Finally, write the merged peak-area table and the three adjusted correlation estimates (coefficients and p-values) to an output CSV such as mars_pac_csf_correlations.csv.

## Related tools

- **tissueMASST** (Used to map the murine metabolite feature (phenylacetyl-carnitine) to human AD cohort datasets, bridging animal-model findings with human data prior to the CSF correlation analysis)
- **GNPS** (Public spectral reference library used to confirm the metabolite annotation level (Level 1 for PAC, M+H, m/z 280.153) before correlating with CSF biomarkers)

## Examples

```
python mars_pac_csf_correlations.py --metabolite-table pac_peak_areas.csv --csf-biomarkers mars_csf_subset.csv --covariates age,sex,bmi --output mars_pac_csf_correlations.csv
```

## Evaluation signals

- The partial correlation sign for phosphorylated tau is positive, for α-synuclein is positive, and for Aβ42/Aβ40 ratio is negative — matching the reported directions in Figure 6G
- The analysis subset contains exactly 77 individuals with paired CSF and metabolite data
- The metabolite annotation is confirmed as Level 1 (M+H, m/z 280.153) via spectral matching against the GNPS library as reported for Figure S9
- Correlation estimates are adjusted for the three covariates: age, sex, and BMI
- Output CSV contains both the merged peak-area table and the three adjusted correlation coefficients with p-values

## Limitations

- Annotations obtained via spectral matching against the public GNPS reference libraries represent only a Level 2 or 3 annotation according to the Metabolomics Standards Initiative, not a fully validated structure
- Feature detection in reference datasets can be influenced by study-specific extraction methods, workflows, and instrumentation, so lack of detection does not necessarily mean absence of the metabolite in the sample
- Whether the conjugation of microbial phenylalanine-derived precursors into carnitines is mediated by microbes or host enzymes remains unclear, complicating biological interpretation of the correlation
- Hundreds of differentially abundant molecules remain non-annotated, so this association analysis is limited to features that can be structurally confirmed

## Evidence

- [supplementary] PAC annotation level: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative"
- [abstract] tissueMASST purpose: "Molecular feature tracking via tissueMASST, a mass spectrometry search tool we developed to bridge animal model findings with human data"
- [abstract] PAC association with AD phenotypes: "identifies microbially-modulated phenylacetyl-carnitine as positively associated with aging and cognitive impairment across human AD studies"
- [discussion] Feature detection confound in reference datasets: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily"
- [discussion] Microbial vs host mediation uncertainty: "it remains unclear if the conjugation is mediated by microbes or host enzymes"
