---
name: nist20-benchmark-result-extraction
description: Use when a published Table 1-style results table reports mean and standard
  deviation performance over multiple random seeds for two model variants (e.g., fragmentation
  depth ablations) on NIST20, and a delta/ablation summary is needed.
when_to_use_negative:
- When raw spectra or model checkpoints are available and scores should be recomputed
  from scratch rather than transcribed from the published table
- When the comparison involves models evaluated under different experimental conditions
  (e.g., different instrument platforms or precursor adducts), since the paper only
  reports NIST20 held-out performance for merged spectra
- When the target table mixes metrics other than C_BIN/C_HUN or splits other than
  InChIKey/Scaffold without clearly documented values
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Scripting environment used to parse the reported values and build the delta
    table
- name: Pandas
  role: Assembles metrics, standard deviations, and deltas into a structured CSV table
- name: NumPy
  role: Numerical verification of computed deltas and standard deviations
- name: FraGNNet
  role: Source model whose D3 and D4 variants are the subjects of the extracted benchmark
    comparison
  repo: github.com/FraGNNet/fragnnet
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:68086b70600ff76e542d00caec8d6e25437535a069c2fb9e270455b3c6d292d1
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - NumPy
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nist20-benchmark-result-extraction

## Summary

Extracts the reported NIST20 MS/MS compound-identification benchmark figures (C_BIN and C_HUN, mean±std over 5 seeds) for FraGNNet-D4 and FraGNNet-D3 from the paper's results table, computes per-split deltas, and assembles them into a validated CSV comparison table.

## When to use

Use when a published Table 1-style results table reports mean and standard deviation performance over multiple random seeds for two model variants (e.g., fragmentation depth ablations) on NIST20, and a delta/ablation summary is needed.

## When NOT to use

- When raw spectra or model checkpoints are available and scores should be recomputed from scratch rather than transcribed from the published table
- When the comparison involves models evaluated under different experimental conditions (e.g., different instrument platforms or precursor adducts), since the paper only reports NIST20 held-out performance for merged spectra
- When the target table mixes metrics other than C_BIN/C_HUN or splits other than InChIKey/Scaffold without clearly documented values

## Inputs

- Published results table (Table 1) reporting NIST20 C_BIN and C_HUN values as mean and standard deviation over 5 random seeds
- InChIKey and Scaffold split designations for each reported metric

## Outputs

- CSV table with columns for model variant (D3/D4), metric (C_BIN, C_HUN), split (InChIKey, Scaffold), mean, standard deviation, and delta (D4 minus D3)

## How to apply

Load the reported spectrum prediction performance results for FraGNNet-D4 and FraGNNet-D3 on the NIST20 MS/MS dataset (Table 1), which are given as mean and standard deviation over 5 random seeds. Extract the C_BIN and C_HUN values for FraGNNet-D4 (0.736±0.002 and 0.709±0.002 on InChIKey; 0.678±0.005 and 0.654±0.006 on Scaffold) and FraGNNet-D3 (0.721±0.002 and 0.693±0.002 on InChIKey; 0.659±0.001 and 0.633±0.001 on Scaffold). Compute deltas (D4 minus D3) per metric and split: +0.015/+0.016 (InChIKey) and +0.019/+0.021 (Scaffold). Use Python with Pandas and NumPy to assemble metrics, standard deviations, and deltas into a structured CSV table. Validate by confirming D4 > D3 for all four metric-split combinations and that values match the published Table 1 figures exactly.

## Related tools

- **Python** (Scripting environment used to parse the reported values and build the delta table)
- **Pandas** (Assembles metrics, standard deviations, and deltas into a structured CSV table)
- **NumPy** (Numerical verification of computed deltas and standard deviations)
- **FraGNNet** (Source model whose D3 and D4 variants are the subjects of the extracted benchmark comparison) — github.com/FraGNNet/fragnnet

## Examples

```
import pandas as pd; df = pd.DataFrame([{'model':'D4','metric':'C_BIN','split':'InChIKey','mean':0.736,'std':0.002},{'model':'D3','metric':'C_BIN','split':'InChIKey','mean':0.721,'std':0.002}]); df.to_csv('nist20_ablation.csv', index=False)
```

## Evaluation signals

- Recomputed delta for each metric-split combination matches the difference of the reported means (e.g., 0.736 - 0.721 = +0.015 for C_BIN InChIKey)
- D4 > D3 holds for all four metric-split combinations (C_BIN InChIKey, C_HUN InChIKey, C_BIN Scaffold, C_HUN Scaffold)
- All 16 extracted values (mean and std for 2 models × 2 metrics × 2 splits) reproduce the published Table 1 figures exactly
- CSV round-trips through Pandas without dtype loss and preserves the ±0.002/±0.005/±0.006/±0.001 standard deviations verbatim

## Limitations

- Values are transcribed from the paper rather than recomputed, so any error in the source table propagates into the CSV
- The benchmark covers NIST20 merged spectra with [M+H]+ Orbitrap HCD conditions; it does not cover unmerged spectra or broader experimental conditions (different instrument platforms and precursor adducts)
- Standard deviations over only 5 random seeds give limited power for detecting small performance differences

## Evidence

- [methods] Extract the C_BIN and C_HUN values for FraGNNet-D4 (0.736±0.002, 0.709±0.002 on InChIKey; 0.678±0.005, 0.654±0.006 on Scaffold) and FraGNNet-D3 (0.721±0.002, 0.693±0.002 on InChIKey; 0.659±0.001, 0.633±0.001 on Scaffold): [evidence span withheld — anchor status: not_found]
- [readme] The FraGNNet model and baselines were implemented in Python, using Pytorch: [evidence span withheld — anchor status: external_doc]
