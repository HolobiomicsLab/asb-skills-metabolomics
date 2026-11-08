---
name: matrix-matched-calibration-curve-analysis
description: Use when you have raw LC-MS/MS files (e.g., Bruker Q-ToF positive-mode
  fecal extracts) plus spiked matrix-matched standards and need absolute per-sample
  concentrations for statistical testing.
when_to_use_negative:
- Input is already a concentration table or relative-abundance feature table rather
  than raw chromatographic peak areas requiring calibration
- No blank matrix-matched extract or spiked standards are available, so a linear response-vs-amount
  regression cannot be built
- The analytes of interest do not ionize well or are present in quantities insufficient
  for MS/MS detection, as reverse metabolomics and targeted quantification both require
  detectable spectra
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Bruker Q-ToF
  role: Instrument on which the conjugated bile acid MS/MS data in fecal extracts
    were recorded (positive mode); its output peak areas feed the calibration curves
- name: MASST
  role: Repository-scale spectral search used upstream to discover and validate conjugated
    bile acid disease associations across public IBD cohorts
- name: pandas
  role: Used in Python to convert internal-standard-normalized responses to mg/kg
    concentrations and assemble the results table
- name: scipy.stats
  role: One-sided Wilcoxon rank-sum tests comparing active vs inactive CD concentrations
    per analyte
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:2983fefe36b10aa2496599e33c547a681dac6347cf4d9bcdc415abdefd58ea95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bruker Q-ToF
  - MASST
  - pandas
  - scipy.stats
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# matrix-matched-calibration-curve-analysis

## Summary

Quantifies metabolites (e.g., 19 conjugated bile acids) from LC-MS/MS peak areas by building linear calibration regressions in blank fecal-extract matrix and converting internal-standard-normalized responses to mg/kg concentrations.

## When to use

Use when you have raw LC-MS/MS files (e.g., Bruker Q-ToF positive-mode fecal extracts) plus spiked matrix-matched standards and need absolute per-sample concentrations for statistical testing.

## When NOT to use

- Input is already a concentration table or relative-abundance feature table rather than raw chromatographic peak areas requiring calibration
- No blank matrix-matched extract or spiked standards are available, so a linear response-vs-amount regression cannot be built
- The analytes of interest do not ionize well or are present in quantities insufficient for MS/MS detection, as reverse metabolomics and targeted quantification both require detectable spectra

## Inputs

- Raw LC-MS/MS fecal-extract files from MassIVE (MSV000092337, Bruker Q-ToF, positive mode)
- Sample metadata with CD activity status annotations (active vs inactive)
- Matrix-matched calibration points: known analyte amounts spiked into blank fecal extract
- Internal standard (sulfadimethoxine) peak areas for normalization

## Outputs

- bile_acid_cd_quantification.csv with per-sample mg/kg concentrations, group summaries, and one-sided Wilcoxon statistics and p-values for each of the 19 bile acids

## How to apply

Download the LC-MS/MS runs (e.g., MSV000092337, Bruker Q-ToF positive mode) and metadata separating active (n=23) from inactive (n=72) CD samples. Extract ion chromatograms for the 19 conjugated bile acid targets and the sulfadimethoxine internal standard, integrating peak areas in Python with scipy.integrate/numpy. Spike known analyte amounts into blank fecal extract matrix, normalize each analyte peak area by the sulfadimethoxine peak area, and fit a linear response-vs-amount regression per analyte (matrix-matched calibration). Convert sample responses to mg/kg concentrations via the fitted regressions using pandas, then run one-sided Wilcoxon rank-sum tests (scipy.stats) comparing active vs inactive CD for each analyte. Matrix matching is essential because ionization suppression in fecal extract differs from neat solvent calibration.

## Related tools

- **Bruker Q-ToF** (Instrument on which the conjugated bile acid MS/MS data in fecal extracts were recorded (positive mode); its output peak areas feed the calibration curves)
- **MASST** (Repository-scale spectral search used upstream to discover and validate conjugated bile acid disease associations across public IBD cohorts)
- **pandas** (Used in Python to convert internal-standard-normalized responses to mg/kg concentrations and assemble the results table)
- **scipy.stats** (One-sided Wilcoxon rank-sum tests comparing active vs inactive CD concentrations per analyte)

## Examples

```
from scipy.stats import linregress, ranksums; cal = linregress(spiked_amounts, area/area_sulfadimethoxine); U, p = ranksums(active_mgkg, inactive_mgkg, alternative='greater')
```

## Evaluation signals

- Internal-standard-normalized response vs spiked amount should follow the fitted linear regression for each of the 19 bile acids before converting sample responses to mg/kg
- Output CSV (bile_acid_cd_quantification.csv) must contain per-sample mg/kg concentrations, group summaries, and one-sided Wilcoxon statistics and p-values for every analyte
- Result should reproduce the reported outcome that 11 of 19 quantified bile acids are significantly increased in active CD only
- Significant increases should be restricted to the CD group; the same trend must not appear for UC
- Significant analytes should include cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr

## Limitations

- Without isolation and NMR or X-ray structural analyses, isomer ambiguity cannot be excluded for the quantified bile acid structures
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds
- The reported significant increase (11/19) was validated in the CD group but was not observed for UC, so extrapolating the concentration differences to other IBD subtypes is unsupported

## Evidence

- [full_text] Build matrix-matched calibration curves by spiking known analyte amounts into blank fecal extract matrix and fit linear response-vs-amount regressions: [evidence span withheld — anchor status: not_found]
- [full_text] Convert internal-standard-normalized responses to mg/kg: [evidence span withheld — anchor status: not_found]
- [supplementary] Q-ToF detection of conjugated bile acids in IBD fecal samples: "conjugated bile acids detected in human IBD fecal samples using a Bruker Q-ToF in positive mode."
- [discussion] Isomer ambiguity caveat: "However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data."
- [abstract] Cholic acid conjugates increased in Crohn's disease: "Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease."
