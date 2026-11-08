---
name: triplet-margin-contrastive-fine-tuning
description: Use when zero-shot self-supervised MS/MS embeddings (or supervised baselines
  like MS2DeepScore) inadequately separate structurally similar molecules — e.g. you
  need a learned spectral similarity for library retrieval within a narrow precursor
  m/z window or analog search by MCES distance.
when_to_use_negative:
- The goal is molecular fingerprint prediction or fluorine detection from spectra
  — those are separate supervised fine-tuning tasks of DreaMS, not the similarity-tuned
  embedding space
- Only a general-purpose zero-shot representation is needed and no paired precursor-mass-constrained
  library is available — the zero-shot DreaMS embeddings already outperform MS2DeepScore
  before any fine-tuning
- Input spectra are outside DreaMS's scope (e.g. negative ionization mode, multiply
  charged precursors, or reliance on MS1 isotopic patterns / adduct distributions),
  which the method disregards
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DreaMS
  role: Pre-trained 116M-parameter transformer that is fine-tuned with triplet margin
    loss on hard triplets; its 1,024-dim embeddings supply the similarity space
  repo: https://github.com/pluskal-lab/DreaMS
- name: MS2DeepScore
  role: Supervised spectral similarity baseline that the fine-tuned embedding similarity
    must exceed in correlation with Tanimoto similarity
  repo: https://github.com/matchms/ms2deepscore
- name: SpectralEntropy
  role: Provides the official implementations of spectral entropy and additional standard
    spectral similarity measures used as the 44-method comparison baselines
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:f2ac4f669b359f2a7994840d6ac9c27ef46e49eb148d7f1ffcfceccd2827b348
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - MS2DeepScore
  - MoNA
  - NIST20
  - SpectralEntropy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# triplet-margin-contrastive-fine-tuning

## Summary

Disentangles the embeddings of structurally similar molecules in DreaMS's 1,024-dim representation space via a short contrastive fine-tuning on hard triplet examples drawn from a MoNA-derived library of ~5,500 molecules, so that embedding cosine similarity better predicts Tanimoto structural similarity than zero-shot embeddings, MS2DeepScore, or 44 standard spectral similarity measures.

## When to use

Use when zero-shot self-supervised MS/MS embeddings (or supervised baselines like MS2DeepScore) inadequately separate structurally similar molecules — e.g. you need a learned spectral similarity for library retrieval within a narrow precursor m/z window or analog search by MCES distance.

## When NOT to use

- The goal is molecular fingerprint prediction or fluorine detection from spectra — those are separate supervised fine-tuning tasks of DreaMS, not the similarity-tuned embedding space
- Only a general-purpose zero-shot representation is needed and no paired precursor-mass-constrained library is available — the zero-shot DreaMS embeddings already outperform MS2DeepScore before any fine-tuning
- Input spectra are outside DreaMS's scope (e.g. negative ionization mode, multiply charged precursors, or reliance on MS1 isotopic patterns / adduct distributions), which the method disregards

## Inputs

- Pre-trained DreaMS transformer weights (116M parameters; Zenodo 10997887 / 13843034)
- Fine-tuning spectral library derived from MoNA (~5,500 molecules with preprocessed MS/MS spectra)
- NIST20 evaluation spectra for benchmarking
- Baseline pairwise similarities from MS2DeepScore and spectral entropy (plus 43 additional standard spectral similarity measures)
- Molecular fingerprints (e.g. Morgan/ECFP-style) of library compounds for Tanimoto similarity targets

## Outputs

- Fine-tuned DreaMS encoder whose embedding cosine similarity reflects Tanimoto structural similarity
- Pairwise cosine similarity matrix over NIST20 evaluation spectra
- CSV metric table reproducing Fig. 4a–c with per-measure correlation, retrieval AUROC and analog AUROC columns

## How to apply

Start from the 116-million-parameter DreaMS transformer pre-trained self-supervised on GeMS spectra (masked peak prediction and retention order prediction) rather than training from scratch. Load the fine-tuning spectral library derived from MoNA (~5,500 molecules with preprocessed MS/MS spectra), and construct hard triplets in which positive and negative pairs are constrained to precursor mass differences within 0.05 Da — the hardness of negatives is the key design choice that forces the model to disentangle otherwise-similar embeddings. Fine-tune with a triplet margin loss for a short schedule, then embed evaluation spectra (e.g. NIST20) and compute pairwise cosine similarities in the resulting embedding space. Benchmark against baselines by Pearson correlation to Tanimoto fingerprint similarity, AUROC for same-molecule retrieval within a 10-ppm precursor m/z difference, and analog-retrieval AUROC at maximum common edge substructure (MCES) distance thresholds k = 0..7. Assemble per-measure metrics into a CSV reproducing the Fig. 4a–c comparison table.

## Related tools

- **DreaMS** (Pre-trained 116M-parameter transformer that is fine-tuned with triplet margin loss on hard triplets; its 1,024-dim embeddings supply the similarity space) — https://github.com/pluskal-lab/DreaMS
- **MS2DeepScore** (Supervised spectral similarity baseline that the fine-tuned embedding similarity must exceed in correlation with Tanimoto similarity) — https://github.com/matchms/ms2deepscore
- **SpectralEntropy** (Provides the official implementations of spectral entropy and additional standard spectral similarity measures used as the 44-method comparison baselines) — https://github.com/YuanyueLi/SpectralEntropy

## Evaluation signals

- Pearson correlation between embedding cosine similarity and Tanimoto similarity of molecular fingerprints must exceed both the zero-shot DreaMS model and MS2DeepScore
- Retrieval AUROC for same-molecule pairs within a 10-ppm precursor m/z difference window must substantially outperform all 44 standard spectral similarity measures
- Analog-search AUROC evaluated across MCES distance thresholds k = 0..7 should improve over the zero-shot model
- Embeddings must remain 1,024-dimensional real-valued vectors organized by structural similarity after fine-tuning
- Hard triplets must be constructed with positive/negative precursor mass differences constrained to within 0.05 Da — verify triplet construction before training

## Limitations

- Fine-tuning used a library of only ~5,500 MoNA molecules; the standard training spectral libraries MoNA and NIST20 cover only a limited subset of known natural molecules
- The method focuses solely on tandem mass spectra, disregarding other important features such as MS1 isotopic patterns or adduct distributions
- Training used only a subset of available mass spectra, and data beyond positive ionization mode or singly charged precursor ions were not incorporated, so the full potential remains to be unlocked

## Evidence

- [abstract] hard triplet construction rationale: "we disentangle the embeddings of similar molecules through a short but carefully designed contrastive fine-tuning on hard examples"
- [abstract] zero-shot baseline must be beaten: "even before any fine-tuning, cosine similarity in the embedding space outperforms the cutting-edge supervised algorithm MS2DeepScore13"
- [abstract] benchmark target vs 44 measures: "fine-tuned DreaMS substantially outperforms 44 standard spectral similarity measures9 (Fig. 4b)"
- [discussion] MS2DeepScore official implementation baseline: "the official implementation (https://github.com/matchms/ms2deepscore) of the state-of-the-art MS2DeepScore model13"
- [readme] contrastive learning for embeddings in adjacent tool MIST: [evidence span withheld — anchor status: external_doc]
