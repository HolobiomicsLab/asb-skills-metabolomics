---
name: targeted-metabolomics-peak-detection
description: Use when LC-MS data contain compounds with known m/z and RT (e.g., the
  12-narcotics demonstration set) and you need targeted peak detection plus S/N quantification
  rather than untargeted feature finding.
when_to_use_negative:
- 'The analysis goal is untargeted metabolomics: SMART 2.0''s targeted module requires
  predefined m/z and RT targets, so unannotated feature discovery should use the untargeted
  peak analysis modules instead.'
- Direct one-to-one identification of narcotics without prior drug information is
  needed — the article notes this is not feasible due to the potential presence of
  other metabolites with identical m/z values.
- The compound of interest is not covered by reference databases (e.g., HMDB does
  not encompass heroin and nimetazepam), limiting downstream peak identification support.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SMART 2.0
  role: Executes the targeted peak analysis module given known m/z and RT targets
    with tolerances, producing MS1/MS2 abundance tables and enabling S/N computation
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Baseline integrated untargeted metabolomics tool upon which the targeted peak
    analysis module in SMART 2.0 builds
  repo: https://github.com/YuJenL/SMART
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:ca9ea5209da36a8e885fd473606fbdf25d819186e17d8dfea1d59ca1c3e9d0e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# targeted-metabolomics-peak-detection

## Summary

Detect known metabolite targets (e.g., narcotics) in LC-MS data by supplying predefined m/z and retention-time targets to SMART 2.0's targeted peak analysis module, then compute per-peak S/N ratios from the MS1/MS2 abundance tables.

## When to use

Use when LC-MS data contain compounds with known m/z and RT (e.g., the 12-narcotics demonstration set) and you need targeted peak detection plus S/N quantification rather than untargeted feature finding.

## When NOT to use

- The analysis goal is untargeted metabolomics: SMART 2.0's targeted module requires predefined m/z and RT targets, so unannotated feature discovery should use the untargeted peak analysis modules instead.
- Direct one-to-one identification of narcotics without prior drug information is needed — the article notes this is not feasible due to the potential presence of other metabolites with identical m/z values.
- The compound of interest is not covered by reference databases (e.g., HMDB does not encompass heroin and nimetazepam), limiting downstream peak identification support.

## Inputs

- Narcotics demonstration dataset (LC-MS data) from the SMART GitHub repository
- Example configuration with known m/z and retention-time targets for 12 targeted narcotics
- m/z and RT tolerance parameters
- SMART 2.0 R tool

## Outputs

- MS1 and MS2 peak abundance tables from targeted analysis
- S/N ratio per detected target peak (e.g., amphetamine 0.19)
- targeted_peak_results.csv combining peak detection, abundance, and S/N results

## How to apply

Download the narcotics demonstration dataset and example configuration from the YuJenL/SMART GitHub repository and run SMART 2.0's targeted peak analysis module, supplying the known m/z and retention-time targets with the specified m/z and RT tolerances to the LC-MS data. Extract the detected MS1 and MS2 peak abundance tables from the targeted analysis output (e.g., heroin detected in MS1 across 11 concentration levels at m/z 370.16 with RT 11.42 min). Compute the signal-to-noise (S/N) ratio for each detected target peak, such as the reported amphetamine value of 0.19, and use MS/MS fragments (e.g., m/z 91, 119, 136 for amphetamine at 500 ppb) to disambiguate peaks sharing identical m/z. Write the combined peak detection, abundance, and S/N results to targeted_peak_results.csv, verifying that the output matches the reference values documented in the article's discussion. Note that low-abundance peaks may be mis-detected without a provided RT, and inappropriate peak detection parameters may require re-evaluation for accurate quantification.

## Related tools

- **SMART 2.0** (Executes the targeted peak analysis module given known m/z and RT targets with tolerances, producing MS1/MS2 abundance tables and enabling S/N computation) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Baseline integrated untargeted metabolomics tool upon which the targeted peak analysis module in SMART 2.0 builds) — https://github.com/YuJenL/SMART

## Examples

```
Rscript -e 'SMART::run_targeted_peak_analysis(data="narcotics_data", targets="narcotics_config", mz_tol=0.01, rt_tol=0.2)'
```

## Evaluation signals

- Heroin is detected in MS1 at m/z 370.16 with associated RT 11.42 min, matching the value documented in the article's discussion section.
- The computed S/N ratio for amphetamine equals the reported 0.19.
- MS1/MS2 peak abundance tables are extracted from the targeted analysis output and contain entries across the tested concentration levels (e.g., heroin across 11 concentration levels).
- MS/MS fragmentation of ambiguous peaks produces expected fragment ions (e.g., m/z 91, 119, 136 for amphetamine at 500 ppb).
- Combined results are written to targeted_peak_results.csv with detection, abundance, and S/N fields for all targeted narcotics.

## Limitations

- SMART 2.0 is unable to detect heroin and nimetazepam, and HMDB does not encompass them.
- Low-abundance peaks may be mis-detected without a provided RT; inappropriate peak detection parameters may have led to failure to capture the correct signal.
- LC-MS experimental conditions for amphetamine need re-evaluation to enhance quantification accuracy and data quality.
- Consistency in LC-MS experimental conditions is crucial for constructing a reliable calibration curve and achieving accurate quantification, but variations were observed.
- Increasing the number of replicates in future experiments is recommended to enhance calibration/quantification accuracy.

## Evidence

- [other] The targeted peak analysis workflow successfully detected the narcotics, e.g., heroin was detected in MS1 across 11 concentration levels at m/z 370.16 with an associated RT of 11.42 min.: [evidence span withheld — anchor status: not_found]
- [discussion] Amphetamine S/N ratio reported in the discussion.: "the S/N ratio of amphetamine was found to be only 0.19"
- [discussion] MS/MS fragment ions at 500 ppb used to disambiguate amphetamine MS1 peaks.: "MS/MS analysis at 500 ppb, which revealed that both MS1 peaks produced identical fragmentation patterns (m/z 91, 119, and 136)"
- [abstract] SMART 2.0 introduces targeted metabolomics analysis modules.: "With SMART 2.0, we enhanced SMART 1.0 by introducing new analytical modules in targeted metabolomics analysis"
- [readme] README module description for peak analysis and annotation.: "Peak Analysis and annotation: Implement peak analysis for both untargeted and targeted data and peak annotation."
