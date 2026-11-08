---
name: precision-recall-evaluation
description: Use when benchmarking an RT alignment or feature correspondence tool
  on simulated LC-MS data where the true correspondences are known from the perturbation,
  and per-shift-magnitude precision/recall curves are needed.
when_to_use_negative:
- Input is real-world data without simulated RT shifts — ground-truth correspondences
  implied by the perturbation do not exist, so precision/recall cannot be computed
- The goal is to evaluate quantification accuracy rather than alignment/correspondence
  correctness — the article notes alignment has less influence on quantification than
  feature extraction
- Feature lists lack m/z, charge, RT and intensity columns or come from an unsupported
  feature extraction format without the required column mapping
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: DeepRTAlign
  role: Alignment tool under test; run per simulated dataset with defaults (time_window
    1 min, bin_width 0.03, fdr 0.01) and -pn parallel flag; groups in information_target.csv
    provide predicted correspondences
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: OpenMS
  role: Comparator alignment method; its feature grouping is run on the same simulated
    feature lists to produce baseline precision/recall
- name: numpy
  role: Generates the 24 perturbed copies per real-world dataset by adding normally
    distributed RT shifts to the RT column only
- name: PyTorch
  role: CPU-version dependency installed before DeepRTAlign; powers the deep learning-based
    direct matching step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:a68aa391321371dd72fea8de71e2e2ab1db9daf4cb71487e4f40615e0580a2f2
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
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# precision-recall-evaluation

## Summary

A benchmarking procedure that scores LC-MS feature alignment tools by computing precision and recall against ground-truth feature correspondences implied by simulated retention time (RT) shifts, as used to establish DeepRTAlign's generalizability boundaries versus OpenMS on 336 simulated proteomic and metabolomic datasets.

## When to use

Use when benchmarking an RT alignment or feature correspondence tool on simulated LC-MS data where the true correspondences are known from the perturbation, and per-shift-magnitude precision/recall curves are needed.

## When NOT to use

- Input is real-world data without simulated RT shifts — ground-truth correspondences implied by the perturbation do not exist, so precision/recall cannot be computed
- The goal is to evaluate quantification accuracy rather than alignment/correspondence correctness — the article notes alignment has less influence on quantification than feature extraction
- Feature lists lack m/z, charge, RT and intensity columns or come from an unsupported feature extraction format without the required column mapping

## Inputs

- OpenMS feature lists (text files with m/z, charge, RT and intensity columns) from example_files
- Excel sample list mapping feature files to sample names (sample_file.xlsx)
- numpy-generated perturbed feature lists with normal RT shifts (means 0/5/10 min, sigma 0–5 min, 24 copies per dataset)
- DeepRTAlign output files information_target.csv and information_all.csv from mass_align_all_information folder

## Outputs

- Per-sigma precision and recall values for DeepRTAlign and OpenMS on proteomic and metabolomic simulated datasets
- Aggregated precision/recall curves versus RT shift standard deviation
- Summary table written to simulated_benchmark_results.csv
- Reproduced generalizability boundaries (sigma > 1 min proteomic, sigma > 0.3 min metabolomic)

## How to apply

Generate perturbed feature lists with numpy: create 24 copies per real-world OpenMS feature list, adding normally distributed RT shifts (means of 0, 5, and 10 min; sigma spanning 0–5 min) to the RT column while leaving m/z, charge, and intensity unchanged, so ground-truth correspondences are implied by the perturbation. Run DeepRTAlign on each simulated dataset with default parameters (time_window 1 min, bin_width 0.03, fdr 0.01) and the -pn flag for parallel processing, then run OpenMS feature grouping on the same feature lists as the comparator. Read aligned groups from mass_align_all_information/information_target.csv, where aligned features share the same 'group' name. Evaluate each tool's output against ground truth, computing precision and recall per sigma level separately for proteomic and metabolomic datasets. Verify the reported generalizability boundaries are reproduced: precision and recall drop when sigma exceeds 1 min on proteomic datasets and 0.3 min on metabolomic datasets, then write the summary table to simulated_benchmark_results.csv.

## Related tools

- **DeepRTAlign** (Alignment tool under test; run per simulated dataset with defaults (time_window 1 min, bin_width 0.03, fdr 0.01) and -pn parallel flag; groups in information_target.csv provide predicted correspondences) — https://github.com/PHOENIXcenter/deeprtalign
- **OpenMS** (Comparator alignment method; its feature grouping is run on the same simulated feature lists to produce baseline precision/recall)
- **numpy** (Generates the 24 perturbed copies per real-world dataset by adding normally distributed RT shifts to the RT column only)
- **PyTorch** (CPU-version dependency installed before DeepRTAlign; powers the deep learning-based direct matching step)

## Examples

```
deeprtalign -m OpenMS -f file_dir -s sample_file.xlsx -pn -1
```

## Evaluation signals

- Precision and recall are computed per sigma level against ground-truth correspondences implied by the RT perturbation (features differing only by the added shift must be matched to the same group)
- Aggregated curves reproduce the reported generalizability boundaries: degradation when sigma exceeds 1 min (proteomic) and 0.3 min (metabolomic)
- DeepRTAlign output exists as mass_align_all_information/information_target.csv and aligned features share the same value in the 'group' column
- The default QC uses fdr 0.01; if few results are reported (very different samples or many near-identical m/z features), the README prescribes setting FDR to 1 and selecting score > 0.5
- Summary table is written to simulated_benchmark_results.csv with per-sigma rows for both proteomic and metabolomic datasets and both tools

## Limitations

- The default QC workflow usually reports very few results when samples are quite different or many features share almost the same m/z; the README recommends setting FDR to 1 and selecting score > 0.5 in that case
- Do not run different projects under the same folder — results will be overwritten; create a new folder per project
- The generalizability boundary was determined but limits of applicability remain; simulation perturbs only the RT column, leaving m/z, charge and intensity unchanged, so shifts in other dimensions are untested
- Each DeepRTAlign run takes about 30 minutes on a normal computer without the -pn parallel parameter

## Evidence

- [abstract] DeepRTAlign has been demonstrated to have improved performances by benchmarking it against current state-of-the-art approaches on multiple real-world and simulated proteomic and metabolomic datasets.: "DeepRTAlign has been demonstrated to have improved performances by benchmarking it against current state-of-the-art approaches on multiple real-world and simulated proteomic and metabolomic datasets."
- [discussion] Generalizability boundary: "and determined its generalizability boundary on multiple proteomic and metabolomic datasets."
- [readme] step 6 result output: "The results will output to the mass_align_all_information folder.If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv (target"
- [readme] fdr threshold: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] OpenMS demo invocation: "To handle the OpenMS test data in example_files folder, you can **create a new folder** and put the file_dir (containing result files from feature extraction tool ) and sample_file in, **switch the"
