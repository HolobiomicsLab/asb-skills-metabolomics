---
name: single-cell-proteomics-sample-metadata-mapping
description: Use when you have per-run feature lists (m/z, charge, RT, intensity)
  from single-cell DIA-MS runs and need to associate each feature file with its cell/sample
  identity before cohort-wide RT alignment and alignment-vs-identification benchmarking.
when_to_use_negative:
- You already have a peptide-level identification table from DIA-NN or another MBR
  workflow — the DeepRTAlign feature-based path is for ID-free feature alignment,
  not reprocessing identified peptides
- You have raw vendor Thermo/Bruker files — Dinosaur feature extraction must be run
  first; DeepRTAlign consumes text feature lists, not raw spectra
- The single-cell runs come from different projects — results are overwritten if multiple
  projects are run under the same folder
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Dinosaur
  role: Feature extraction tool producing per-run feature lists (m/z, charge, RT,
    intensity) that DeepRTAlign consumes in Dinosaur mode
- name: DeepRTAlign
  role: Deep learning-based RT alignment of the single-cell DIA feature lists across
    the 18-cell cohort; outputs the post-QC aligned feature table
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: DIA-NN
  role: Identification-based baseline whose match-between-runs (MBR) alignment results
    (aligned peptides per cell) provide the reference for the comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:656af3c89e8cc41416266c97530a6bcf9b4cf1dcc671af7f77a1e5c06994d125
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DIA-NN
  - DeepRTAlign
  - Dinosaur
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-proteomics-sample-metadata-mapping

## Summary

Map per-run Dinosaur feature lists from single-cell DIA-MS runs (e.g., 18 HT-22 cells, PRIDE PXD025634) to sample names via an Excel sample list, then run DeepRTAlign retention-time alignment so aligned peptides and features can be counted per cell and compared against a DIA-NN MBR baseline.

## When to use

Use when you have per-run feature lists (m/z, charge, RT, intensity) from single-cell DIA-MS runs and need to associate each feature file with its cell/sample identity before cohort-wide RT alignment and alignment-vs-identification benchmarking.

## When NOT to use

- You already have a peptide-level identification table from DIA-NN or another MBR workflow — the DeepRTAlign feature-based path is for ID-free feature alignment, not reprocessing identified peptides
- You have raw vendor Thermo/Bruker files — Dinosaur feature extraction must be run first; DeepRTAlign consumes text feature lists, not raw spectra
- The single-cell runs come from different projects — results are overwritten if multiple projects are run under the same folder

## Inputs

- Per-run Dinosaur feature lists (m/z, charge, RT, intensity columns) from 18 HT-22 single-cell DIA-MS runs (PRIDE PXD025634)
- Excel sample list (sample_file.xlsx) mapping each Dinosaur feature file to its sample/cell name
- Fresh working folder containing file_dir and sample_file

## Outputs

- information_target.csv in the mass_align_all_information folder with sample, fraction, time, mz, charge, intensity, score and group columns
- Per-cell counts of aligned features and aligned peptides present in at least two cells for comparison against DIA-NN MBR

## How to apply

Extract features from each of the 18 single-cell DIA-MS runs with Dinosaur, yielding per-run lists with m/z, charge, RT and intensity columns. Build an Excel sample list recording the correspondences between feature files and sample names, and place it together with the feature-file directory in a fresh working folder (results are overwritten otherwise). Run the DeepRTAlign CLI in Dinosaur mode with the parallel processing parameter set to the CPU core count, keeping defaults (1 min coarse-alignment time window, 0.03 m/z bin width, 2-decimal bin precision, score-based QC with FDR cutoff 0.01, keep_best_feature=1, keep_best_group=1). The pipeline runs six internal steps: preprocessing of input files, coarse alignment, binning, filtering, alignment, and generating the result file. Collect information_target.csv from the mass_align_all_information folder, then count aligned features and aligned peptides present in at least two cells per cell to compare against the DIA-NN MBR reference (39 additional aligned peptides per cell, i.e., 6.33%; ~42.3x feature-to-peptide ratio).

## Related tools

- **Dinosaur** (Feature extraction tool producing per-run feature lists (m/z, charge, RT, intensity) that DeepRTAlign consumes in Dinosaur mode)
- **DeepRTAlign** (Deep learning-based RT alignment of the single-cell DIA feature lists across the 18-cell cohort; outputs the post-QC aligned feature table) — https://github.com/PHOENIXcenter/deeprtalign
- **DIA-NN** (Identification-based baseline whose match-between-runs (MBR) alignment results (aligned peptides per cell) provide the reference for the comparison)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -pn 8
```

## Evaluation signals

- information_target.csv exists in mass_align_all_information with sample, fraction, time, mz, charge, intensity, score and group columns
- Each CSV line represents one feature; aligned features share the same group name
- Per-cell counts of aligned peptides present in at least two cells reproduce the reported 39 (6.33%) DeepRTAlign-over-DIA-NN advantage within the cohort of 18 cells
- Average number of aligned features present in at least two cells is approximately 42.3 times the average number of aligned peptides
- Only one result file exists in mass_align_all_information when keep_temp=0 (default); two files (target and all/decoy) when kt=1

## Limitations

- Do not run different projects under a same folder — results will be overwritten; always create a new folder
- Default score-based QC (FDR 0.01) reports very few results when samples are quite different or many features share nearly the same m/z; in that case set FDR to 1 and select features with score > 0.5
- A full run on a normal computer takes about 30 minutes; use the -pn parameter set to the CPU core count for speed

## Evidence

- [readme] The sample list is an excel file recording the correspondences between feature files  and sample names.: "The sample list is an excel file recording the correspondences between feature files  and sample names."
- [readme] Create a new folder and run in it: "Do not run the different projects under a same folder, the results will be overwritten."
- [methods] Six-step pipeline: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [readme] Result file schema: "In result files, each line represent a feature, the meaning of each column is as follows."
