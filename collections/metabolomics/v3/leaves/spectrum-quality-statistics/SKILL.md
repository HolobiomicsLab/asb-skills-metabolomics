---
name: spectrum-quality-statistics
description: Use when validated peptide-spectrum matches and de novo predictions are
  available and you must quantify missing fragment ions, noise-peak prevalence, or
  quality-dependent peptide recall per tool.
when_to_use_negative:
- Spectra have not been validated (no trusted peptide-spectrum matches), so peptide
  peaks cannot be distinguished from noise peaks
- The goal is end-to-end antibody assembly rather than characterizing why de novo
  tools fail on low-quality spectra
- Only high-resolution fragment-ion assignments with isoleucine/leucine discrimination
  are needed — MS/MS alone cannot resolve such isomers
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Pyteomics
  role: 'Peak labeling: matching eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+),
    y-NH3, y-H2O) per cleavage site within 0.5 Da tolerance to detect missing fragment
    ions and classify noise'
- name: Casanovo
  role: De novo predictions whose peptide recall is binned by missing cleavage sites
    and noise factor; performs best on spectra with up to eight missing cleavage sites
- name: Novor
  role: De novo predictions contributing recall values to the quality-binned curves
- name: pNovo 3
  role: De novo predictions contributing recall values to the quality-binned curves
- name: DeepNovo
  role: De novo predictions contributing recall values to the quality-binned curves
- name: SMSNet
  role: De novo predictions contributing recall values to the quality-binned curves
- name: PointNovo
  role: De novo predictions contributing recall values to the quality-binned curves
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:8eafac53267e3bbe8d86312ab832d7b0168184d9004977f5d137793e0ca4ee7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - DeepNovo
  - Novor
  - PointNovo
  - Pyteomics
  - SMSNet
  - pNovo 3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectrum-quality-statistics

## Summary

Quantifies spectral quality of validated antibody MS/MS spectra — missing cleavage sites, noise-peak fraction, and noise factor — and relates these quality metrics to peptide recall of de novo sequencing tools (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo). It explains why de novo tools succeed or fail on antibody spectra beyond raw accuracy numbers.

## When to use

Use when validated peptide-spectrum matches and de novo predictions are available and you must quantify missing fragment ions, noise-peak prevalence, or quality-dependent peptide recall per tool.

## When NOT to use

- Spectra have not been validated (no trusted peptide-spectrum matches), so peptide peaks cannot be distinguished from noise peaks
- The goal is end-to-end antibody assembly rather than characterizing why de novo tools fail on low-quality spectra
- Only high-resolution fragment-ion assignments with isoleucine/leucine discrimination are needed — MS/MS alone cannot resolve such isomers

## Inputs

- Validated MS/MS spectra with peptide-spectrum match annotations (23,227 validated antibody spectra)
- De novo peptide predictions from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo
- Median noise intensity threshold per data set

## Outputs

- Spectrum-level quality table (missing fragment ions, noise-peak fraction, noise factor) as spectrum_statistics.csv
- Recall-vs-missing-cleavages curve (Figure 3A)
- Recall-vs-noise-factor curve (Figure 3B)
- Peptide length x missing-cleavage-sites accuracy heatmap (Figure 4)

## How to apply

Load the validated spectra and de novo predictions for Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo, then use Pyteomics to label each peak as a peptide peak or noise peak: for every cleavage site attempt to match eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) within a 0.5 Da tolerance, declaring a cleavage site missing when no ion matches. Count noise peaks only when they exceed the median noise intensity per data set, and compute the noise factor per spectrum as the ratio of high-intensity noise peaks to fragment ion peaks. Bin spectra by number of missing cleavage sites and by noise factor to compute per-tool peptide recall curves, and cross-tabulate recall by peptide length and missing cleavage sites for the accuracy matrix. The analysis reproduces the paper's finding that 90.51% of the 23,227 validated spectra miss at least one fragment ion and 84.32% of peaks are noise, with Casanovo achieving the highest recall on spectra with up to eight missing cleavage sites while noise factor alone has little effect.

## Related tools

- **Pyteomics** (Peak labeling: matching eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) per cleavage site within 0.5 Da tolerance to detect missing fragment ions and classify noise)
- **Casanovo** (De novo predictions whose peptide recall is binned by missing cleavage sites and noise factor; performs best on spectra with up to eight missing cleavage sites)
- **Novor** (De novo predictions contributing recall values to the quality-binned curves)
- **pNovo 3** (De novo predictions contributing recall values to the quality-binned curves)
- **DeepNovo** (De novo predictions contributing recall values to the quality-binned curves)
- **SMSNet** (De novo predictions contributing recall values to the quality-binned curves)
- **PointNovo** (De novo predictions contributing recall values to the quality-binned curves)

## Examples

```
from pyteomics.mass import calculate_mass; tol = 0.5  # match b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O per cleavage site
```

## Evaluation signals

- Reproduced aggregate statistics must match the paper: 90.51% of 23,227 validated spectra missing at least one fragment ion and 84.32% of peaks classified as noise
- Recall-vs-missing-cleavages curves must show Casanovo with the highest peptide recall on spectra with up to eight missing cleavage sites
- Noise-factor bins should show little effect of noise factor alone on peptide recall, mirroring Figure 3B
- Peak-labeling invariants: every peak assigned to exactly one class (peptide peak vs noise peak), with noise peaks restricted to those above the per-dataset median noise intensity
- Per-spectrum noise factor equals the ratio of high-intensity noise peaks to fragment ion peaks and lies in [0, 1]

## Limitations

- The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools
- Noise factor alone has little effect on peptide recall, so quality statistics must be interpreted jointly with missing cleavage sites and peptide length
- Isoleucine/leucine discrimination and other ambiguous residue assignments cannot be resolved via MS/MS, capping achievable peptide-recall fidelity

## Evidence

- [other] 90.51% of the 23,227 validated spectra were missing at least one fragment ion, 84.32% of peaks in these spectra were noise: [evidence span withheld — anchor status: not_found]
- [intro] Investigation of error types and noise impact: "we investigate common error types, the impact of noisy spectra and missing fragmentation ions"
- [discussion] Quality limits for de novo tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
- [results] DL tools achieve higher recall: "The deep learning-based tools Casanovo and PointNovo showed an increased peptide recall across different enzymes and data sets compared with spectrum-graph-based approaches"
