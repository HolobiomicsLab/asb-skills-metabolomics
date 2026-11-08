---
name: msms-spectrum-preprocessing-peak-filtering
description: Use when raw or externally sourced MS/MS spectra must be prepared as
  inputs for MADGEN (or a similar spectra-conditioned generative model) and are not
  already included in the preprocessed Zenodo dataset pickles.
when_to_use_negative:
- When using the processed CANOPUS or msgym pickles from Zenodo record 15036069 —
  these datasets are already preprocessed (intensity-normalized and peak-filtered),
  so re-filtering would alter the published benchmark conditions.
- When the analysis goal is retrieval ranking or generation evaluation only — this
  skill is an upstream data-preparation step, not a substitute for the contrastive
  ranking or Markov-bridge generation stages.
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:692e4d20ba142a9debd87b7a7fe4d8ffd959a1a7b21dacb605f9757ac1c5f661
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MADGEN
  - Python
  - conda
  - pip
  - wget
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# msms-spectrum-preprocessing-peak-filtering

## Summary

Preprocess tandem mass spectrometry (MS/MS) spectra for MADGEN by normalizing peak intensities and removing low-intensity peaks below a predefined threshold, so downstream scaffold retrieval and spectra-guided molecular generation operate on denoised spectra.

## When to use

Use when raw or externally sourced MS/MS spectra must be prepared as inputs for MADGEN (or a similar spectra-conditioned generative model) and are not already included in the preprocessed Zenodo dataset pickles.

## When NOT to use

- When using the processed CANOPUS or msgym pickles from Zenodo record 15036069 — these datasets are already preprocessed (intensity-normalized and peak-filtered), so re-filtering would alter the published benchmark conditions.
- When the analysis goal is retrieval ranking or generation evaluation only — this skill is an upstream data-preparation step, not a substitute for the contrastive ranking or Markov-bridge generation stages.

## Inputs

- Raw MS/MS spectra with peak lists (m/z, intensity pairs) from NIST23, CANOPUS, MassSpecGym, MoNA, or GNPS
- Predefined intensity threshold value for low-intensity peak removal

## Outputs

- Normalized, noise-filtered MS/MS spectra suitable for scaffold retrieval training (scaffold_retrieval/data/{canopus,massspecgym}) and for generation sampling (data/{msgym,canopus,nist}/raw/*.pkl)

## How to apply

Apply two preprocessing operations to every MS/MS spectrum before training or sampling: (1) normalize peak intensities so spectra are on a comparable scale, and (2) remove peaks whose intensity falls below a predefined threshold to reduce noise. In the published workflow this step is already baked into the processed CANOPUS and MassSpecGym (msgym) pickles distributed on Zenodo record 15036069, so re-processing is only needed when starting from raw spectra or the separately licensed NIST23 data. After preprocessing, spectra feed both the contrastive scaffold-retrieval ranker and the scaffold-conditioned Markov-bridge generator with classifier-free spectral guidance, so any change in the threshold affects both stages and must be documented for reproducibility.

## Evaluation signals

- All peak intensities in the preprocessed spectra fall within a consistent normalized range across datasets.
- No peaks below the predefined intensity threshold remain in any processed spectrum.
- Downstream reproduction check: preprocessed spectra used with the published checkpoints (Zenodo 17115068) should reproduce published metrics such as 49.0% top-1 accuracy on NIST23 with the oracle retriever.
- Preprocessed spectra retain the train/validation/test splits documented in the dataset structure.

## Limitations

- The exact intensity threshold value is described as 'predefined' but is not stated numerically in the article or README, so it must be recovered from the code or the published pickles.
- The NIST23 dataset is commercially available and requires separate licensing from NIST, limiting preprocessing to locally acquired spectra.
- Applying a different threshold than the one used for the published pickles may change generation accuracy relative to reported values.

## Evidence
