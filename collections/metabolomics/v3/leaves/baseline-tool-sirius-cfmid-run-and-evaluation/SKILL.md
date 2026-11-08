---
name: baseline-tool-sirius-cfmid-run-and-evaluation
description: Use when evaluating a new metabolomics annotation/ranking method against
  pretrained baselines (SIRIUS, CFM-ID) and you need leakage-controlled test subsets
  so each baseline is only scored on molecules unseen in its own pre-training.
when_to_use_negative:
- When the goal is to evaluate JESTR on the full unfiltered MassSpecGym test set rather
  than leakage-controlled subsets — the subsets must exclude each baseline's pre-training
  molecules or the comparison is invalid.
- 'When evaluating JESTR on NPLIB1 or a custom dataset: the README notes other datasets
  are under licensing agreements prohibiting public release, so the MassSpecGym-based
  SIRIUS/CFM-ID subset construction applies only where you have legal access to the
  relevant data.'
- When a baseline other than SIRIUS or CFM-ID is being compared, as the filtered subsets
  are specific to those two tools' pre-training corpora.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS
  role: Pretrained baseline annotation/ranking tool evaluated on the SIRIUS-filtered
    MassSpecGym subset to compute its rank@1
- name: CFM-ID
  role: Pretrained baseline annotation/ranking tool evaluated on the CFM-ID-filtered
    MassSpecGym subset to compute its rank@1
- name: JESTR
  role: Method under test; ranks candidates by cosine similarity between query-spectrum
    and candidate-molecule embeddings in a joint embedding space
  repo: https://github.com/HassounLab/JESTR1/
- name: dataset.py / utils.py
  role: Data preparation utilities used to build the filtered test subsets that exclude
    SIRIUS and CFM-ID pre-training molecules
  repo: https://github.com/HassounLab/JESTR1/
- name: cand_rank_canopus.py / JESTR.ipynb
  role: Ranking pipeline used to produce JESTR candidate rankings on the filtered
    MassSpecGym subsets with pretrained weights
  repo: https://github.com/HassounLab/JESTR1/blob/main/JESTR.ipynb
- name: params.yaml
  role: Configuration file setting dataset, batch sizes, epochs, learning rates, early
    stopping, and pretrained weight path
  repo: https://github.com/HassounLab/JESTR1/
- name: PyTorch
  role: Framework under which all JESTR code runs for training and ranking
  repo: https://pytorch.org
- name: Python
  role: Runtime language for the JESTR scripts and utilities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf354
    title: 'JESTR: Joint Embedding Space Technique for Ranking candidate molecules
      for the annotation of untargeted metabolomics data'
  source_content_hash: sha256:6757e4af47d6627cc56c0fa830d492e6014f3ee71c6c578dd548b11a2a82b6c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/JESTR1
  tools:
  - CFM-ID
  - JESTR
  - PyTorch
  - Python
  - SIRIUS
  - cand_rank_canopus.py / JESTR.ipynb
  - dataset.py / utils.py
  - params.yaml
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# baseline-tool-sirius-cfmid-run-and-evaluation

## Summary

Benchmark JESTR against the pretrained SIRIUS and CFM-ID annotation tools on MassSpecGym test subsets that exclude each tool's pre-training molecules, computing rank@1 for each tool and quantifying JESTR's percent improvement. This establishes a fair head-to-head baseline comparison by controlling for data leakage from each baseline's training corpus.

## When to use

Use when evaluating a new metabolomics annotation/ranking method against pretrained baselines (SIRIUS, CFM-ID) and you need leakage-controlled test subsets so each baseline is only scored on molecules unseen in its own pre-training.

## When NOT to use

- When the goal is to evaluate JESTR on the full unfiltered MassSpecGym test set rather than leakage-controlled subsets — the subsets must exclude each baseline's pre-training molecules or the comparison is invalid.
- When evaluating JESTR on NPLIB1 or a custom dataset: the README notes other datasets are under licensing agreements prohibiting public release, so the MassSpecGym-based SIRIUS/CFM-ID subset construction applies only where you have legal access to the relevant data.
- When a baseline other than SIRIUS or CFM-ID is being compared, as the filtered subsets are specific to those two tools' pre-training corpora.

## Inputs

- MassSpecGym dataset downloaded from the HuggingFace repository (roman-bushuiev/MassSpecGym)
- Lists of molecule InChIKeys present in SIRIUS's pre-training corpus
- Lists of molecule InChIKeys present in CFM-ID's pre-training corpus
- Query mass spectra (m/z and intensity arrays with metadata) from the MassSpecGym test split
- JESTR data preparation utilities: dataset.py, utils.py (from HassounLab/JESTR1)
- params.yaml configuration file specifying the dataset and runtime parameters

## Outputs

- Per-subset rank@1 comparison table (e.g. baseline_comparison.csv) with rank@1 entries for SIRIUS, CFM-ID, and JESTR on each filtered subset
- Percent improvement of JESTR over SIRIUS on the SIRIUS-filtered subset (~31%)
- Percent improvement of JESTR over CFM-ID on the CFM-ID-filtered subset (~238%)
- Candidate molecule rankings per query spectrum from SIRIUS, CFM-ID, and JESTR on the filtered subsets

## How to apply

Download the MassSpecGym dataset from the HuggingFace repository (roman-bushuiev/MassSpecGym) and use the JESTR data preparation utilities (dataset.py / utils.py) to construct filtered test subsets that exclude molecules present in SIRIUS's and CFM-ID's pre-training corpora, so that neither baseline is scored on data it has seen. Run SIRIUS and CFM-ID with their publicly available pretrained models on these filtered subsets to obtain candidate rankings for each query spectrum, and run the JESTR ranking pipeline (cand_rank_canopus.py or JESTR.ipynb with pretrained weights) on the identical subsets. Compute rank@1 for each tool on each subset and express JESTR's improvement as a percentage relative to each baseline. The paper reports JESTR outperforms SIRIUS by 31% and CFM-ID by 238% on the respective filtered subsets — reproducing figures in this range validates the comparison. The expected targets are SIRIUS 10.41% vs JESTR 13.65% rank@1 (31.12% improvement) and CFM-ID 2.00% vs JESTR median 6.76% (238.00% improvement).

## Related tools

- **SIRIUS** (Pretrained baseline annotation/ranking tool evaluated on the SIRIUS-filtered MassSpecGym subset to compute its rank@1)
- **CFM-ID** (Pretrained baseline annotation/ranking tool evaluated on the CFM-ID-filtered MassSpecGym subset to compute its rank@1)
- **JESTR** (Method under test; ranks candidates by cosine similarity between query-spectrum and candidate-molecule embeddings in a joint embedding space) — https://github.com/HassounLab/JESTR1/
- **dataset.py / utils.py** (Data preparation utilities used to build the filtered test subsets that exclude SIRIUS and CFM-ID pre-training molecules) — https://github.com/HassounLab/JESTR1/
- **cand_rank_canopus.py / JESTR.ipynb** (Ranking pipeline used to produce JESTR candidate rankings on the filtered MassSpecGym subsets with pretrained weights) — https://github.com/HassounLab/JESTR1/blob/main/JESTR.ipynb
- **params.yaml** (Configuration file setting dataset, batch sizes, epochs, learning rates, early stopping, and pretrained weight path) — https://github.com/HassounLab/JESTR1/
- **PyTorch** (Framework under which all JESTR code runs for training and ranking) — https://pytorch.org
- **Python** (Runtime language for the JESTR scripts and utilities)

## Examples

```
python cand_rank_canopus.py  # rank candidates for the filtered MassSpecGym subsets with pretrained JESTR weights
```

## Evaluation signals

- Reproduced JESTR rank@1 on the SIRIUS-filtered subset should be ~13.65% vs SIRIUS ~10.41%, i.e. a ~31% relative improvement.
- Reproduced JESTR median rank@1 on the CFM-ID-filtered subset should be ~6.76% vs CFM-ID ~2.00%, i.e. a ~238% relative improvement.
- The filtered subsets must exclude every molecule InChIKey found in SIRIUS's and CFM-ID's pre-training corpora respectively — verify by checking that no filtered-subset InChIKey appears in the corresponding pre-training list.
- The output CSV must contain rank@1 entries for all three tools (SIRIUS, CFM-ID, JESTR) on the filtered subsets.
- JESTR rankings must come from the same pretrained weights used for NPLIB1/MassSpecGym, loaded via the pretrained path in params.yaml, and the environment must be GPU-enabled since released weights are GPU-trained.

## Limitations

- The MassSpecGym dataset must be obtained from HuggingFace; the NPLIB1 dataset is the only one released with pretrained weights, and other datasets are under licensing agreements prohibiting public release.
- The released JESTR weights are GPU-trained (nVidia A100 with CUDA 11.8), so the environment must be set up for GPU for faithful comparison.
- Only SIRIUS and CFM-ID are covered by the filtered-subset design; extending to other baselines requires building analogous exclusion lists from each baseline's pre-training data.
- The paper does not state tool/library versions for SIRIUS or CFM-ID, which may affect reproducibility of their rank@1 numbers.

## Evidence

- [abstract] JESTR outperforms these tools by 31% and 238%, respectively.: "JESTR outperforms these tools by 31% and 238%, respectively."
- [abstract] publicly available pretrained models of SIRIUS and CFM-ID: "publicly available pretrained models of SIRIUS and CFM-ID"
- [readme] The MassSpecGym dataset can be downloaded from the HuggingFace repository: "The MassSpecGym dataset can be downloaded from the HuggingFace repository"
- [readme] dataset.py - load and preprocess dataset: "dataset.py - load and preprocess dataset"
