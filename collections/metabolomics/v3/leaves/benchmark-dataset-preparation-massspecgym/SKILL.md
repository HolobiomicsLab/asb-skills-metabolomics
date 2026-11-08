---
name: benchmark-dataset-preparation-massspecgym
description: Use when evaluating a candidate-ranking tool (e.g. JESTR vs SIRIUS/CFM-ID)
  on tandem mass spectra and you need benchmark subsets from MassSpecGym that exclude
  molecules a pretrained baseline already saw.
when_to_use_negative:
- You only need to run JESTR's ranking demo on a single target molecule against its
  candidates — use the JESTR.ipynb notebook path instead of building filtered benchmark
  subsets.
- You are working on the NPLIB1 dataset, for which JESTR provides its own prepared
  dataset files and pretrained weights; MassSpecGym-specific filtering is not applicable.
- Your evaluation goal does not require excluding pre-trained molecules from baseline
  tools' corpora — e.g. a pure ablation of JESTR's own components on its released
  splits.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3316
tools:
- name: JESTR
  role: Provides the data preparation utilities (dataset.py, utils.py) used to load,
    preprocess, and filter MassSpecGym subsets, and the candidate-ranking pipeline
    whose rank@1 is benchmarked
  repo: https://github.com/HassounLab/JESTR1/
- name: dataset.py
  role: Loads and preprocesses the MassSpecGym dataset into JESTR's data structures
  repo: https://github.com/HassounLab/JESTR1/
- name: utils.py
  role: Utility functions for data preparation; must be updated to load custom dataset
    variants
  repo: https://github.com/HassounLab/JESTR1/
- name: params.yaml
  role: Configuration file setting which dataset (exp) is used and related parameters
  repo: https://github.com/HassounLab/JESTR1/
- name: cand_rank_canopus.py
  role: Template script for running candidate ranking on a complete dataset; adapt
    it for the MassSpecGym filtered subsets
  repo: https://github.com/HassounLab/JESTR1/
- name: MassSpecGym
  role: The benchmark dataset being downloaded and filtered
  repo: https://huggingface.co/datasets/roman-bushuiev/MassSpecGym
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf354
    title: 'JESTR: Joint Embedding Space Technique for Ranking candidate molecules
      for the annotation of untargeted metabolomics data'
  source_content_hash: sha256:e5597b85744d0f675281db2e993375ca0a25df92d6f632b12822b9178439743c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/JESTR1
  tools:
  - CFM-ID
  - JESTR
  - MassSpecGym
  - Python
  - cand_rank_canopus.py
  - dataset.py
  - params.yaml
  - utils.py
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# benchmark-dataset-preparation-massspecgym

## Summary

Prepare MassSpecGym test subsets for molecule-vs-spectrum candidate-ranking benchmarks: download the dataset from HuggingFace, build filtered subsets that exclude molecules present in competitor tools' pre-training corpora, and produce candidate lists plus rank@[1-20] evaluation tables for head-to-head comparison.

## When to use

Use when evaluating a candidate-ranking tool (e.g. JESTR vs SIRIUS/CFM-ID) on tandem mass spectra and you need benchmark subsets from MassSpecGym that exclude molecules a pretrained baseline already saw.

## When NOT to use

- You only need to run JESTR's ranking demo on a single target molecule against its candidates — use the JESTR.ipynb notebook path instead of building filtered benchmark subsets.
- You are working on the NPLIB1 dataset, for which JESTR provides its own prepared dataset files and pretrained weights; MassSpecGym-specific filtering is not applicable.
- Your evaluation goal does not require excluding pre-trained molecules from baseline tools' corpora — e.g. a pure ablation of JESTR's own components on its released splits.

## Inputs

- MassSpecGym dataset (downloaded from HuggingFace roman-bushuiev/MassSpecGym)
- Lists of molecules present in SIRIUS's and CFM-ID's pre-training corpora (for exclusion filtering)
- Candidate molecule lists per target InChIKey downloaded from PubChem (cand_dict.pkl)
- params.yaml configuration file (exp, batch_size*, num_epochs*, aug_cands, pretrained* parameters)
- JESTR data preparation scripts: dataset.py, utils.py

## Outputs

- Filtered test subsets of MassSpecGym excluding SIRIUS-pretrained and CFM-ID-pretrained molecules
- data_dict.pkl-style dictionary of spectra (m/z and intensity arrays, metadata, molecule SMILES and InChIKey)
- split.pkl with train/valid/test lists of InChIKeys
- Per-tool rank@1 (and rank@[1-20]) values on each filtered subset
- Comparison table of rank@1 for JESTR vs SIRIUS vs CFM-ID with percent improvement

## How to apply

Download the MassSpecGym dataset from the HuggingFace repository (roman-bushuiev/MassSpecGym), then use JESTR's data preparation utilities (dataset.py / utils.py) to load and preprocess spectra into data_dict.pkl-style structures (m/z and intensity arrays, metadata, molecule SMILES and InChIKey) plus split.pkl train/valid/test lists of InChIKeys. Build two filtered test subsets: one excluding molecules present in SIRIUS's pre-training corpus and one excluding molecules present in CFM-ID's, so that each pretrained baseline is evaluated only on unseen molecules. For each target molecule InChIKey, assemble the candidate list from PubChem (cand_dict.pkl format); if training with regularization, also prepare cand_dict_train_updated.pkl. Run each tool on the same filtered subsets and compute rank@1 (and optionally rank@[1-20]) per tool, then compare JESTR's percent improvement — the paper reports 31.12% over SIRIUS and 238.00% over CFM-ID on the filtered subsets. Set the dataset to be used via params.yaml (exp parameter); if you create your own dataset variant, you must update utils.py to load it.

## Related tools

- **JESTR** (Provides the data preparation utilities (dataset.py, utils.py) used to load, preprocess, and filter MassSpecGym subsets, and the candidate-ranking pipeline whose rank@1 is benchmarked) — https://github.com/HassounLab/JESTR1/
- **dataset.py** (Loads and preprocesses the MassSpecGym dataset into JESTR's data structures) — https://github.com/HassounLab/JESTR1/
- **utils.py** (Utility functions for data preparation; must be updated to load custom dataset variants) — https://github.com/HassounLab/JESTR1/
- **params.yaml** (Configuration file setting which dataset (exp) is used and related parameters) — https://github.com/HassounLab/JESTR1/
- **cand_rank_canopus.py** (Template script for running candidate ranking on a complete dataset; adapt it for the MassSpecGym filtered subsets) — https://github.com/HassounLab/JESTR1/
- **MassSpecGym** (The benchmark dataset being downloaded and filtered) — https://huggingface.co/datasets/roman-bushuiev/MassSpecGym

## Examples

```
python cand_rank_canopus.py  # rank candidates for the complete dataset; adapt this file for MassSpecGym filtered subsets per README: 'For your own dataset you can create a file based on this file'
```

## Evaluation signals

- JESTR's rank@1 on the filtered subset excluding SIRIUS's pre-training molecules should exceed SIRIUS's by approximately 31% (paper reports 10.41% vs 13.65%, a 31.12% improvement).
- JESTR's median rank@1 on the filtered subset excluding CFM-ID's pre-training molecules should exceed CFM-ID's by approximately 238% (paper reports 2.00% vs 6.76%, a 238.00% improvement).
- The filtered subsets must exclude all molecules present in the respective baseline tool's pre-training corpus — verify no overlap between subset molecule InChIKeys and the exclusion lists.
- The comparison table (e.g. baseline_comparison.csv) must contain rank@1 entries for all three tools (JESTR, SIRIUS, CFM-ID) on both filtered subsets.
- Output data structures should follow JESTR's expected formats: data_dict.pkl indexed by spectra id with m/z/intensity arrays, metadata, SMILES and InChIKey; split.pkl with 'train', 'valid', 'test' InChIKey lists.

## Limitations

- MassSpecGym and other non-NPLIB1 datasets are under licensing agreements that prohibit their public release; users must ensure they have the datasets under appropriate licenses.
- The pretrained weights released for MassSpecGym were trained on GPU (nVidia A100 with CUDA 11.8); environments must be GPU-capable.
- Pretrained model weights for MassSpecGym are available from data/MassSpecGym in the repo, but some NPLIB1 files are too large for the repo and are on zenodo with download instructions in data/NPLIB1/*.txt — check availability per dataset.
- No tool or library versions are stated in the paper beyond the package versions listed in jestr_requirements.txt; code was tested only on those versions.
- If you create your own dataset variant, you need to update utils.py to load it — filtering logic is not parameterized for arbitrary datasets without code changes.

## Evidence

- [abstract] On subsets of the MassSpecGym dataset, JESTR outperforms SIRIUS by 31% and CFM-ID by 238%.: [evidence span withheld — anchor status: not_found]
- [readme] MassSpecGym downloaded from HuggingFace: "The MassSpecGym dataset can be downloaded from the HuggingFace repository [MassSpecGym](https://huggingface.co/datasets/roman-bushuiev/MassSpecGym)."
- [readme] licensing restriction on other datasets: "The other datasets are under licensing agreements that prohibit their public release. If the user wants to test other datasets, they need to ensure the datasets are available to them under"
- [readme] params.yaml exp parameter: "- exp: dataset to be used. If you create your own dataset, you need to update utils.py to load it"
- [readme] candidate lists downloaded from PubChem: "Each entry is a list of candidates for that molecule downloaded from PubChem. This dictionary is for the test set"
