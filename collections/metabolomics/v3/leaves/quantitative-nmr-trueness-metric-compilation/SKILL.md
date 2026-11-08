---
name: quantitative-nmr-trueness-metric-compilation
description: Use when benchmarking quantitative NMR mixture analysis tools on spectra
  of graded difficulty (overlap, contaminants, low-field benchtop, lineshape distortion,
  peak shifts) and you need a per-experiment trueness gap table.
when_to_use_negative:
- The goal is per-component relative error decomposition (e.g., α-pinene RE 125.0%/100.0%,
  limonene RE 100.0%/11.1% in Experiment 3) rather than averaged trueness gaps
- Tuning κmixture/κcomponents behavior is the analysis target — this skill uses fixed
  reported AvRE values, not parameter sweeps (e.g., lowering κmixture to remove contaminant
  peaks in Experiment 2)
- The spectra are from a different modality (e.g., mass spectrometry via Masserstein)
  where the AvRE difficulty ordering of Experiments 1–5 does not apply
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: Magnetstein
  role: Open-source optimal-transport-based 1H NMR quantification algorithm whose
    AvRE values form the reference column of the comparison table
  repo: github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: MANIQ
  role: Peak-picking commercial baseline whose AvRE values provide the gap_maniq column
- name: ACD/Spectrus-Target-Protected Profiling
  role: Peak-picking commercial baseline whose AvRE values provide the gap_acdtp column
- name: pandas
  role: Python library used to compute per-experiment AvRE gaps and assemble the sorted
    CSV table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:925d48c8ff24c5695c463f50c664ba8f4ad35a3b0daef49188c569aaf2a784f8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - ACD/Spectrus-Target-Protected Profiling
  - MANIQ
  - Magnetstein
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# quantitative-nmr-trueness-metric-compilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compiles averaged relative error (AvRE) values for Magnetstein and peak-picking baselines (MANIQ, ACD/Spectrus-Target-Protected Profiling) across difficulty-graded 1H NMR mixture experiments into a gap-comparison CSV, testing whether trueness advantages grow with spectral difficulty.

## When to use

Use when benchmarking quantitative NMR mixture analysis tools on spectra of graded difficulty (overlap, contaminants, low-field benchtop, lineshape distortion, peak shifts) and you need a per-experiment trueness gap table.

## When NOT to use

- The goal is per-component relative error decomposition (e.g., α-pinene RE 125.0%/100.0%, limonene RE 100.0%/11.1% in Experiment 3) rather than averaged trueness gaps
- Tuning κmixture/κcomponents behavior is the analysis target — this skill uses fixed reported AvRE values, not parameter sweeps (e.g., lowering κmixture to remove contaminant peaks in Experiment 2)
- The spectra are from a different modality (e.g., mass spectrometry via Masserstein) where the AvRE difficulty ordering of Experiments 1–5 does not apply

## Inputs

- Reported AvRE values for Magnetstein, ACD/TP, and MANIQ per experiment (from the results section, Experiments 1–5)
- Experiment difficulty annotations (well-separated mixture, contaminant/overlap, low-field benchtop, lineshape distortion, peak shifts)

## Outputs

- avre_gap_comparison_table.csv with columns experiment, difficulty_type, avre_magnetstein, avre_acdtp, avre_maniq, gap_acdtp, gap_maniq sorted by increasing difficulty
- Per-experiment AvRE gap values in percentage points validated against reported claims

## How to apply

Compile reported AvRE values per experiment (Exp1: 0.0% vs 18.3%/12.2%; Exp2: 3.0% vs 6.9%/1.0%; Exp3: 9.0% vs 56.9%/28.8%; Exp4: 3.6% vs 39.7%/34.9%; Exp5: 3.6% vs 35.8%/7.2% for Magnetstein/ACD-TP/MANIQ). Annotate each experiment with its difficulty type in increasing order: well-separated mixture (~1:10 concentration ratio), contaminant/overlap (limonene and α-pinene with degradation-contaminant resonances), low-field benchtop overlap, inhomogeneous-field lineshape distortion, and temperature-induced peak shifts (36.7 °C sample vs 22.27 °C library). Compute per-experiment gaps as baseline AvRE minus Magnetstein AvRE (e.g., ACD/TP − Magnetstein = 36.1 pp in Experiment 4) using pandas, write avre_gap_comparison_table.csv sorted by increasing difficulty, and validate that gaps near or exceed an order of magnitude in Experiments 1, 3, 4, and 5 while capturing the single reversal in Experiment 2 where MANIQ (1.0%) beats Magnetstein (3.0%).

## Related tools

- **Magnetstein** (Open-source optimal-transport-based 1H NMR quantification algorithm whose AvRE values form the reference column of the comparison table) — github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **MANIQ** (Peak-picking commercial baseline whose AvRE values provide the gap_maniq column)
- **ACD/Spectrus-Target-Protected Profiling** (Peak-picking commercial baseline whose AvRE values provide the gap_acdtp column)
- **pandas** (Python library used to compute per-experiment AvRE gaps and assemble the sorted CSV table)

## Examples

```
import pandas as pd; df = pd.DataFrame({'experiment': [1,2,3,4,5], 'difficulty_type': ['well-separated','contaminant/overlap','low-field benchtop','lineshape distortion','peak shifts'], 'avre_magnetstein': [0.0,3.0,9.0,3.6,3.6], 'avre_acdtp': [18.3,6.9,56.9,39.7,35.8], 'avre_maniq': [12.2,1.0,28.8,34.9,7.2]}); df['gap_acdtp'] = df.avre_acdtp - df.avre_magnetstein; df['gap_maniq'] = df.avre_maniq - df.avre_magnetstein; df.to_csv('avre_gap_comparison_table.csv', index=False)
```

## Evaluation signals

- Computed gaps reproduce the reported magnitudes, e.g., gap_acdtp = 36.1 pp in Experiment 4 (39.7% − 3.6%)
- The gap near or exceeds an order of magnitude in Experiments 1, 3, 4, and 5, matching the claim that Magnetstein's 'estimation trueness [is] an order of magnitude higher than that of commercial tools'
- The single reversal is captured: Experiment 2 shows MANIQ (1.0%) outperforming Magnetstein (3.0%), so gap_maniq must be negative there
- CSV schema check: exactly seven columns (experiment, difficulty_type, avre_magnetstein, avre_acdtp, avre_maniq, gap_acdtp, gap_maniq), five rows sorted by increasing difficulty
- All AvRE values fall in the 0–57% range reported across Experiments 1–5; any value outside this range signals a transcription error

## Limitations

- Experiment 2 is a genuine reversal where MANIQ (1.0%) outperforms Magnetstein (3.0%), so the order-of-magnitude claim does not hold universally
- AvRE in Experiment 1 (0.0%) is very high trueness on a simple ~1:10 two-component mixture; small κmixture values can cause Magnetstein to assign the smaller component to vortex ω, producing large relative error
- Reported AvRE values come from default settings unless otherwise stated; optimal κmixture/κcomponents results are reported separately in Table S1
- The article does not specify versions of the commercial baselines used for comparison

## Evidence
