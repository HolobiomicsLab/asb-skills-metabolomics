---
name: kendrick-mass-defect-homologous-series-detection
description: Use when MS1 feature lists (m/z, RT, carbon count) from LC- or GC-HRMS
  runs of PFAS-contaminated samples must be reduced by detecting fluorinated homologous
  series, e.g. CF2- or CF2O-based, prior to annotation.
when_to_use_negative:
- When the input is raw vendor data (e.g., Agilent .d) not yet converted to centroided
  mzML via MSConvert — the README requires vendor-independent mzML with DDA centroided
  spectra
- When no MS1 feature list with isotope-derived carbon counts exists (e.g., MS2-only
  or profile-mode data), since KMD is computed on the FeatureFinding output features
- When the sample is not expected to contain homologous fluorinated series (e.g.,
  purely small-molecule metabolomics) — KMD with CF2/CF2O repeating units will flag
  little beyond noise
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3518
tools:
- name: PFΔScreen
  role: Implements KMD analysis with PFAS-representative repeating units ((CF2)n,
    CF2O) within the PFASPrioritization tab, and renders the KMD vs. m/z plot linked
    to the m/z vs. RT plot
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Provides the upstream feature detection (FeatureFinding tab) whose MS1 feature
    list with M+1-derived carbon counts feeds the KMD calculation
- name: MSConvert
  role: Converts vendor raw data (Agilent .d) into the centroided mzML files that
    PFΔScreen requires for KMD analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:aea5d9cfb25aebfa527a0f66309829f3ffbc09ec9684000787c648e6f7c4819f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Kendrick mass defect homologous series detection

## Summary

Flags PFAS-like homologous series in HRMS feature lists by computing Kendrick mass defects with fluorinated repeating units such as (CF2)n and CF2O, as implemented in PFΔScreen's PFASPrioritization step.

## When to use

Use when MS1 feature lists (m/z, RT, carbon count) from LC- or GC-HRMS runs of PFAS-contaminated samples must be reduced by detecting fluorinated homologous series, e.g. CF2- or CF2O-based, prior to annotation.

## When NOT to use

- When the input is raw vendor data (e.g., Agilent .d) not yet converted to centroided mzML via MSConvert — the README requires vendor-independent mzML with DDA centroided spectra
- When no MS1 feature list with isotope-derived carbon counts exists (e.g., MS2-only or profile-mode data), since KMD is computed on the FeatureFinding output features
- When the sample is not expected to contain homologous fluorinated series (e.g., purely small-molecule metabolomics) — KMD with CF2/CF2O repeating units will flag little beyond noise

## Inputs

- Centroided ddMS2 mzML sample file (optionally with a matching extraction-blank mzML) loaded in the PFΔScreen GUI
- FeatureFinding-tab output: sample feature list with m/z, RT, and M+1-derived carbon counts (optionally blank feature list)
- Choice of Kendrick repeating unit definition (e.g., CF2, CF2O)

## Outputs

- KMD vs. m/z interactive HTML plot linked to the m/z vs. RT plot for systematic RT-shift verification
- KMD homologous-series assignments recorded as a field in the exported Excel prioritization results table

## How to apply

After the MD/C-m/C prefilter has discarded non-PFAS-like features (m/C cutoff of 30, MD/C cutoff of < +0.003, carbon counts derived from M+1 isotopes), compute Kendrick mass defect for the remaining features with a PFAS-representative repeating unit such as (CF2)n or CF2O. Flag homologous series using a mass tolerance of ±2 mDa and require at least 3 homologues per series (±1 mDa was used for the figure in the article). Inspect the KMD vs. m/z plot interactively linked to the m/z vs. RT plot to verify that flagged series members show a systematic RT shift rather than random co-assignment. Treat flagged series as prioritized PFAS candidates only — KMD series assignment alone does not confirm identity, so carry the features forward to MS2 diagnostic-fragment matching and accurate-mass suspect screening (4 mDa tolerance) for confirmation. Results, including KMD series assignments, are written to the Excel results table and interactive HTML plots in a folder named after the sample.

## Related tools

- **PFΔScreen** (Implements KMD analysis with PFAS-representative repeating units ((CF2)n, CF2O) within the PFASPrioritization tab, and renders the KMD vs. m/z plot linked to the m/z vs. RT plot) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Provides the upstream feature detection (FeatureFinding tab) whose MS1 feature list with M+1-derived carbon counts feeds the KMD calculation)
- **MSConvert** (Converts vendor raw data (Agilent .d) into the centroided mzML files that PFΔScreen requires for KMD analysis)

## Examples

```
Run_PFAScreen.bat  # start PFΔScreen GUI; after Run FeatureFinding, set KMD repeating unit (CF2)n or CF2O (±2 mDa, ≥3 homologues) and click Run PFASPrioritization
```

## Evaluation signals

- Every flagged homologous series contains at least 3 homologues within the ±2 mDa mass tolerance of the chosen repeating unit
- The sample-named results folder contains the KMD vs. m/z HTML plot, and its link to the m/z vs. RT plot shows a systematic RT shift for valid series members
- The exported Excel results table includes a populated KMD series-assignment field for the prioritized features
- Features that failed the MD/C-m/C prefilter (m/C cutoff 30, MD/C < +0.003) do not appear in KMD series assignments
- KMD-flagged features overlap substantially with features later supported by MS2 diagnostic-fragment matches or suspect-list hits (4 mDa tolerance), rather than being exclusively unconfirmed

## Limitations

- Series flagging is tolerance-sensitive: ±2 mDa was used in the workflow, but a tighter ±1 mDa was used in Fig. 4 of the paper, so results differ with parameter choice
- A minimum of 3 homologues is required, so shorter fluorinated chains without at least 3 detected homologues are missed
- KMD series assignment is a prioritization signal only; features still require MS2 diagnostic-fragment evidence or suspect-screen confirmation to be called as PFAS
- Input parameters have no published defaults and should be adjusted per sample, instrument, and end-user needs; PFΔScreen runtimes (<1 min for ~4000 spectra) are reported without hardware specifications

## Evidence

- [results] repeating units representative for PFAS such as (CF2)n and CF2O were applied to detect HS (mass tolerance was set at ± 2 mDa, with at least 3 homologues): "repeating units representative for PFAS such as (CF2)n and CF2O were applied to detect HS (mass tolerance was set at ± 2 mDa, with at least 3 homologues)"
- [readme] a KMD vs. m/z with linked m/z vs. RT plot (to verify systematic RT-shifts): "a KMD vs. m/z with linked m/z vs. RT plot (to verify systematic RT-shifts)"
- [readme] PFΔScreen uses several techniques for prioritization such as the MD/C-m/C approach, Kendrick mass defect (KMD) analysis: "PFΔScreen uses several techniques for prioritization such as the MD/C-m/C approach, Kendrick mass defect (KMD) analysis and fragment mass differences and diagnostic fragments in the MS2 data."
