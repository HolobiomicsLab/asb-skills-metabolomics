---
name: spectral-difficulty-stratification
description: Use when comparing quantification methods across multiple mixture-spectrum
  benchmarks and you need to show whether a method's advantage scales with spectral
  difficulty rather than just averaging scores.
when_to_use_negative:
- You only have a single experiment or a single difficulty level — no stratification
  or trend can be established
- Methods report different error metrics than AvRE (e.g., only per-component RE) without
  a common aggregate, making gap computation incomparable
- The goal is absolute quantification accuracy rather than relative comparison of
  methods across difficulty strata
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Magnetstein
  role: Method under evaluation whose AvRE is compared against baselines across difficulty
    strata
  repo: https://pubs.acs.org/doi/pdf/10.1021/acs.analchem.3c03594
- name: MANIQ
  role: Commercial peak-picking baseline whose AvRE gap to Magnetstein is computed
    per experiment
- name: ACD/Spectrus-Target-Protected Profiling
  role: Commercial peak-picking baseline whose AvRE gap to Magnetstein is computed
    per experiment
- name: pandas
  role: Used to compute per-experiment AvRE gaps and assemble the sorted comparison
    CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:96a2a85d1f5a98864ad1fe807e938d22e3a7ad9ef958a3bd19a7fe545cf0eea2
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
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# spectral-difficulty-stratification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for ordering NMR mixture-analysis benchmark experiments along an axis of spectral difficulty (well-separated → contaminant/overlap → low-field overlap → lineshape distortion → peak shifts) and quantifying how each method's AvRE gap to baselines grows with difficulty.

## When to use

Use when comparing quantification methods across multiple mixture-spectrum benchmarks and you need to show whether a method's advantage scales with spectral difficulty rather than just averaging scores.

## When NOT to use

- You only have a single experiment or a single difficulty level — no stratification or trend can be established
- Methods report different error metrics than AvRE (e.g., only per-component RE) without a common aggregate, making gap computation incomparable
- The goal is absolute quantification accuracy rather than relative comparison of methods across difficulty strata

## Inputs

- Published per-experiment AvRE values for the method and baselines (e.g., Experiments 1–5 from the Magnetstein results section)
- Experiment difficulty annotations (separation quality, field strength, lineshape distortion, temperature mismatch)
- Ordering of experiments by increasing spectral difficulty

## Outputs

- CSV table (avre_gap_comparison_table.csv) with columns experiment, difficulty_type, avre_magnetstein, avre_acdtp, avre_maniq, gap_acdtp, gap_maniq sorted by increasing difficulty
- Per-experiment AvRE gap values in percentage points for each baseline
- Validation summary confirming order-of-magnitude claims and identified reversals

## How to apply

Compile per-experiment AvRE values for the method under test and all baselines from the results section (e.g., Magnetstein vs ACD/TP vs MANIQ in Experiments 1–5: 0.0%/18.3%/12.2%; 3.0%/6.9%/1.0%; 9.0%/56.9%/28.8%; 3.6%/39.7%/34.9%; 3.6%/35.8%/7.2%). Annotate each experiment with its difficulty category in increasing order: well-separated mixture, contaminant/overlapping peaks, low-field benchtop overlap, lineshape distortion from inhomogeneous field, and peak shifts (36.7 °C sample vs 22.27 °C library). Using pandas, compute the per-experiment gap as baseline AvRE minus method AvRE (e.g., ACD/TP − Magnetstein = 36.1 pp in Experiment 4) and write a CSV sorted by difficulty with columns experiment, difficulty_type, avre_<method>, avre_<baseline>, gap_<baseline>. Validate by checking gaps reproduce reported claims: the advantage should approach or exceed an order of magnitude in Experiments 1, 3, 4, and 5, while correctly capturing reversals such as Experiment 2 where MANIQ (1.0%) beats Magnetstein (3.0%).

## Related tools

- **Magnetstein** (Method under evaluation whose AvRE is compared against baselines across difficulty strata) — https://pubs.acs.org/doi/pdf/10.1021/acs.analchem.3c03594
- **MANIQ** (Commercial peak-picking baseline whose AvRE gap to Magnetstein is computed per experiment)
- **ACD/Spectrus-Target-Protected Profiling** (Commercial peak-picking baseline whose AvRE gap to Magnetstein is computed per experiment)
- **pandas** (Used to compute per-experiment AvRE gaps and assemble the sorted comparison CSV)

## Examples

```
import pandas as pd; df = pd.DataFrame({'experiment':['Exp1','Exp2','Exp3','Exp4','Exp5'],'gap_acdtp':[18.3,3.9,47.9,36.1,32.2],'gap_maniq':[12.2,-2.0,19.8,31.3,3.6]}); df.to_csv('avre_gap_comparison_table.csv', index=False)
```

## Evaluation signals

- Computed gaps reproduce reported values, e.g., ACD/TP − Magnetstein = 36.1 pp in Experiment 4 (39.7% − 3.6%)
- Gap remains near or exceeds an order of magnitude in Experiments 1, 3, 4, and 5, matching the claim of 'estimation trueness an order of magnitude higher'
- The single reversal is correctly captured: Experiment 2 where MANIQ (1.0%) outperforms Magnetstein (3.0%)
- CSV row count equals the number of experiments (5) and rows are sorted by increasing difficulty category
- All AvRE values in the CSV match the source figures exactly (0.0, 3.0, 9.0, 3.6, 3.6 for Magnetstein)

## Limitations

- Difficulty ordering is a manual annotation; the article defines categories qualitatively (e.g., '36.7 °C sample vs 22.27 °C library') rather than via a numeric difficulty score
- Small κmixture values can cause Magnetstein to assign the smaller component to vortex ω, producing large relative errors that distort per-experiment gaps
- Gap magnitudes depend on default settings; tuning κmixture and κcomponents changes the values being compared

## Evidence
