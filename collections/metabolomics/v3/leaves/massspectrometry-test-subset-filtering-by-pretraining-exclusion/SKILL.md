---
name: massspectrometry-test-subset-filtering-by-pretraining-exclusion
description: 'Decision support for comparing a learned ranking model against pretrained
  baselines whose training corpora overlap the benchmark test set: filter the test
  molecules by pretraining membership before computing rank@1.'
when_to_use_negative:
- The benchmark test set has no overlap with the baseline tools' pre-training corpora,
  so no exclusion filtering is needed.
- You are evaluating on a dataset whose molecules are unavailable under appropriate
  licensing (per README, other datasets besides NPLIB1/MassSpecGym are license-restricted).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: JESTR
  role: Ranking model benchmarked on the filtered subsets; provides data preparation
    utilities (dataset.py, utils.py) and pretrained MassSpecGym weights
  repo: https://github.com/HassounLab/JESTR1/
- name: SIRIUS
  role: Pretrained baseline whose rank@1 is recomputed on the subset excluding its
    pre-training molecules
- name: CFM-ID
  role: Pretrained baseline whose rank@1 is recomputed on the subset excluding its
    pre-training molecules
- name: MassSpecGym
  role: Benchmark dataset downloaded from HuggingFace; source of test spectra and
    candidate molecules to be filtered
  repo: https://huggingface.co/datasets/roman-bushuiev/MassSpecGym
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf354
    title: 'JESTR: Joint Embedding Space Technique for Ranking candidate molecules
      for the annotation of untargeted metabolomics data'
  source_content_hash: sha256:c687b19ed2fe0aa908d82f73932e65dd2598be3f034511c81d630cf87fc6f4be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/JESTR1
  tools:
  - CFM-ID
  - JESTR
  - MassSpecGym
  - Python
  - SIRIUS
  - params.yaml
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# massspectrometry-test-subset-filtering-by-pretraining-exclusion

## Summary

Builds filtered MassSpecGym test subsets that exclude molecules seen in SIRIUS's and CFM-ID's pre-training corpora, enabling a leakage-controlled rank@1 comparison where JESTR is benchmarked against these pretrained candidate-ranking tools.

## When to use

Decision support for comparing a learned ranking model against pretrained baselines whose training corpora overlap the benchmark test set: filter the test molecules by pretraining membership before computing rank@1.

## When NOT to use

- The benchmark test set has no overlap with the baseline tools' pre-training corpora, so no exclusion filtering is needed.
- You are evaluating on a dataset whose molecules are unavailable under appropriate licensing (per README, other datasets besides NPLIB1/MassSpecGym are license-restricted).

## Inputs

- MassSpecGym dataset from HuggingFace (roman-bushuiev/MassSpecGym)
- SIRIUS pre-training molecule list for exclusion
- CFM-ID pre-training molecule list for exclusion
- JESTR data preparation scripts (dataset.py, utils.py)
- Pretrained JESTR weights (data/MassSpecGym)

## Outputs

- Filtered MassSpecGym test subset excluding SIRIUS pre-training molecules
- Filtered MassSpecGym test subset excluding CFM-ID pre-training molecules
- Per-tool rank@1 on each filtered subset
- Percent improvement of JESTR over SIRIUS and CFM-ID
- baseline_comparison.csv with rank@1 for JESTR, SIRIUS, CFM-ID on the filtered subsets

## How to apply

Download the MassSpecGym dataset from the HuggingFace repository roman-bushuiev/MassSpecGym, then use the JESTR data preparation utilities (dataset.py / utils.py) to construct two test subsets: one excluding molecules present in SIRIUS's pre-training corpus and one excluding molecules present in CFM-ID's pre-training corpus. Run SIRIUS and CFM-ID with their publicly available pretrained models on these filtered subsets to obtain candidate rankings, and rank the same subsets with JESTR using the ranking pipeline (cand_rank_canopus.py / JESTR.ipynb) with pretrained weights (MassSpecGym weights are distributed under data/MassSpecGym in the repo). Compute rank@1 for each tool on each subset and record the percent improvement of JESTR over each baseline; the paper reports JESTR beats SIRIUS by 31% (10.41% vs 13.65%) and CFM-ID by 238% (2.00% vs median 6.76%) on the respective filtered subsets. Save the per-subset rank@1 table as baseline_comparison.csv and verify reproduced improvements match those figures.

## Related tools

- **JESTR** (Ranking model benchmarked on the filtered subsets; provides data preparation utilities (dataset.py, utils.py) and pretrained MassSpecGym weights) — https://github.com/HassounLab/JESTR1/
- **SIRIUS** (Pretrained baseline whose rank@1 is recomputed on the subset excluding its pre-training molecules)
- **CFM-ID** (Pretrained baseline whose rank@1 is recomputed on the subset excluding its pre-training molecules)
- **MassSpecGym** (Benchmark dataset downloaded from HuggingFace; source of test spectra and candidate molecules to be filtered) — https://huggingface.co/datasets/roman-bushuiev/MassSpecGym

## Examples

```
python cand_rank_canopus.py
```

## Evaluation signals

- Reproduced JESTR rank@1 on the SIRIUS-filtered subset must exceed SIRIUS's by ≈31% (10.41% vs 13.65%).
- Reproduced JESTR median rank@1 on the CFM-ID-filtered subset must exceed CFM-ID's by ≈238% (2.00% vs 6.76%).
- baseline_comparison.csv must contain rank@1 entries for all three tools on both filtered subsets.
- Filtered subsets must exclude molecules present in the respective tool's pre-training corpus, verified via the exclusion lists used with dataset.py / utils.py.
- Environment must be GPU-capable (nVidia A100, CUDA 11.8) as the released weights are GPU-trained.

## Limitations

- Other datasets besides NPLIB1 and MassSpecGym are under licensing agreements that prohibit public release; users must ensure dataset availability under appropriate licenses.
- Missing tool/library versions are not stated in the article.
- Pretrained weights are GPU-trained; the environment must be set up for GPU.
- The article does not provide explicit discussion-section details (only a 'No changelog found.' placeholder).

## Evidence

- [abstract] JESTR outperforms SIRIUS by 31% and CFM-ID by 238% on filtered MassSpecGym subsets: [evidence span withheld — anchor status: not_found]
- [readme] MassSpecGym dataset source: "The MassSpecGym dataset can be downloaded from the HuggingFace repository MassSpecGym."
- [readme] Data preparation utilities for custom datasets: "For ranking candidates for other datasets, the user can prepare the dataset using the appropriate data preparation files explained below"
- [intro] GPU requirement for pretrained weights: "The model was trained and tested on GPU nVidia A100 with CUDA 11.8."
- [intro] Dataset licensing restrictions: "We have released the dataset and pretrained weights for the NPLIB1 dataset."
- [methods] Configuration parameters for exclusion filtering: "The parameters set in this file are: - exp: dataset to be used."
