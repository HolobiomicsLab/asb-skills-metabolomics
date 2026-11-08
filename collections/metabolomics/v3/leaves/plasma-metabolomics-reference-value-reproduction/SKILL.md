---
name: plasma-metabolomics-reference-value-reproduction
description: Use when a published precision claim (e.g., ISTD-normalized average RSD
  per derivatization agent such as PCF, MTBSTFA, MSTFA on BioIVT plasma samples) must
  be independently reproduced from a raw peak-area table.
when_to_use_negative:
- The goal is large-scale untargeted batch normalization of hundreds of metabolite
  annotations (563 annotations, >4000 samples) — use SERDA/SERRF normalization instead,
  where internal standards alone failed to reach median RSD < 30%.
- The peak-area table already contains normalized (rather than raw) peak areas, making
  one-to-one ISTD division invalid.
- The dataset does not pair each detectable amino acid with its own stable-isotope-labeled
  internal standard, so cognate normalization cannot be applied.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Performs one-to-one ISTD normalization (dividing each amino acid peak area
    by its cognate labeled internal standard) and per-trial RSD computation; the SERDA
    runner script runs under R 3.6.3 on Windows 10 x64
- name: gas chromatography-mass spectrometry (GC-MS)
  role: Instrument platform that generated the derivatized plasma peak-area data being
    compared across PCF, MTBSTFA, and MSTFA agents
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:64fb25d53457d887f6c56b70732d5f0d66cbdb3498af237527be9badc2c30b1f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - gas chromatography-mass spectrometry (GC-MS)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# plasma-metabolomics-reference-value-reproduction

## Summary

Reproduces reported precision reference values for derivatization/normalization strategies in GC-MS plasma metabolomics by recomputing internal-standard-normalized RSDs from raw peak-area tables, e.g. PCF (2.7% avg RSD) vs MTBSTFA (8.9%) vs MSTFA (9.6%) for 16 amino acids across 30 BioIVT plasma samples.

## When to use

Use when a published precision claim (e.g., ISTD-normalized average RSD per derivatization agent such as PCF, MTBSTFA, MSTFA on BioIVT plasma samples) must be independently reproduced from a raw peak-area table.

## When NOT to use

- The goal is large-scale untargeted batch normalization of hundreds of metabolite annotations (563 annotations, >4000 samples) — use SERDA/SERRF normalization instead, where internal standards alone failed to reach median RSD < 30%.
- The peak-area table already contains normalized (rather than raw) peak areas, making one-to-one ISTD division invalid.
- The dataset does not pair each detectable amino acid with its own stable-isotope-labeled internal standard, so cognate normalization cannot be applied.

## Inputs

- Supplementary Table S1 peak-area table: 16 amino acids and paired stable-isotope-labeled internal standards, 30 BioIVT plasma samples, three monthly replicate trials, three derivatization agents (PCF, MTBSTFA, MSTFA)
- Mapping of each amino acid to its cognate stable-isotope-labeled internal standard

## Outputs

- istd_aminoacid_rsd.csv with one summary average RSD per derivatization reagent
- Comparison table of reproduced vs reported reference RSD values (PCF 2.7%, MTBSTFA 8.9%, MSTFA 9.6%)

## How to apply

Load the peak-area table (e.g., Supplementary Table S1) containing the 16 detectable amino acids and their paired stable-isotope-labeled internal standards for the 30 BioIVT plasma samples acquired in three monthly replicate trials under PCF, MTBSTFA, and MSTFA derivatization. Apply one-to-one ISTD normalization in R by dividing each amino acid's raw peak area by its cognate internal-standard peak area for every injection — do not use sum-based or batch-correction normalization for this comparison. Compute the RSD (%) of each ISTD-normalized amino acid across the 30 BioIVT plasma samples within each of the three monthly replicate trials, separately per derivatization reagent. Average the per-trial amino-acid RSDs across the three monthly replicate trials to obtain one summary RSD per reagent. Compare reproduced averages against the reported reference values (PCF 2.7%, MTBSTFA 8.9%, MSTFA 9.6%, shown in Figure 3 / Supplementary Table S1) and write the summary to istd_aminoacid_rsd.csv.

## Related tools

- **R** (Performs one-to-one ISTD normalization (dividing each amino acid peak area by its cognate labeled internal standard) and per-trial RSD computation; the SERDA runner script runs under R 3.6.3 on Windows 10 x64)
- **gas chromatography-mass spectrometry (GC-MS)** (Instrument platform that generated the derivatized plasma peak-area data being compared across PCF, MTBSTFA, and MSTFA agents)

## Examples

```
data_file = "filelocation.xlsx"  # edit in SERDA runner.R, then run all script lines in R 3.6.3
```

## Evaluation signals

- Reproduced average RSDs match the reported references: PCF ≈ 2.7%, MTBSTFA ≈ 8.9%, MSTFA ≈ 9.6% (Figure 3 / Supplementary Table S1).
- RSD is computed per monthly replicate trial and then averaged across the three trials, not pooled across all 30 samples in one step.
- Each amino acid is divided only by its own cognate stable-isotope-labeled internal standard (one-to-one pairing), not by a global or summed normalizer.
- Exactly the 16 detectable amino acids and 30 BioIVT plasma samples are used per derivatization reagent.
- istd_aminoacid_rsd.csv is produced with one row per reagent and values within small tolerance of the reference percentages.

## Limitations

- The reference values (2.7%/8.9%/9.6%) come from a small-set derivatization comparison (16 amino acids, 30 BioIVT plasma samples) that is distinct from the paper's large-scale SERDA experiment, where ISTD normalization alone yielded median RSD above 30% across 563 annotations.
- In the large-scale setting, internal-standard normalization gave 19% median precision on pooled QC samples but 34% median RSD on external BioIVT validation pools, so ISTD precision numbers do not transfer across experimental scales.
- Reproduction depends on access to Supplementary Table S1 and correct pairing of amino acids to their cognate labeled internal standards; mismatches will inflate RSDs.

## Evidence

- [abstract] ISTD normalization fails at scale without learned correction: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [abstract] BioIVT plasma pools serve as the external validation set: "external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [methods] How to run the provided R normalization script: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [readme] Repository provides the SERDA normalization implementation: "This is the repository for SERDA normalization"
- [readme] Execution environment for the R pipeline: "R version 3.6.3 (2020-02-29)
Platform: x86_64-w64-mingw32/x64 (64-bit)
Running under: Windows 10 x64 (build 19043)"
