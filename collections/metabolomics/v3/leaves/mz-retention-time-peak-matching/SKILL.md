---
name: mz-retention-time-peak-matching
description: Use when LC-MS metabolomics data (mzXML, .raw, .d) must be screened for
  a predefined target list of known compounds using their m/z and retention times
  with explicit tolerances, and S/N ratios per target are required.
when_to_use_negative:
- When the goal is untargeted peak detection across all features rather than matching
  a predefined target list (use SMART's untargeted module instead)
- When a target is absent from annotation databases such as HMDB (e.g., heroin and
  nimetazepam), where identification rather than matching is expected to fail
- When data are not LC-MS peak data (e.g., gene expression matrices, pathway p-values),
  since the m/z/RT matching logic does not apply
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMART 2.0
  role: Runs the targeted peak analysis module that matches supplied m/z/RT targets
    with tolerances against LC-MS data and outputs MS1/MS2 abundance and S/N tables
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Base R tool providing the integrated metabolomics analysis flow (import, visualization,
    peak analysis) that SMART 2.0 extends with the targeted module
  repo: https://github.com/YuJenL/SMART
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:63a6c1741127624c099d9231ef392d401116fed967222aa74876243cca580362
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# mz-retention-time-peak-matching

## Summary

Targeted LC-MS peak detection that matches detected features against a list of known m/z values and retention times using user-specified m/z and RT tolerances, as implemented in the SMART 2.0 targeted peak analysis module for the 12-narcotics demonstration data.

## When to use

Use when LC-MS metabolomics data (mzXML, .raw, .d) must be screened for a predefined target list of known compounds using their m/z and retention times with explicit tolerances, and S/N ratios per target are required.

## When NOT to use

- When the goal is untargeted peak detection across all features rather than matching a predefined target list (use SMART's untargeted module instead)
- When a target is absent from annotation databases such as HMDB (e.g., heroin and nimetazepam), where identification rather than matching is expected to fail
- When data are not LC-MS peak data (e.g., gene expression matrices, pathway p-values), since the m/z/RT matching logic does not apply

## Inputs

- LC-MS raw or converted data files (.raw, .d, mzXML)
- Target list of known m/z values and retention times
- m/z and RT tolerance specifications for peak matching

## Outputs

- Detected MS1 peak abundance table per target
- Detected MS2 peak abundance table per target
- Per-target signal-to-noise (S/N) ratios (e.g., targeted_peak_results.csv)

## How to apply

Build a target table of known m/z and retention time values (e.g., narcotics targets such as heroin at m/z 370.16 / RT 11.42 min) and supply it with specified m/z and RT tolerances to the SMART 2.0 targeted peak analysis module, which reads LC-MS files (.raw, .d, mzXML). The module performs targeted peak detection in MS1 (and MS2) within the tolerance windows and extracts per-target peak abundance tables. Compute the signal-to-noise (S/N) ratio for each detected target; treat very low S/N (e.g., amphetamine S/N = 0.19 in the narcotics data) as a flag that detection parameters or LC-MS conditions need re-evaluation. Because other metabolites can share identical m/z, always use the RT window to disambiguate targets rather than m/z alone. Validate by confirming that reference targets (heroin at m/z 370.16 with RT 11.42 min, detected across 11 concentration levels) are recovered and write combined detection, abundance, and S/N results (e.g., to targeted_peak_results.csv).

## Related tools

- **SMART 2.0** (Runs the targeted peak analysis module that matches supplied m/z/RT targets with tolerances against LC-MS data and outputs MS1/MS2 abundance and S/N tables) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Base R tool providing the integrated metabolomics analysis flow (import, visualization, peak analysis) that SMART 2.0 extends with the targeted module) — https://github.com/YuJenL/SMART

## Evaluation signals

- Heroin is detected in MS1 across 11 concentration levels at m/z 370.16 with an associated RT of 11.42 min
- The amphetamine S/N ratio reproduces the reported value of 0.19 for the narcotics data set
- Output targeted_peak_results.csv contains combined peak detection, abundance, and S/N entries for all 12 targeted narcotics
- MS/MS fragmentation for ambiguous MS1 peaks is consistent (e.g., amphetamine fragments at m/z 91, 119, 136 at 500 ppb)
- All targets reported as detected fall within the specified m/z and RT tolerance windows

## Limitations

- Direct one-to-one identification of narcotics without prior drug information is not feasible because other metabolites may have identical m/z values; RT information is essential
- SMART 2.0 cannot detect heroin and nimetazepam for peak identification because HMDB does not encompass them
- Low-abundance peaks may be mis-detected when no RT is provided, and inappropriate peak detection parameters can cause failure to capture the correct signal (as suspected for amphetamine, S/N 0.19)
- Consistency in LC-MS experimental conditions is crucial but not guaranteed; observed variations degrade matching and quantification accuracy

## Evidence

- [discussion] the S/N ratio of amphetamine was found to be only 0.19: "the S/N ratio of amphetamine was found to be only 0.19"
- [discussion] Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
- [discussion] direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values: "direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values"
- [abstract] The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART.: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
- [readme] Peak Analysis and annotation: Implement peak analysis for both untargeted and targeted data and peak annotation.: "Peak Analysis and annotation: Implement peak analysis for both untargeted and targeted data and peak annotation."
