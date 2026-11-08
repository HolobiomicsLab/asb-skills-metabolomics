---
name: retention-time-shift-simulation
description: Use when you need to stress-test an LC-MS RT alignment tool against ground-truth
  correspondences, or to determine at what RT-shift variability an aligner's precision
  and recall degrade.
when_to_use_negative:
- The goal is quantification accuracy assessment — the article notes alignment has
  less influence on quantification than feature extraction, and this simulation alters
  only RT
- Input is raw LC-MS profile data rather than post-feature-extraction feature lists;
  DeepRTAlign expects processed feature lists, not centroided spectra
- Only a single RT shift level is needed as a quick sanity check — this skill targets
  a systematic sigma sweep to map degradation boundaries
tools:
- name: DeepRTAlign
  role: Deep learning-based RT alignment tool whose performance is evaluated on each
    simulated RT-shift dataset
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: OpenMS
  role: Comparator feature-grouping alignment method run on the same simulated feature
    lists
- name: PyTorch
  role: Deep learning dependency installed (CPU version) to run DeepRTAlign's direct-matching
    model
- name: numpy
  role: Used to generate normally distributed RT perturbations for the simulated datasets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:0fcf51811235db290c580c3c4bdc6972171f47d667c7c0e63a5d9ea283c5f29e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - OpenMS
  - PyTorch
  - numpy
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# retention-time-shift-simulation

## Summary

Simulates controlled retention time (RT) shifts on LC-MS feature lists to benchmark RT alignment tools such as DeepRTAlign against OpenMS feature grouping. It exposes generalizability boundaries — where alignment precision/recall collapse as RT shift variability grows — for proteomic and metabolomic data.

## When to use

Use when you need to stress-test an LC-MS RT alignment tool against ground-truth correspondences, or to determine at what RT-shift variability an aligner's precision and recall degrade.

## When NOT to use

- The goal is quantification accuracy assessment — the article notes alignment has less influence on quantification than feature extraction, and this simulation alters only RT
- Input is raw LC-MS profile data rather than post-feature-extraction feature lists; DeepRTAlign expects processed feature lists, not centroided spectra
- Only a single RT shift level is needed as a quick sanity check — this skill targets a systematic sigma sweep to map degradation boundaries

## Inputs

- Feature lists from a feature extraction tool supported by DeepRTAlign (Dinosaur, OpenMS, MaxQuant, XICFinder, or TXT/CSV with m/z, charge, RT, intensity columns)
- Sample list (.xlsx) mapping feature files to sample names
- Per-dataset sigma grid (0–5 min) and shift means (0, 5, 10 min) defining the perturbation design

## Outputs

- 24 perturbed feature-list copies per real-world dataset with shifted RT columns
- Per-sigma precision and recall values for DeepRTAlign and OpenMS on proteomic and metabolomic datasets
- Summary table of alignment performance (e.g. simulated_benchmark_results.csv) with generalizability boundaries

## How to apply

Load OpenMS feature lists and the sample list (e.g. from the DeepRTAlign example_files folder, such as the UPS2-Y dataset). Use numpy to generate 24 perturbed copies per real-world dataset, adding normally distributed RT shifts with means of 0, 5, and 10 min and sigma spanning 0–5 min to the RT column only, leaving m/z, charge and intensity unchanged so perturbed features define ground-truth correspondence. Run DeepRTAlign per simulated dataset with default parameters (time_window 1 min, bin_width 0.03, fdr 0.01) and the -pn parallel flag, and run OpenMS feature grouping on the same lists as comparator. Compute precision and recall per sigma level against the ground-truth pairs; the article's reported boundaries are degradation when sigma exceeds 1 min (proteomics) and 0.3 min (metabolomics).

## Related tools

- **DeepRTAlign** (Deep learning-based RT alignment tool whose performance is evaluated on each simulated RT-shift dataset) — https://github.com/PHOENIXcenter/deeprtalign
- **OpenMS** (Comparator feature-grouping alignment method run on the same simulated feature lists)
- **PyTorch** (Deep learning dependency installed (CPU version) to run DeepRTAlign's direct-matching model)
- **numpy** (Used to generate normally distributed RT perturbations for the simulated datasets)

## Examples

```
deeprtalign -m OpenMS -f file_dir -s sample_file.xlsx -pn 8
```

## Evaluation signals

- Perturbed features must differ from originals only in the RT column; m/z, charge and intensity values are invariant
- Ground-truth correspondences are exactly the perturbation pairs, so precision/recall are computable without ID-based matching
- Reproduced boundary: DeepRTAlign precision/recall drop significantly when RT-shift sigma exceeds 1 min on most simulated proteomic datasets and exceeds 0.3 min on most metabolomic datasets
- DeepRTAlign outputs land in mass_align_all_information/information_target.csv with a group column; aligned true pairs should share a group name
- Default parameters must match the paper's settings: time_window 1 min, bin_width 0.03, fdr 0.01

## Limitations

- The generalizability boundary was determined, but limits of applicability remain: performance degrades beyond ~1 min (proteomics) / ~0.3 min (metabolomics) sigma
- Default QC workflow can report very few results when samples are quite different or many features share almost the same m/z; in that case set FDR to 1 and select features with score > 0.5
- Simulation assumes Gaussian RT shifts with fixed means; non-Gaussian or non-monotonic shift regimes of other shapes are not covered by this design

## Evidence

- [readme] DeepRTAlign is a deep learning-based retention time alignment tool for large cohort LC-MS data analysis.: "DeepRTAlign is a deep learning-based retention time alignment tool for large cohort LC-MS data analysis."
- [discussion] generalizability boundary on multiple datasets: "and determined its generalizability boundary on multiple proteomic and metabolomic datasets."
- [readme] Feature lists and sample list requirement: "Feature lists and sample list should be prepared before running DeepRTAlign."
- [readme] default fdr threshold: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] time_window default rationale: "time_window (float, default 1) depends on your chromatography, 1 min is the parameter we used in training"
