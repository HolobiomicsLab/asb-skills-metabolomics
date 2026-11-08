---
name: signal-to-noise-ratio-calculation
description: Use when you need to quantify detection quality (S/N) of known target
  analytes in LC-MS data after targeted peak detection, especially for low-abundance
  narcotics where the S/N reveals poor quantification.
when_to_use_negative:
- Low-abundance peaks lack a provided retention time — the article warns such peaks
  may be mis-detected and peak detection parameters must be re-evaluated first
- The goal is de novo identification of narcotics without prior drug information —
  identical m/z values from other metabolites make one-to-one identification infeasible,
  so S/N of a targeted hit cannot confirm identity
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMART 2.0
  role: Runs the targeted peak analysis module on the narcotics LC-MS data with specified
    m/z/RT tolerances and outputs the peak abundance tables used for S/N computation
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Baseline integrated untargeted metabolomics tool that SMART 2.0 extends; useful
    for the upstream preprocessing of the same data
  repo: https://github.com/YuJenL/SMART
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:29b72f6302f219079fe7532bb47b5249a832b2c182ff77e20c911b528ff45a70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# signal-to-noise-ratio-calculation

## Summary

Computes signal-to-noise (S/N) ratios for targeted LC-MS metabolomics peaks using SMART 2.0's targeted peak analysis module, exemplified by the 12-narcotics demonstration data set (e.g., amphetamine S/N of 0.19, heroin at m/z 370.16 / RT 11.42 min).

## When to use

Use when you need to quantify detection quality (S/N) of known target analytes in LC-MS data after targeted peak detection, especially for low-abundance narcotics where the S/N reveals poor quantification.

## When NOT to use

- Low-abundance peaks lack a provided retention time — the article warns such peaks may be mis-detected and peak detection parameters must be re-evaluated first
- The goal is de novo identification of narcotics without prior drug information — identical m/z values from other metabolites make one-to-one identification infeasible, so S/N of a targeted hit cannot confirm identity

## Inputs

- Narcotics (or other targeted) LC-MS data files (.raw, .d, mzXML) from the SMART demonstration data set
- Target list with known m/z and retention time values for the 12 narcotics (e.g., heroin m/z 370.16, RT 11.42 min)
- SMART 2.0 example configuration specifying m/z and RT tolerances

## Outputs

- targeted_peak_results.csv containing peak detection, abundance, and S/N ratio results
- MS1 and MS2 peak abundance tables per detected target across concentration levels
- Per-target S/N ratios (e.g., amphetamine S/N = 0.19)

## How to apply

Download the narcotics demonstration data and example configuration from the SMART GitHub repository (https://github.com/YuJenL/SMART), then run SMART 2.0's targeted peak analysis module supplying known m/z and retention time targets. Extract the MS1 and MS2 peak abundance tables from the targeted analysis output and compute the S/N ratio for each detected target peak. Verify reference detections such as heroin at m/z 370.16 with RT 11.42 min, and note that a low S/N (amphetamine = 0.19) is a red flag: the authors interpreted it as indicating that LC-MS experimental conditions and peak detection parameters need re-evaluation for that analyte. Write the combined peak detection, abundance, and S/N results to targeted_peak_results.csv.

## Related tools

- **SMART 2.0** (Runs the targeted peak analysis module on the narcotics LC-MS data with specified m/z/RT tolerances and outputs the peak abundance tables used for S/N computation) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Baseline integrated untargeted metabolomics tool that SMART 2.0 extends; useful for the upstream preprocessing of the same data) — https://github.com/YuJenL/SMART

## Evaluation signals

- Heroin is detected in MS1 across 11 concentration levels at m/z 370.16 with RT 11.42 min
- Amphetamine S/N ratio equals the reported reference value of 0.19
- MS/MS at 500 ppb shows amphetamine MS1 peaks producing identical fragments at m/z 91, 119, and 136
- Output file targeted_peak_results.csv contains detection, abundance, and S/N columns for all detected targets
- Heroin and nimetazepam absence from HMDB-linked peak identification does not block targeted detection, but is a known caveat to document

## Limitations

- SMART 2.0 cannot detect heroin and nimetazepam via HMDB-based peak identification because HMDB does not encompass them
- A very low S/N (amphetamine 0.19) suggests inappropriate peak detection parameters or LC-MS experimental conditions; re-evaluation is required to improve quantification accuracy
- One-to-one identification of narcotics without prior drug information is not feasible due to other metabolites with identical m/z values
- Consistency in LC-MS experimental conditions is crucial for reliable calibration and quantification but is not guaranteed; increasing replicates is recommended

## Evidence

- [discussion] the S/N ratio of amphetamine was found to be only 0.19: "the S/N ratio of amphetamine was found to be only 0.19"
- [discussion] MS/MS analysis at 500 ppb, which revealed that both MS1 peaks produced identical fragmentation patterns (m/z 91, 119, and 136): "MS/MS analysis at 500 ppb, which revealed that both MS1 peaks produced identical fragmentation patterns (m/z 91, 119, and 136)"
- [readme] Implement peak analysis for both untargeted and targeted data and peak annotation.: "Implement peak analysis for both untargeted and targeted data and peak annotation."
- [abstract] This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities.: "This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities."
- [discussion] Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
