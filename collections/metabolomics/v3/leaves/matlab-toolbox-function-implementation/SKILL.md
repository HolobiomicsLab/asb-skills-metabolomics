---
name: matlab-toolbox-function-implementation
description: Use when a scientific agent must implement, run, or extend the M2S (Match
  2 Sets) MATLAB toolbox to find feature correspondence between two untargeted LC–MS
  metabolomics datasets using only RT, m/z, and optionally FI.
when_to_use_negative:
- When the input is a single peak-picked feature table and no second dataset to match
  against it — the skill requires two datasets (reference and target) for correspondence
- When the analysis goal is to annotate metabolites rather than find inter-dataset
  feature correspondence — M2S does not perform annotation; it only matches features
  by RT/MZ/FI thresholds
- When datasets are from different modalities (e.g. one NMR, one LC-MS) or have very
  different acquisition parameters, since the model assumes a shared, smoothly varying
  inter-dataset shift structure
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: M2S (Match 2 Sets)
  role: Primary MATLAB toolbox for neighbor-based inter-dataset shift modeling and
    feature correspondence between two untargeted LC-MS metabolomics datasets
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Execution environment in which the M2S functions are implemented, run, and
    extended
- name: XCMS
  role: Used to peak-pick each dataset separately, yielding the RT/MZ/FI feature tables
    that serve as inputs to M2S
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:335548d5d05a5d97ea21752467f106b1ef40dce49c9aa62f87caf391c7e94484
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# matlab-toolbox-function-implementation

## Summary

Implementing and extending the M2S MATLAB toolbox for feature correspondence between two LC–MS metabolomics datasets — including neighbor-based inter-dataset shift models (the 'cross' and 'circle' methods) — to match features using only RT, m/z, and optionally FI.

## When to use

Use when a scientific agent must implement, run, or extend the M2S (Match 2 Sets) MATLAB toolbox to find feature correspondence between two untargeted LC–MS metabolomics datasets using only RT, m/z, and optionally FI.

## When NOT to use

- When the input is a single peak-picked feature table and no second dataset to match against it — the skill requires two datasets (reference and target) for correspondence
- When the analysis goal is to annotate metabolites rather than find inter-dataset feature correspondence — M2S does not perform annotation; it only matches features by RT/MZ/FI thresholds
- When datasets are from different modalities (e.g. one NMR, one LC-MS) or have very different acquisition parameters, since the model assumes a shared, smoothly varying inter-dataset shift structure

## Inputs

- Two peak-picked feature tables (reference and target LC-MS datasets) with RT, m/z, and FI columns from XCMS
- User-defined matching thresholds for RT, MZ, and log10FI (e.g. RT 1 min, MZ 0.025 Da)
- Choice of neighbor-definition method ('cross' or 'circle')

## Outputs

- Shift curves and residuals from the 'cross' and 'circle' neighbor models
- Match assignments for both methods
- results.csv file with all above outputs

## How to apply

Load the two peak-picked feature tables (reference and target) with RT, m/z, and FI columns into MATLAB from the M2S toolbox example data. Apply initial candidate matching with user-defined RT, MZ, and log10FI thresholds (e.g. RT 1 min, MZ 0.025 Da) to collect candidate feature pairs. Run M2S's 'cross' neighbor definition, computing per-dimension k nearest neighbors with smoothing to derive a single expected shift per RTref and the corresponding residuals. Run M2S's 'circle' neighbor definition, computing k nearest neighbors by normalized Euclidean RT+MZ distance so that distinct shifts can coexist at the same RTref. Compare the two methods' shift curves and residuals and the resulting match assignments to verify the reported distinguishing behavior (cross gives one shift per RTref, circle gives multiple shifts at the same RTref). Save the shift curves, residuals, and match assignments from both methods to results.csv.

## Related tools

- **M2S (Match 2 Sets)** (Primary MATLAB toolbox for neighbor-based inter-dataset shift modeling and feature correspondence between two untargeted LC-MS metabolomics datasets) — https://github.com/rjdossan/M2S
- **MATLAB** (Execution environment in which the M2S functions are implemented, run, and extended)
- **XCMS** (Used to peak-pick each dataset separately, yielding the RT/MZ/FI feature tables that serve as inputs to M2S)

## Examples

```
MATLAB (run from the M2S repo root after cloning https://github.com/rjdossan/M2S):
run('M2S_main.m'); % or execute example scripts showing cross vs circle neighbor models on MESA/Rotterdam feature tables
```

## Evaluation signals

- Cross method yields a single robustly determined inter-dataset shift per reference feature (as in Step 2 of the paper's matching, row 2 of Figure 3)
- Circle method allows multiple shifts at the same RTref, distinct for matches at the same RTref but different MZref
- Both methods produce consistent residuals and match assignments
- Matching quality verified against manually annotated features — reported accuracy of 96.8% (585/604) in main example, 90.8% (79/87) in second example, 99.6% in simulated data (4755 features per dataset)
- Match assignments are not adversely affected by a small number of incorrect residual estimates, since residuals that exceed tight thresholds are rejected

## Limitations

- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- The decision on whether to use step 3 (tighten thresholds / delete poor matches) rests with the analyst and may inadvertently delete some correct matches
- Validation by annotated features is limited by the number of annotations in the datasets
- Most missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI

## Evidence

- [abstract] The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [readme] Matlab package to match untargeted metabolomic features of two LC-MS datasets: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
- [discussion] The 'cross' method defines k neighbors separately per dimension to produce a smoothed curve giving one inter-dataset shift for all matches sharing an RTref, while the 'circle' method uses normalized Euclidean distance over RT and MZ (not FI) so matches at the same RTref but different MZref can have distinct shifts.: [evidence span withheld — anchor status: not_found]
- [discussion] Threshold selection is subjective, guided by visual inspection: "guided by visual inspection of the distance plots resulting from the initial matching"
- [methods] Both datasets were (separately) peak-picked using XCMS, yielding a table of samples: "Both datasets were (separately) peak-picked using XCMS,13 yielding a table of samples"
- [abstract] In the main example, 4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [intro] Objective: aid integration of untargeted LC–MS metabolomics data in applications such as batch combination, discovery validation, and multicohort integration: "will aid the integration of untargeted LC–MS metabolomics data in a wide variety of applications such as batch combination, discovery validation, and multicohort integration"
