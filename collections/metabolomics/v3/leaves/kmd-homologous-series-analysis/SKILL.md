---
name: kmd-homologous-series-analysis
description: Use when prioritizing PFAS features in non-target ddMS2 HRMS data (centroided
  mzML) and you need to flag homologous series sharing fluorinated repeating units,
  e.g. after MD/C-m/C data reduction in PFΔScreen.
when_to_use_negative:
- Input data are not centroided ddMS2 mzML (e.g. profile spectra or non-DDA acquisitions)
  — PFΔScreen expects DDA data with centroided spectra and ideally one collision energy
  per precursor.
- The analysis goal is a non-fluorinated compound class without PFAS-characteristic
  repeating units such as (CF2)n or CF2O — KMD series detection here is tuned to fluorine-dominated
  moieties.
- A vendor feature table is provided without the corresponding mzML file — MS2-based
  prioritization steps require the raw data alongside external feature lists.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Executes KMD analysis with (CF2)n and CF2O repeating units within the PFASPrioritization
    step and renders KMD vs. m/z HTML plots
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Provides the upstream feature detection whose feature list feeds the KMD homologous
    series detection
  repo: https://github.com/JonZwe/PFAScreen
- name: Python
  role: Runtime environment (Python 3.9) in which PFΔScreen's KMD calculations are
    implemented and executed
  repo: https://github.com/JonZwe/PFAScreen
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:06fefd153df5c5b6b22c045f1e41af53556a2fb342570d7f9b62c381f054e31e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - OpenMS
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# KMD homologous series analysis

## Summary

Kendrick mass defect (KMD) analysis as implemented in PFΔScreen, which detects PFAS homologous series in feature tables from LC/GC-HRMS mzML data using fluorinated repeating units such as (CF2)n and CF2O.

## When to use

Use when prioritizing PFAS features in non-target ddMS2 HRMS data (centroided mzML) and you need to flag homologous series sharing fluorinated repeating units, e.g. after MD/C-m/C data reduction in PFΔScreen.

## When NOT to use

- Input data are not centroided ddMS2 mzML (e.g. profile spectra or non-DDA acquisitions) — PFΔScreen expects DDA data with centroided spectra and ideally one collision energy per precursor.
- The analysis goal is a non-fluorinated compound class without PFAS-characteristic repeating units such as (CF2)n or CF2O — KMD series detection here is tuned to fluorine-dominated moieties.
- A vendor feature table is provided without the corresponding mzML file — MS2-based prioritization steps require the raw data alongside external feature lists.

## Inputs

- Feature list from PFΔScreen feature finding (pyOpenMS FeatureFinding on a ddMS2 centroided mzML sample file, optionally after blank correction)
- MS1 m/z and RT values of detected features, post MD/C-m/C reduction (m/C > 30, MD/C < +0.003)
- KMD analysis parameters: repeating units ((CF2)n, CF2O), mass tolerance (±2 mDa), minimum homologue count (≥3)

## Outputs

- Interactive KMD vs. m/z plot (HTML) linked to m/z vs. RT plot, for verifying systematic RT shifts of homologous series
- PFΔScreen results table (Excel) with features flagged as members of KMD homologous series

## How to apply

Run KMD analysis within the PFΔScreen 'Run PFASPrioritization' step, after feature finding via pyOpenMS (10 ppm mass error, 2000 counts MS1 intensity threshold, OpenMS metabolites 5% RMS isotope model) and MD/C-m/C data reduction (m/C cutoff 30, MD/C cutoff < +0.003). Apply KMD analysis using repeating units representative for PFAS such as (CF2)n and CF2O, with a mass tolerance of ±2 mDa (±1 mDa can be used for stricter visualization as in Fig. 4) and requiring at least 3 homologues per series. Inspect the interactive KMD vs. m/z plot linked to the m/z vs. RT plot to verify systematic RT shifts that confirm true homologous series rather than coincidental mass-defect matches. Series flagged by KMD are reported in the PFΔScreen results table (Excel) together with features passing diagnostic fragment, fragment mass difference, and suspect screening (4 mDa) checks.

## Related tools

- **PFΔScreen** (Executes KMD analysis with (CF2)n and CF2O repeating units within the PFASPrioritization step and renders KMD vs. m/z HTML plots) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Provides the upstream feature detection whose feature list feeds the KMD homologous series detection) — https://github.com/JonZwe/PFAScreen
- **Python** (Runtime environment (Python 3.9) in which PFΔScreen's KMD calculations are implemented and executed) — https://github.com/JonZwe/PFAScreen

## Examples

```
Run_PFAScreen.bat  # launch PFΔScreen GUI, load sample.mzML, Run FeatureFinding then Run PFASPrioritization with (CF2)n/CF2O KMD units, ±2 mDa, ≥3 homologues
```

## Evaluation signals

- Detected homologous series each contain at least 3 homologues within the ±2 mDa mass tolerance of the chosen repeating unit ((CF2)n or CF2O).
- Series members show systematic RT shifts consistent with homologous chain extension, verifiable in the linked KMD vs. m/z / m/z vs. RT plots.
- KMD analysis is applied to features that survived upstream reduction (m/C cutoff 30, MD/C cutoff < +0.003), not the raw feature table.
- Results Excel table contains KMD-flagged features alongside diagnostic fragment and suspect screening hits (4 mDa tolerance), and runtime for ~4000 spectra stays below 1 min.

## Limitations

- KMD is one of several prioritization techniques; it complements but does not replace the MD/C-m/C approach, diagnostic fragment detection (±2 mDa, 2000 counts MS2 intensity), and suspect screening.
- Figure 4 used a stricter ±1 mDa tolerance, so tolerance choice affects the number of flagged homologues; the paper reports no quantitative benchmark against alternative data-reduction tools.
- The manual verification step (checking blank occurrence and peak shape until <1% of the most abundant feature) remains necessary after automated prioritization.

## Evidence

- [results] KMD repeating units and tolerance: "repeating units representative for PFAS such as (CF2)n and CF2O were applied to detect HS (mass tolerance was set at ± 2 mDa, with at least 3 homologues)"
- [readme] KMD as part of PFΔScreen prioritization techniques: "PFΔScreen uses several techniques for prioritization such as the MD/C-m/C approach, Kendrick mass defect (KMD) analysis and fragment mass differences"
- [readme] KMD plot for RT-shift verification: "a KMD vs. m/z with linked m/z vs. RT plot (to verify systematic RT-shifts)"
- [discussion] Upstream MD/C-m/C reduction rationale: "the MD/C-m/C approach is a powerful tool to drastically decrease the number of features and thus reduce false-positive assignments"
