---
name: alignment-benchmark-comparison
description: Use when you need to quantify how many more peptides or MS features a
  candidate LC-MS retention time alignment workflow aligns versus MBR-based or feature-based
  baselines, while checking that quantitative accuracy is not compromised.
when_to_use_negative:
- The goal is biomarker discovery or downstream modeling on aligned features rather
  than comparing alignment tool performance
- Only a single LC-MS run is available — RT alignment and MBR comparison are undefined
  without multiple runs
- You need raw feature extraction optimization rather than alignment evaluation (the
  authors note alignment has less influence on quantification than feature extraction)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Candidate deep-learning RT alignment tool whose aligned feature groups (information_target.csv)
    are counted and compared against baselines
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: MaxQuant
  role: Feature extraction (allPeptides.txt) and MBR baseline workflow against which
    the candidate alignment is benchmarked
- name: MSFragger
  role: Supplies peptide-spectrum identifications mapped onto aligned feature groups;
    its MBR results serve as a baseline
- name: OpenMS
  role: Feature alignment baseline on the same dataset
- name: DIA-NN
  role: MBR baseline for single-cell DIA comparison of aligned peptides per cell
- name: Dinosaur
  role: Feature extraction tool producing per-run feature lists for single-cell DIA
    inputs to DeepRTAlign
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:ed0882eec92fb19313c494d1d178abe97b0e0ecb3396e334ea224a547284b583
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DIA-NN
  - DeepRTAlign
  - Dinosaur
  - MSFragger
  - MaxQuant
  - OpenMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# alignment-benchmark-comparison

## Summary

Benchmarks a deep-learning retention time alignment workflow (e.g. DeepRTAlign combined with feature extraction and identification tools) against established match-between-runs and feature-alignment baselines (MaxQuant MBR, MSFragger MBR, OpenMS, DIA-NN) by counting aligned peptides/features and comparing quantification accuracy.

## When to use

Use when you need to quantify how many more peptides or MS features a candidate LC-MS retention time alignment workflow aligns versus MBR-based or feature-based baselines, while checking that quantitative accuracy is not compromised.

## When NOT to use

- The goal is biomarker discovery or downstream modeling on aligned features rather than comparing alignment tool performance
- Only a single LC-MS run is available — RT alignment and MBR comparison are undefined without multiple runs
- You need raw feature extraction optimization rather than alignment evaluation (the authors note alignment has less influence on quantification than feature extraction)

## Inputs

- Per-run feature lists (m/z, charge, RT, intensity columns) from MaxQuant allPeptides.txt, Dinosaur, OpenMS, or XICFinder
- Excel sample list mapping feature files to sample names
- Peptide-spectrum identification files (e.g. MSFragger PSM/peptide outputs) for mapping IDs onto aligned groups
- Baseline workflow outputs: OpenMS feature alignment results, MSFragger MBR results, DIA-NN MBR results on the same runs

## Outputs

- Per-workflow counts of identified aligned peptides (and aligned MS features present in at least two runs)
- Quantification accuracy metrics per workflow
- Benchmark comparison table (e.g. benchmark_comparison.csv) of candidate vs. baseline alignment performance

## How to apply

Prepare per-run feature lists (e.g. MaxQuant allPeptides.txt or Dinosaur outputs with m/z, charge, RT and intensity columns) plus an Excel sample list mapping feature files to sample names, then run the candidate aligner (e.g. `deeprtalign -m MaxQuant -f file_dir -s sample_file.xlsx` with -pn set to the CPU core count) so it executes preprocessing, coarse alignment, binning (bin_width 0.03, bin_precision 2), filtering, deep-learning direct matching, and score-based QC with FDR cutoff 0.01. Map peptide-spectrum identifications (e.g. from MSFragger) onto the aligned feature groups to obtain identified aligned peptides per workflow. Run the baselines on the same dataset: OpenMS feature alignment and MSFragger MBR for DDA benchmark-FC data, or DIA-NN MBR for single-cell DIA data. Count identified aligned peptides (and aligned features present in at least two runs for single-cell comparisons) per workflow and compute quantification accuracy metrics, then report relative gains (reference: 150% more peptides than MaxQuant-only, 7% and 14.5% more than OpenMS and MSFragger MBR; 39 (6.33%) more aligned peptides per cell than DIA-NN).

## Related tools

- **DeepRTAlign** (Candidate deep-learning RT alignment tool whose aligned feature groups (information_target.csv) are counted and compared against baselines) — https://github.com/PHOENIXcenter/deeprtalign
- **MaxQuant** (Feature extraction (allPeptides.txt) and MBR baseline workflow against which the candidate alignment is benchmarked)
- **MSFragger** (Supplies peptide-spectrum identifications mapped onto aligned feature groups; its MBR results serve as a baseline)
- **OpenMS** (Feature alignment baseline on the same dataset)
- **DIA-NN** (MBR baseline for single-cell DIA comparison of aligned peptides per cell)
- **Dinosaur** (Feature extraction tool producing per-run feature lists for single-cell DIA inputs to DeepRTAlign)

## Examples

```
deeprtalign -m MaxQuant -f file_dir -s sample_file.xlsx -pn 8
```

## Evaluation signals

- information_target.csv exists in mass_align_all_information with sample, fraction, time, mz, charge, intensity, score and group columns
- Candidate workflow yields substantially more aligned peptides than the MaxQuant-only workflow and the OpenMS/MSFragger MBR baselines (reported 150%, 7% and 14.5% more) with comparable quantification accuracy
- On single-cell DIA data, DeepRTAlign aligns on average ~39 (6.33%) more peptides per cell than DIA-NN MBR, and aligned features present in ≥2 cells are ~42.3x the aligned peptide count
- All comparisons are computed on the same underlying runs (e.g. Benchmark-FC HEK/E. coli mixture or 18 HT-22 single-cell DIA runs from PXD025634) so counts are directly comparable
- Score-based QC is applied consistently (FDR cutoff 0.01) before counting identified aligned peptides in every workflow

## Limitations

- DeepRTAlign's default workflow may report very few results when samples are quite different or many features share almost the same m/z; in that case set FDR to 1 and select features with score > 0.5
- Current alignment has limited influence on quantification accuracy; the authors plan to improve quantification by optimizing feature extraction and alignment simultaneously
- DeepRTAlign's generalizability boundary was determined on multiple proteomic and metabolomic datasets but limits of applicability remain
- Results are overwritten if different projects are run under the same folder — always create a new working folder

## Evidence

- [results] Benchmark-FC peptide-alignment comparison finding: [evidence span withheld — anchor status: not_found]
- [methods] Pipeline steps: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [methods] QC threshold: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [results] Single-cell DIA comparison finding: [evidence span withheld — anchor status: not_found]
- [readme] Output file and columns: "The results will output to the mass_align_all_information folder.If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv (target"
- [readme] FDR fallback guidance: "In this case, set FDR to 1, and select features with score greater than 0.5 are recomended."
