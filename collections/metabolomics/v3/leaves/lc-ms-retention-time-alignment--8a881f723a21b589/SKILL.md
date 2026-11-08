---
name: lc-ms-retention-time-alignment
description: Use when you have per-run feature lists (m/z, charge, RT, intensity)
  from Dinosaur, OpenMS, MaxQuant, XICFinder or TXT/CSV, need to align corresponding
  features/peptides across many LC-MS runs, or compare against MBR-based tools.
when_to_use_negative:
- Your tool already performs MBR-style alignment and identification jointly (MaxQuant,
  MSFragger, DIA-NN) and you only need identification counts, not cross-run feature
  correspondence
- You have no feature lists yet — DeepRTAlign requires prior feature extraction (e.g.
  Dinosaur on DIA runs) and does not process raw LC-MS files
- Samples require quantification improvement from alignment alone — the authors note
  alignment has less influence on quantification than feature extraction
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: 'Primary deep learning-based RT alignment tool: coarse alignment plus direct
    matching across LC-MS runs with score-based QC (FDR 0.01)'
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: MaxQuant
  role: Feature extraction source; its allPeptides.txt feature lists serve as DeepRTAlign
    input (-m MaxQuant); also a benchmark baseline via its MBR workflow
- name: OpenMS
  role: Feature extraction source (-m OpenMS) and feature-alignment baseline in the
    Benchmark-FC comparison
- name: Dinosaur
  role: Feature extractor for DIA data whose per-run feature lists are aligned by
    DeepRTAlign in Dinosaur mode (e.g. single-cell PXD025634 data)
- name: MSFragger
  role: Identification tool whose peptide-spectrum matches are mapped onto DeepRTAlign-aligned
    feature groups; its MBR serves as a baseline
- name: DIA-NN
  role: MBR-based alignment baseline for single-cell DIA comparison
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:212ca427ad5854d6b8f1d7b573846f565736706e823fb84376de67cca5a3fe57
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
  - PyTorch
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# lc-ms-retention-time-alignment

## Summary

Retention time (RT) alignment/correspondence of LC-MS feature lists across large cohorts using DeepRTAlign, which combines coarse alignment (pseudo warping) with deep-learning direct matching to handle both monotonic and non-monotonic RT shifts in proteomic and metabolomic data.

## When to use

Use when you have per-run feature lists (m/z, charge, RT, intensity) from Dinosaur, OpenMS, MaxQuant, XICFinder or TXT/CSV, need to align corresponding features/peptides across many LC-MS runs, or compare against MBR-based tools.

## When NOT to use

- Your tool already performs MBR-style alignment and identification jointly (MaxQuant, MSFragger, DIA-NN) and you only need identification counts, not cross-run feature correspondence
- You have no feature lists yet — DeepRTAlign requires prior feature extraction (e.g. Dinosaur on DIA runs) and does not process raw LC-MS files
- Samples require quantification improvement from alignment alone — the authors note alignment has less influence on quantification than feature extraction

## Inputs

- Per-run feature lists containing m/z, charge, RT and intensity columns (Dinosaur, OpenMS, XICFinder, or any TXT/CSV text list)
- MaxQuant allPeptides.txt files (first column must match the sample list)
- Excel sample list (.xlsx) mapping feature files to sample names

## Outputs

- mass_align_all_information/information_target.csv — post-QC aligned feature table with sample, fraction, time, mz, charge, intensity, score and group columns
- mass_align_all_information/information_all.csv — pre-QC results including decoy alignments (only when keep_temp=1 / disk mode)

## How to apply

Prepare feature lists and an Excel sample list mapping feature files to sample names (for MaxQuant use allPeptides.txt and delete the 'Resolution' column for versions <2.0). Run DeepRTAlign in a fresh working folder, e.g. `deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx`, choosing -m from Dinosaur/XICFinder/OpenMS/MaxQuant/TXT/CSV (for TXT/CSV supply --mz_col, --charge_col, --rt_col, --intensity_col counted from 1). The pipeline runs six steps: preprocessing, coarse alignment (default 1 min time_window), binning (default bin_width 0.03, bin_precision 2), filtering, deep-learning alignment, and result generation; set -pn to the CPU core count to speed up execution. Post-alignment, score-based quality control applies an FDR cutoff of 0.01 by default; for highly dissimilar samples or many near-identical m/z features, set FDR to 1 and keep features with score greater than 0.5 instead. Use keep_best_feature=1 and keep_best_group=1 (defaults) to keep the highest-scoring feature per sample per group and the group with the most features. Validate by checking mass_align_all_information/information_target.csv with sample, fraction, time, mz, charge, intensity, score and group columns, where aligned features share the same group name.

## Related tools

- **DeepRTAlign** (Primary deep learning-based RT alignment tool: coarse alignment plus direct matching across LC-MS runs with score-based QC (FDR 0.01)) — https://github.com/PHOENIXcenter/deeprtalign
- **MaxQuant** (Feature extraction source; its allPeptides.txt feature lists serve as DeepRTAlign input (-m MaxQuant); also a benchmark baseline via its MBR workflow)
- **OpenMS** (Feature extraction source (-m OpenMS) and feature-alignment baseline in the Benchmark-FC comparison)
- **Dinosaur** (Feature extractor for DIA data whose per-run feature lists are aligned by DeepRTAlign in Dinosaur mode (e.g. single-cell PXD025634 data))
- **MSFragger** (Identification tool whose peptide-spectrum matches are mapped onto DeepRTAlign-aligned feature groups; its MBR serves as a baseline)
- **DIA-NN** (MBR-based alignment baseline for single-cell DIA comparison)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -pn 8
```

## Evaluation signals

- information_target.csv exists in mass_align_all_information with the expected schema: sample, fraction, time, mz, charge, intensity, score, group columns; aligned features share the same group name
- All QC FDR values lie in [0,1] and the FDR at the retained score cutoff is at or below the configured threshold (default 0.01)
- Aligned peptide counts exceed baselines as reported: 150% more than original MaxQuant workflow, 7% and 14.5% more than OpenMS and MSFragger MBR on Benchmark-FC; ~39 (6.33%) more aligned peptides per cell than DIA-NN MBR on single-cell DIA data
- Quantification accuracy is comparable to baseline workflows (no compromise in quantitative accuracy)
- With keep_temp=1, retained groups in information_target.csv match the target entries in information_all.csv after decoy-based FDR filtering

## Limitations

- Default score-based QC reports very few results when samples are quite different or many features share almost the same m/z; in that case set FDR to 1 and keep features with score > 0.5
- Alignment has limited influence on quantification accuracy; future work is needed to improve quantification by optimizing feature extraction and alignment simultaneously
- Results are overwritten if different projects are run under the same folder — always create a new working folder
- MaxQuant inputs require allPeptides.txt with the sample list matching its first column, and the 'Resolution' column must be deleted for MaxQuant versions <2.0
- time_window, bin_width and bin_precision depend on chromatography and feature extraction parameters (defaults: 1 min, 0.03, 2)

## Evidence

- [intro] Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts as well as non-monotonic shifts: "Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts as well as non-monotonic shifts"
- [methods] steps 1-6: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [methods] FDR threshold: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] README output description: "The results will output to the mass_align_all_information folder.If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv"
- [readme] README QC caveat: "when the samples are quite different, or there are too many features with the almost same m/z, the default workflow usually report very few results"
