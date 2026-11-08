---
name: spectral-similarity-benchmarking
description: Use when evaluating whether a spectral embedding or similarity measure
  reflects molecular structural similarity, or when comparing library/analog retrieval
  performance on NIST20 spectra against baselines like MS2DeepScore.
when_to_use_negative:
- The task is fingerprint-prediction-based structure retrieval benchmarking (MIST
  CANOPUS benchmark) rather than pairwise spectral similarity evaluation
- Spectra come from a domain the model was not designed for, e.g. negative ionization
  mode or multiply charged precursors, which DreaMS training did not cover
edam_operation: http://edamontology.org/operation_0405
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore
  role: Supervised baseline similarity measure computed from its official implementation
    on NIST20
  repo: https://github.com/matchms/ms2deepscore
- name: SpectralEntropy
  role: Provides spectral entropy and the additional standard spectral similarity
    measures used as baselines
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: DreaMS
  role: Produces 1,024-dimensional embeddings; cosine similarity in this space (zero-shot
    or fine-tuned) is the measure under evaluation
  repo: https://github.com/pluskal-lab/DreaMS
- name: HDF5
  role: Compact binary format in which spectral libraries used for fine-tuning and
    evaluation are stored
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:06fcee57bce86c8a727b07c3a7b7b2f2699f19f1e02b1f932f8b074644ace436
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - DreaMS
  - HDF5
  - MS2DeepScore
  - MoNA
  - NIST20
  - SpectralEntropy
  edam_operation: http://edamontology.org/operation_0405
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-similarity-benchmarking

## Summary

Benchmark MS/MS spectral similarity measures (DreaMS embeddings, MS2DeepScore, spectral entropy and 44 standard measures) by correlating them with Tanimoto fingerprint similarity and by AUROC for same-molecule and analog retrieval, reproducing the DreaMS Fig. 4 comparison.

## When to use

Use when evaluating whether a spectral embedding or similarity measure reflects molecular structural similarity, or when comparing library/analog retrieval performance on NIST20 spectra against baselines like MS2DeepScore.

## When NOT to use

- The task is fingerprint-prediction-based structure retrieval benchmarking (MIST CANOPUS benchmark) rather than pairwise spectral similarity evaluation
- Spectra come from a domain the model was not designed for, e.g. negative ionization mode or multiply charged precursors, which DreaMS training did not cover

## Inputs

- NIST20 evaluation MS/MS spectra (preprocessed)
- MoNA fine-tuning spectral library (~5,500 molecules)
- Molecular fingerprints (e.g. from structures) for Tanimoto similarity
- Official MS2DeepScore and SpectralEntropy implementations

## Outputs

- CSV table of per-measure metrics: Pearson correlation vs Tanimoto, retrieval AUROC (10-ppm precursor window), analog AUROC for MCES thresholds k = 0..7 (Fig. 4a–c reproduction)

## How to apply

Embed NIST20 evaluation spectra with the DreaMS encoder (zero-shot or after contrastive fine-tuning on ~5,500 MoNA molecules with triplet margin loss using hard triplets constrained to precursor mass differences within 0.05 Da) and compute pairwise cosine similarities in the embedding space. Compute baselines with MS2DeepScore and spectral entropy plus 43 additional standard spectral similarity measures from their official implementations. Score each measure by Pearson correlation against Tanimoto similarity of molecular fingerprints, by AUROC for classifying same-molecule retrieval pairs within a 10-ppm precursor m/z difference window, and by AUROC for retrieving structural analogs defined by maximum common edge substructure (MCES) distance thresholds k = 0..7. Assemble per-measure metrics into a CSV reproducing the Fig. 4a–c comparison.

## Related tools

- **MS2DeepScore** (Supervised baseline similarity measure computed from its official implementation on NIST20) — https://github.com/matchms/ms2deepscore
- **SpectralEntropy** (Provides spectral entropy and the additional standard spectral similarity measures used as baselines) — https://github.com/YuanyueLi/SpectralEntropy
- **DreaMS** (Produces 1,024-dimensional embeddings; cosine similarity in this space (zero-shot or fine-tuned) is the measure under evaluation) — https://github.com/pluskal-lab/DreaMS
- **HDF5** (Compact binary format in which spectral libraries used for fine-tuning and evaluation are stored)

## Examples

```
cos_sim = cosine_similarity(dreams_embed(nist20_spectra)); pearsonr(cos_sim.ravel(), tanimoto_fp_sim.ravel())  # then AUROC at 10 ppm precursor window
```

## Evaluation signals

- Fine-tuned DreaMS similarity shows higher Pearson correlation with Tanimoto molecular similarity than MS2DeepScore
- Fine-tuned DreaMS correlation exceeds that of the zero-shot DreaMS model, confirming fine-tuning improves global correlation
- Fine-tuned DreaMS AUROC for retrieval within a 10-ppm precursor m/z difference substantially exceeds all 44 standard spectral similarity measures
- Analog-search AUROC curve across MCES thresholds k = 0..7 is completed for every measure and populated in the output CSV
- Output CSV columns cover all similarity measures with correlation, retrieval AUROC and analog AUROC values as in Fig. 4a–c

## Limitations

- Contrastive fine-tuning on ~5,500 MoNA molecules is task-specific; gains may not transfer to all similarity regimes
- DreaMS focuses solely on tandem mass spectra, disregarding MS1 isotopic patterns and adduct distributions
- Training/evaluation coverage is limited to positive ionization mode and singly charged precursor ions; spectra beyond this were not incorporated

## Evidence

- [methods] hard triplets: [evidence span withheld — anchor status: not_found]
- [methods] retrieval AUROC: [evidence span withheld — anchor status: not_found]
- [methods] MCES analog AUROC: [evidence span withheld — anchor status: not_found]
- [abstract] outperforms 44 measures: "fine-tuned DreaMS substantially outperforms 44 standard spectral similarity measures9 (Fig. 4b)"
- [discussion] baseline implementations: "the implementation of all the methods from the official spectral entropy GitHub repository (https://github.com/YuanyueLi/SpectralEntropy)"
