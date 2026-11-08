---
name: mces-analog-retrieval-auc-evaluation
description: Use when comparing spectral similarity measures (embedding cosine similarity,
  MS2DeepScore, spectral entropy, dot-product variants) on molecular analog retrieval,
  needing AUROC curves stratified by MCES distance thresholds on paired MS/MS spectra.
when_to_use_negative:
- When the input spectra are outside the paper's scope (e.g., negative ionization
  mode or multiply charged precursor ions), since DreaMS was trained only on positive-mode
  singly charged data
- When the goal is molecular fingerprint prediction or de novo structure annotation
  (MIST/CANOPUS benchmark) rather than similarity-based retrieval ranking
- When MS1 isotopic patterns or adduct distributions should drive the analysis — the
  method focuses solely on tandem mass spectra
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DreaMS
  role: Encoder producing the 1,024-dimensional embedding space whose cosine similarities
    are the candidate similarity measure under evaluation
  repo: github.com__pluskal-lab__DreaMS
- name: MS2DeepScore
  role: Supervised baseline similarity model run on NIST20 from its official implementation
    for head-to-head AUROC/correlation comparison
  repo: https://github.com/matchms/ms2deepscore
- name: SpectralEntropy
  role: Source of spectral entropy and standard spectral similarity measures used
    as baseline similarities on NIST20
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: HDF5
  role: Compact binary storage format for preprocessed spectra used in the fine-tuning
    and evaluation pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:431efe7118de5268470f16fbb69c87f1895544c67555b83cbcf85966b93346f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DreaMS
  - HDF5
  - MS2DeepScore
  - MoNA
  - NIST20
  - SpectralEntropy
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# mces-analog-retrieval-auc-evaluation

## Summary

A benchmark protocol that scores spectral similarity measures by how well they retrieve structural analogs, using AUROC computed at maximum common edge substructure (MCES) distance thresholds k = 0..7 on NIST20 spectra. It was used in the DreaMS paper to show fine-tuned embeddings outperform MS2DeepScore and 44 standard similarity measures.

## When to use

Use when comparing spectral similarity measures (embedding cosine similarity, MS2DeepScore, spectral entropy, dot-product variants) on molecular analog retrieval, needing AUROC curves stratified by MCES distance thresholds on paired MS/MS spectra.

## When NOT to use

- When the input spectra are outside the paper's scope (e.g., negative ionization mode or multiply charged precursor ions), since DreaMS was trained only on positive-mode singly charged data
- When the goal is molecular fingerprint prediction or de novo structure annotation (MIST/CANOPUS benchmark) rather than similarity-based retrieval ranking
- When MS1 isotopic patterns or adduct distributions should drive the analysis — the method focuses solely on tandem mass spectra

## Inputs

- NIST20 evaluation MS/MS spectra (preprocessed)
- MoNA-derived fine-tuning spectral library (~5,500 molecules with preprocessed MS/MS spectra)
- Molecular structures/fingerprints (e.g., SMILES mapped to InChIKeys) for Tanimoto and MCES distance computation
- Pre-trained and fine-tuned DreaMS model weights (Zenodo 10997887 / 13843034)
- Official baseline implementations: MS2DeepScore and SpectralEntropy plus 43 other standard spectral similarity measures

## Outputs

- Per-measure metrics table (CSV) reproducing the Fig. 4a–c comparison: Pearson correlation vs Tanimoto, retrieval AUROC within 10-ppm precursor window, and analog-retrieval AUROC at each MCES threshold k = 0..7
- Pairwise similarity matrices for each evaluated similarity measure

## How to apply

Embed the NIST20 evaluation spectra with the fine-tuned DreaMS encoder (contrastive fine-tuning uses a triplet margin loss on hard triplets drawn from ~5,500 MoNA molecules with positive/negative precursor mass differences constrained within 0.05 Da) and compute pairwise cosine similarities in the 1,024-dimensional embedding space. Compute baseline pairwise similarities on the same spectra with MS2DeepScore, spectral entropy, and 43 additional standard spectral similarity measures from their official implementations. Restrict evaluation pairs to those within a 10-ppm precursor m/z difference, the realistic library-retrieval setting. Score each measure two ways: Pearson correlation against Tanimoto similarity of molecular fingerprints, and AUROC for classifying same-molecule retrieval pairs; additionally compute analog-search AUROC for structural analogs defined by MCES distance thresholds k = 0..7. Assemble all per-measure metrics into a single CSV reproducing the Fig. 4a–c comparison, with columns for each similarity measure and its correlation, retrieval AUROC, and analog AUROC values.

## Related tools

- **DreaMS** (Encoder producing the 1,024-dimensional embedding space whose cosine similarities are the candidate similarity measure under evaluation) — github.com__pluskal-lab__DreaMS
- **MS2DeepScore** (Supervised baseline similarity model run on NIST20 from its official implementation for head-to-head AUROC/correlation comparison) — https://github.com/matchms/ms2deepscore
- **SpectralEntropy** (Source of spectral entropy and standard spectral similarity measures used as baseline similarities on NIST20) — https://github.com/YuanyueLi/SpectralEntropy
- **HDF5** (Compact binary storage format for preprocessed spectra used in the fine-tuning and evaluation pipeline)

## Evaluation signals

- Fine-tuned DreaMS cosine similarity should yield higher Pearson correlation with Tanimoto molecular similarity than MS2DeepScore and the zero-shot DreaMS model
- Analog-retrieval AUROC should be reported at every MCES threshold k = 0..7 so stricter structural-analog definitions (small k) show monotonically harder discrimination
- All retrieval AUROCs must be computed only over candidate pairs within a 10-ppm precursor m/z difference window — including all pairs invalidates the comparison
- Fine-tuned DreaMS should substantially outperform all 44 standard spectral similarity measures in the 10-ppm library retrieval setting (Fig. 4b expectation)
- The output CSV must contain one row/column per similarity measure with correlation, retrieval AUROC, and analog AUROC values matching the Fig. 4a–c layout

## Limitations

- The 10-ppm precursor m/z restriction defines a specific library-retrieval task; performance in unconstrained retrieval across all spectra is not established by this benchmark
- Results depend on DreaMS training scope: only positive ionization mode and singly charged precursor ions were used, and MS1 isotopic patterns and adducts are disregarded
- Only a subset of available mass spectra was used for training, so absolute AUROC values may not transfer to larger or more diverse spectral spaces

## Evidence

- [abstract] Fine-tuned DreaMS outperforms 44 standard spectral similarity measures for library retrieval within 10-ppm precursor m/z difference: "fine-tuned DreaMS substantially outperforms 44 standard spectral similarity measures9 (Fig. 4b)"
- [abstract] Zero-shot DreaMS embedding cosine similarity already outperforms supervised MS2DeepScore: "even before any fine-tuning, cosine similarity in the embedding space outperforms the cutting-edge supervised algorithm MS2DeepScore13"
- [methods] Analog-search performance is scored as AUROC against MCES distance thresholds k = 0..7: [evidence span withheld — anchor status: not_found]
- [methods] Baselines are computed on NIST20 with MS2DeepScore and spectral entropy plus 43 other measures from official implementations: [evidence span withheld — anchor status: not_found]
- [methods] Hard triplet construction constrains positive and negative pairs by precursor mass difference within 0.05 Da: [evidence span withheld — anchor status: not_found]
- [readme] Spectral entropy is an established MS/MS similarity baseline for compound identification: [evidence span withheld — anchor status: external_doc]
