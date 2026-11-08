---
name: narcotics-lcms-data-analysis
description: Use when analyzing LC-MS data for a panel of known narcotic or other
  targeted compounds with known m/z and retention times, and per-target detection
  plus signal-to-noise quantification is required.
when_to_use_negative:
- Input is already a processed peak/feature table rather than raw LC-MS data needing
  targeted peak detection
- Goal is untargeted metabolomics peak picking rather than detection of compounds
  with known m/z and RT targets
- One-to-one identification of narcotics is required without prior drug information,
  since other metabolites can share identical m/z values
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMART 2.0
  role: Runs the targeted peak analysis module on the narcotics LC-MS data using supplied
    m/z and RT targets, and produces MS1/MS2 abundance tables and S/N ratios
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Baseline untargeted metabolomics tool that SMART 2.0 extends; the narcotics
    workflow builds on its data preprocessing and peak analysis capabilities
  repo: https://github.com/YuJenL/SMART
- name: MassBank
  role: m/z reference database used by SMART's postanalysis peak identification module
    for matching detected peaks
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:65e9e67d1fa58338af698860ea6be50b6c1494fa11fcec7926993609bd4d283e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - MassBank
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# narcotics-lcms-data-analysis

## Summary

Reproduce targeted peak analysis of a 12-narcotics LC-MS demonstration data set using SMART 2.0, detecting targeted compounds at specified m/z and retention-time tolerances and computing per-peak S/N ratios (e.g., heroin at m/z 370.16 / RT 11.42 min; amphetamine S/N 0.19).

## When to use

Use when analyzing LC-MS data for a panel of known narcotic or other targeted compounds with known m/z and retention times, and per-target detection plus signal-to-noise quantification is required.

## When NOT to use

- Input is already a processed peak/feature table rather than raw LC-MS data needing targeted peak detection
- Goal is untargeted metabolomics peak picking rather than detection of compounds with known m/z and RT targets
- One-to-one identification of narcotics is required without prior drug information, since other metabolites can share identical m/z values

## Inputs

- narcotics demonstration LC-MS dataset from the SMART GitHub repository (12 targeted narcotics across multiple concentration levels)
- example targeted-analysis configuration with known m/z and RT targets and tolerances
- mzXML or other raw LC-MS data formats supported by SMART's Data Import module

## Outputs

- detected MS1 and MS2 peak abundance tables for the targeted narcotics
- per-target S/N ratio table (e.g., amphetamine S/N = 0.19)
- combined targeted_peak_results.csv with peak detection, abundance, and S/N results

## How to apply

Download the narcotics demonstration data set and example configuration from the SMART GitHub repository and run SMART 2.0's targeted metabolomics analysis module, supplying the known m/z and retention-time targets so peaks are matched within the specified tolerances. Extract the detected MS1 and MS2 peak abundance tables from the targeted analysis output and compute the signal-to-noise (S/N) ratio for each detected target peak. Validate against the article's reference values: heroin detected in MS1 across 11 concentration levels at m/z 370.16 with RT 11.42 min, and an amphetamine S/N ratio of 0.19. Note that low S/N values (e.g., amphetamine 0.19) may indicate inappropriate peak detection parameters or LC-MS condition variability, and that SMART 2.0 cannot detect heroin and nimetazepam via peak identification because HMDB does not encompass them.

## Related tools

- **SMART 2.0** (Runs the targeted peak analysis module on the narcotics LC-MS data using supplied m/z and RT targets, and produces MS1/MS2 abundance tables and S/N ratios) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Baseline untargeted metabolomics tool that SMART 2.0 extends; the narcotics workflow builds on its data preprocessing and peak analysis capabilities) — https://github.com/YuJenL/SMART
- **MassBank** (m/z reference database used by SMART's postanalysis peak identification module for matching detected peaks) — https://github.com/MassBank/MassBank-data

## Evaluation signals

- Heroin is detected in MS1 across 11 concentration levels at m/z 370.16 with an associated RT of 11.42 min
- Computed amphetamine S/N ratio equals the reported value of 0.19
- MS/MS at 500 ppb shows both amphetamine MS1 peaks produce identical fragments at m/z 91, 119, and 136
- targeted_peak_results.csv contains peak detection, abundance, and S/N results for the targeted narcotics
- Peak identification via HMDB/MassBank matching fails for heroin and nimetazepam, consistent with the article's note that HMDB does not encompass them

## Limitations

- SMART 2.0 cannot detect heroin and nimetazepam via peak identification because HMDB does not include them
- Low-abundance peaks may be mis-detected when RT is not provided; amphetamine's S/N of 0.19 suggests peak detection parameters and LC-MS experimental conditions may need re-evaluation
- One-to-one identification of narcotics without prior drug information is not feasible because other metabolites can share identical m/z values; MS/MS fragment confirmation (m/z 91, 119, 136 for amphetamine) is needed
- Consistency of LC-MS experimental conditions is required for reliable calibration curves and accurate quantification, but variations were observed; more experimental replicates are recommended

## Evidence

- [abstract] With SMART 2.0, we enhanced SMART 1.0 by introducing new analytical modules in targeted metabolomics analysis: "With SMART 2.0, we enhanced SMART 1.0 by introducing new analytical modules in targeted metabolomics analysis"
- [discussion] amphetamine S/N 0.19: "the S/N ratio of amphetamine was found to be only 0.19"
- [discussion] identical amphetamine fragments: "MS/MS analysis at 500 ppb, which revealed that both MS1 peaks produced identical fragmentation patterns (m/z 91, 119, and 136)"
- [discussion] HMDB missing heroin/nimetazepam: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
- [readme] targeted peak analysis module in README: "Peak Analysis and annotation: Implement peak analysis for both untargeted and targeted data and peak annotation."
- [abstract] SMART repo availability: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
