---
name: ground-truth-correspondence-matching
description: 'Use when benchmarking feature-correspondence or RT-alignment tools and
  you need ground truth: simulated perturbed datasets with known feature mappings
  let you compute precision/recall per shift magnitude.'
when_to_use_negative:
- You have no ground truth or cannot simulate perturbations — real-world alignment
  evaluation without known correspondences cannot yield precision/recall
- Your goal is quantitative accuracy assessment rather than correspondence correctness
  — the article notes alignment has less influence on quantification than feature
  extraction
- Input is raw spectra rather than extracted feature lists — DeepRTAlign requires
  feature extraction to have been run first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Deep learning-based RT alignment tool under evaluation; run with deeprtalign
    -m OpenMS -f file_dir -s sample_file.xlsx and default parameters
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: OpenMS
  role: Comparator feature-grouping alignment method run on the same simulated feature
    lists
- name: numpy
  role: Generates perturbed dataset copies by adding normally distributed RT shifts
    to the RT column
- name: PyTorch
  role: CPU dependency required for DeepRTAlign's deep learning-based matching model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:6d7c3ae004132df233c650673907ac0aff9dbf4705fb5c91a309cc3da3f55433
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ground-truth-correspondence-matching

## Summary

Evaluate LC-MS feature alignment tools (e.g., DeepRTAlign vs OpenMS) by simulating known retention time shifts with numpy to create ground-truth correspondences, then computing precision and recall per RT-shift sigma level to delineate generalizability boundaries.

## When to use

Use when benchmarking feature-correspondence or RT-alignment tools and you need ground truth: simulated perturbed datasets with known feature mappings let you compute precision/recall per shift magnitude.

## When NOT to use

- You have no ground truth or cannot simulate perturbations — real-world alignment evaluation without known correspondences cannot yield precision/recall
- Your goal is quantitative accuracy assessment rather than correspondence correctness — the article notes alignment has less influence on quantification than feature extraction
- Input is raw spectra rather than extracted feature lists — DeepRTAlign requires feature extraction to have been run first

## Inputs

- Feature lists from OpenMS/Dinosaur/MaxQuant/XICFinder or generic TXT/CSV with m/z, charge, RT and intensity columns
- Sample list excel file (sample_file.xlsx) mapping feature files to sample names
- numpy-generated perturbed RT-shift copies (24 per real dataset; means 0/5/10 min, sigma 0-5 min)

## Outputs

- Per-sigma-level precision and recall values for each alignment tool
- Generalizability boundary (sigma threshold) per data type
- Summary table, e.g., simulated_benchmark_results.csv

## How to apply

Load feature lists (OpenMS format or Dinosaur/MaxQuant/XICFinder outputs containing m/z, charge, RT and intensity) plus the sample list (sample_file.xlsx) from the tool's example_files folder. Use numpy to generate 24 perturbed copies per real-world dataset, adding normally distributed RT shifts with means of 0, 5, and 10 min and sigma spanning 0-5 min to the RT column only, leaving m/z, charge and intensity unchanged so the original features define ground-truth groups. Run each alignment tool with default parameters (time_window 1 min, bin_width 0.03, fdr 0.01) and parallel processing via -pn. Evaluate each output's aligned groups against the perturbation-implied correspondences, computing precision and recall per sigma level; report the sigma beyond which performance drops as the generalizability boundary (sigma > 1 min proteomic, sigma > 0.3 min metabolomic in the reference study).

## Related tools

- **DeepRTAlign** (Deep learning-based RT alignment tool under evaluation; run with deeprtalign -m OpenMS -f file_dir -s sample_file.xlsx and default parameters) — https://github.com/PHOENIXcenter/deeprtalign
- **OpenMS** (Comparator feature-grouping alignment method run on the same simulated feature lists)
- **numpy** (Generates perturbed dataset copies by adding normally distributed RT shifts to the RT column)
- **PyTorch** (CPU dependency required for DeepRTAlign's deep learning-based matching model)

## Examples

```
deeprtalign -m OpenMS -f file_dir -s sample_file.xlsx -tw 1 -bw 0.03 -fd 0.01 -pn -1
```

## Evaluation signals

- Perturbed features must retain unchanged m/z, charge and intensity so ground-truth groups are exactly recoverable
- Precision and recall should stay high at low sigma and drop sharply beyond the boundary (sigma > 1 min proteomic, > 0.3 min metabolomic)
- DeepRTAlign QC output information_target.csv should contain aligned features sharing the same group name per correspondence
- Summary CSV should cover all 24 perturbed copies per dataset and both proteomic and metabolomic sigma curves
- With default settings, results land in mass_align_all_information folder with one file (kt=0); set fdr=1 and score > 0.5 filtering if too few results are reported

## Limitations

- DeepRTAlign precision and recall degrade significantly once RT-shift standard deviation exceeds 1 min (proteomics) or 0.3 min (metabolomics), bounding generalizability
- When samples differ greatly or many features share nearly the same m/z, the default QC workflow reports very few results; fdr=1 with a score > 0.5 cutoff is recommended
- Results are overwritten if different projects are run in the same folder — each run needs a new working directory

## Evidence

- [results] On most simulated proteomic datasets, DeepRTAlign's precision and recall drop significantly when the standard deviation of the RT shift exceeds 1 min: [evidence span withheld — anchor status: not_found]
- [abstract] Simulated datasets rationale: "DeepRTAlign has been demonstrated to have improved performances by benchmarking it against current state-of-the-art approaches on multiple real-world and simulated proteomic and metabolomic datasets."
- [readme] Default QC and fallback thresholds: "In the default workflow, score will be used for quality control (default). This workflow works well in most cases. But when the samples are quite different, or there are too many features with the"
- [readme] Run command and parameters: "then use command `deeprtalign -m OpenMS -f file_dir -s sample_file.xlsx `. The results will output to the mass_align_all_information folder."
