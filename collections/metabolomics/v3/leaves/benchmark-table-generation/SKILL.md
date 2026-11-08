---
name: benchmark-table-generation
description: Use when a paper reports the same error metric (e.g., averaged relative
  error, AvRE) for a method and competing baselines across multiple experiments that
  differ in spectrum difficulty, and a per-experiment gap table sorted by difficulty
  is needed for benchmark validation.
when_to_use_negative:
- The input is raw NMR spectra needing quantification — use Magnetstein itself (or
  peak-picking baselines) to generate the metric values rather than compiling a comparison
  table
- The article reports metrics for a single experiment only — an ordered difficulty-annotated
  benchmark table adds no value without multiple difficulty levels
- The metrics are absolute concentrations (pi, external-standard quantification) rather
  than relative-error aggregates — a gap-in-percentage-points table would misrepresent
  the comparison
tools:
- name: Magnetstein
  role: Method under benchmark; its AvRE column is the reference against which baseline
    gaps are computed (optimal-transport-based 1H NMR quantification with tunable
    κmixture and κcomponents)
  repo: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- name: ACD/Spectrus-Target-Protected Profiling (ACD/TP)
  role: Commercial peak-picking baseline; its AvRE column supplies gap_acdtp = avre_acdtp
    − avre_magnetstein
- name: MANIQ
  role: Commercial peak-picking baseline; its AvRE column supplies gap_maniq = avre_maniq
    − avre_magnetstein, including the Experiment 2 reversal
- name: pandas
  role: Used in Python to compute per-experiment gap columns and write the sorted
    CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:1db0403c2227624656ed3270af62749f245253e45174c169ec3fe5cf6c0a2f96
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  repo_url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - ACD/Spectrus-Target-Protected Profiling (ACD/TP)
  - MANIQ
  - Magnetstein
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# benchmark-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a difficulty-annotated benchmark CSV comparing a method's reported accuracy metric (here, AvRE of Magnetstein) against baseline tools (ACD/TP, MANIQ) across graded-difficulty experiments, computing per-experiment performance gaps for validation.

## When to use

Use when a paper reports the same error metric (e.g., averaged relative error, AvRE) for a method and competing baselines across multiple experiments that differ in spectrum difficulty, and a per-experiment gap table sorted by difficulty is needed for benchmark validation.

## When NOT to use

- The input is raw NMR spectra needing quantification — use Magnetstein itself (or peak-picking baselines) to generate the metric values rather than compiling a comparison table
- The article reports metrics for a single experiment only — an ordered difficulty-annotated benchmark table adds no value without multiple difficulty levels
- The metrics are absolute concentrations (pi, external-standard quantification) rather than relative-error aggregates — a gap-in-percentage-points table would misrepresent the comparison

## Inputs

- Reported AvRE (averaged relative error) values per tool per experiment, transcribed from the results section or figures of the benchmarking article
- Experiment descriptions sufficient to assign an ordinal difficulty_type (e.g., well-separated mixture, overlapping peaks, low-field benchtop spectrum, inhomogeneous-field lineshape distortion, temperature-shifted peaks)
- Optional: per-component relative errors (RE for α-pinene, limonene, etc.) for cross-checking AvRE aggregates

## Outputs

- avre_gap_comparison_table.csv with columns: experiment, difficulty_type, avre_magnetstein, avre_acdtp, avre_maniq, gap_acdtp, gap_maniq, one row per experiment, sorted by increasing difficulty
- Validated gap values (baseline AvRE minus method AvRE, in percentage points) for each experiment/tool pair

## How to apply

First, compile the reported metric values for every tool and experiment from the results section — for Magnetstein vs ACD/TP and MANIQ on 1H NMR mixtures this yields (Exp1–Exp5): Magnetstein 0.0/3.0/9.0/3.6/3.6%, ACD/TP 18.3/6.9/56.9/39.7/35.8%, MANIQ 12.2/1.0/28.8/34.9/7.2%. Second, annotate each experiment with its difficulty category in monotonic order: well-separated mixture (~1:10 concentration difference), contaminant/overlap (limonene/α-pinene with degradation peaks), low-field benchtop overlap, lineshape distortion (spectrum acquired in an inhomogeneous magnetic field), and peak shifts (36.7 °C sample vs 22.27 °C library spectra). Third, in pandas compute gap columns as baseline − Magnetstein (e.g., gap_acdtp = 36.1 pp in Experiment 4). Fourth, write avre_gap_comparison_table.csv with columns experiment, difficulty_type, avre_magnetstein, avre_acdtp, avre_maniq, gap_acdtp, gap_maniq, sorted by increasing difficulty. Fifth, validate: gaps must remain near or exceed an order of magnitude in Experiments 1, 3, 4, and 5, and the single reversal in Experiment 2 (MANIQ 1.0% beating Magnetstein 3.0%) must appear as a negative gap rather than being smoothed over.

## Related tools

- **Magnetstein** (Method under benchmark; its AvRE column is the reference against which baseline gaps are computed (optimal-transport-based 1H NMR quantification with tunable κmixture and κcomponents)) — https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
- **ACD/Spectrus-Target-Protected Profiling (ACD/TP)** (Commercial peak-picking baseline; its AvRE column supplies gap_acdtp = avre_acdtp − avre_magnetstein)
- **MANIQ** (Commercial peak-picking baseline; its AvRE column supplies gap_maniq = avre_maniq − avre_magnetstein, including the Experiment 2 reversal)
- **pandas** (Used in Python to compute per-experiment gap columns and write the sorted CSV)

## Examples

```
import pandas as pd; df = pd.read_csv('avre_raw.csv'); df['gap_acdtp']=df.avre_acdtp-df.avre_magnetstein; df['gap_maniq']=df.avre_maniq-df.avre_magnetstein; df.to_csv('avre_gap_comparison_table.csv', index=False)
```

## Evaluation signals

- Experiment 4 gap check: gap_acdtp must equal 39.7 − 3.6 = 36.1 percentage points, reproducing the reported value
- Order-of-magnitude invariant: gap must be near or exceed the method's own AvRE by ~10x in Experiments 1 (0.0% vs 18.3%/12.2%), 3 (9.0% vs 56.9%/28.8%), 4, and 5 (3.6% vs 35.8%/7.2%)
- Reversal check: Experiment 2 must show gap_maniq = 1.0 − 3.0 = −2.0 pp (negative), correctly capturing MANIQ outperforming Magnetstein with default settings
- Schema check: CSV contains exactly the columns experiment, difficulty_type, avre_magnetstein, avre_acdtp, avre_maniq, gap_acdtp, gap_maniq with 5 rows sorted by increasing difficulty
- Per-component consistency: AvRE aggregates must be reconcilable with reported per-component REs (e.g., Experiment 3: α-pinene RE 125.0%/100.0%, limonene RE 100.0%/11.1% for ACD/TP/MANIQ)

## Limitations

- Small κmixture values cause Magnetstein to assign the smaller component into vortex ω, producing large relative error — default-setting benchmark rows can hide that tuning improves performance (Experiments 1 and 2)
- No version numbers for Magnetstein, MANIQ, or ACD/TP, and no Zenodo DOI/hosting URL given in the article, limiting reproducibility of the benchmarked configurations
- The 'order of magnitude' claim does not hold uniformly — Experiment 2 is a documented reversal where MANIQ (1.0%) beats Magnetstein (3.0%) under default settings

## Evidence

- [abstract] Order-of-magnitude trueness advantage: "Magnetstein can quantitatively analyze difficult spectra with the estimation trueness an order of magnitude higher than that of commercial tools"
- [results] Experiment 2 baseline reversal: "the best results provided by MANIQ (AvRE 3.0% for Magnetstein, 6.9% for ACD/TP, and 1.0% for MANIQ)"
