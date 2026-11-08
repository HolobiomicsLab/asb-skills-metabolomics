---
name: tandem-mass-spectra-embedding-evaluation
description: Use when you have embedded a set of tandem mass spectra (e.g. NIST20,
  MoNA) into a vector space and need to quantify whether that space reflects molecular
  structural similarity or supports analog retrieval.
when_to_use_negative:
- The spectra are unannotated and no reference molecular structures/fingerprints exist,
  so Tanimoto or MCES ground-truth similarity cannot be computed
- The goal is de novo spectrum-to-structure annotation (fingerprint prediction) rather
  than evaluating similarity/embedding quality — use fingerprint prediction benchmarks
  such as the MIST retrieval benchmark instead
- The evaluation targets non-positive-mode or multiply charged precursors, outside
  the scope of the reported benchmark
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore
  role: Supervised spectral similarity baseline; embeddings/similarities computed
    from its official implementation for head-to-head comparison
  repo: https://github.com/matchms/ms2deepscore
- name: SpectralEntropy
  role: Source of the 44 standard spectral similarity measures (including spectral
    entropy) used as classic baseline similarities on the evaluation set
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: DreaMS
  role: Embedding model under test; produces 1,024-dimensional embeddings whose zero-shot
    or fine-tuned cosine similarity is evaluated
- name: MoNA
  role: Fine-tuning spectral library of ~5,500 molecules used to create the contrastively
    fine-tuned embedding model before evaluation
- name: NIST20
  role: Evaluation spectral library on which pairwise similarities and retrieval metrics
    are computed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:5c5e649a875752d69055fdd26004752d8691fb0ab2bd0ba9a59867cacb8bd250
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - DreaMS
  - MS2DeepScore
  - MoNA
  - NIST20
  - SpectralEntropy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tandem-mass-spectra-embedding-evaluation

## Summary

Benchmark the quality of MS/MS spectral embeddings by testing whether cosine similarity in the embedding space correlates with molecular structural similarity (Tanimoto of fingerprints) and supports library retrieval, comparing against MS2DeepScore, spectral entropy and other standard spectral similarity measures.

## When to use

Use when you have embedded a set of tandem mass spectra (e.g. NIST20, MoNA) into a vector space and need to quantify whether that space reflects molecular structural similarity or supports analog retrieval.

## When NOT to use

- The spectra are unannotated and no reference molecular structures/fingerprints exist, so Tanimoto or MCES ground-truth similarity cannot be computed
- The goal is de novo spectrum-to-structure annotation (fingerprint prediction) rather than evaluating similarity/embedding quality — use fingerprint prediction benchmarks such as the MIST retrieval benchmark instead
- The evaluation targets non-positive-mode or multiply charged precursors, outside the scope of the reported benchmark

## Inputs

- Embedded evaluation spectra (e.g. NIST20 MS/MS spectra as 1,024-dimensional DreaMS embedding vectors)
- Reference molecular structures/fingerprints for the evaluation compounds (for Tanimoto and MCES computation)
- Precursor m/z metadata for spectra (to apply the 10-ppm precursor difference constraint)
- Baseline spectral similarity scores from MS2DeepScore, SpectralEntropy and other standard similarity measures

## Outputs

- CSV table of per-measure metrics (Pearson correlation vs. Tanimoto, retrieval AUROC within 10-ppm window, analog AUROC at MCES thresholds k = 0..7) reproducing Fig. 4a-c
- Pairwise cosine similarity matrices for the evaluated embedding space
- Ranked comparison of the embedding model against MS2DeepScore and 44 standard spectral similarity measures

## How to apply

Embed the evaluation spectra (e.g. NIST20) with the encoder under test and compute pairwise cosine similarities in the embedding space. Compute baseline pairwise spectral similarities on the same evaluation set using MS2DeepScore (official matchms implementation) and the 44 standard spectral similarity measures including spectral entropy from the official SpectralEntropy repository. Score each similarity measure by Pearson correlation against Tanimoto similarity of the molecules' fingerprints, and by AUROC for classifying same-molecule retrieval pairs restricted to a 10-ppm precursor m/z difference window. Additionally evaluate analog-search performance as AUROC of retrieving structural analogs defined by maximum common edge substructure (MCES) distance thresholds k = 0..7. Assemble the per-measure metrics into a comparison table (as in Fig. 4a-c of the DreaMS paper). The decision point is whether the embedding similarity surpasses the supervised MS2DeepScore baseline and standard measures on correlation and retrieval AUROC; the paper reports that even zero-shot DreaMS cosine similarity outperforms MS2DeepScore, and contrastive fine-tuning on ~5,500 MoNA molecules with hard triplets (precursor mass differences within 0.05 Da) further improves correlation and retrieval.

## Related tools

- **MS2DeepScore** (Supervised spectral similarity baseline; embeddings/similarities computed from its official implementation for head-to-head comparison) — https://github.com/matchms/ms2deepscore
- **SpectralEntropy** (Source of the 44 standard spectral similarity measures (including spectral entropy) used as classic baseline similarities on the evaluation set) — https://github.com/YuanyueLi/SpectralEntropy
- **DreaMS** (Embedding model under test; produces 1,024-dimensional embeddings whose zero-shot or fine-tuned cosine similarity is evaluated)
- **MoNA** (Fine-tuning spectral library of ~5,500 molecules used to create the contrastively fine-tuned embedding model before evaluation)
- **NIST20** (Evaluation spectral library on which pairwise similarities and retrieval metrics are computed)

## Evaluation signals

- The per-measure CSV contains correlation, retrieval AUROC, and analog AUROC columns for every similarity measure compared (Fig. 4a-c reproduction)
- Zero-shot DreaMS cosine similarity should already exceed MS2DeepScore's correlation with Tanimoto similarity; if it does not, the embedding or evaluation pipeline is suspect
- Retrieval AUROC is computed only over pairs within a 10-ppm precursor m/z difference window — verify the constraint was applied, not all-pairs retrieval
- Analog AUROC values should be reported across the full range of MCES distance thresholds k = 0..7
- Baseline similarities must come from the official implementations (ms2deepscore and SpectralEntropy repos), not re-implementations

## Limitations

- Fingerprint-prediction fine-tuning performance is a proof of concept, not production-ready ('should be understood as a proof of concept that our approach can achieve competitive performance with MIST')
- The retrieval benchmark is restricted to the 10-ppm precursor m/z difference setting; performance in analog retrieval for larger mass windows or unfiltered retrieval is not established
- DreaMS was trained on positive-mode, singly charged precursor spectra; evaluation transferability beyond that regime is unverified

## Evidence

- [abstract] Zero-shot embedding superiority claim: "even before any fine-tuning, cosine similarity in the embedding space outperforms the cutting-edge supervised algorithm MS2DeepScore13"
- [abstract] Fine-tuned model outperforms standard measures: "fine-tuned DreaMS substantially outperforms 44 standard spectral similarity measures9 (Fig. 4b)"
- [discussion] Baseline via official SpectralEntropy implementation: "the implementation of all the methods from the official spectral entropy GitHub repository (https://github.com/YuanyueLi/SpectralEntropy)"
- [discussion] Baseline via official MS2DeepScore implementation: "the official implementation (https://github.com/matchms/ms2deepscore) of the state-of-the-art MS2DeepScore model13"
- [abstract] Fine-tuning context for the improved embedding space: "we disentangle the embeddings of similar molecules through a short but carefully designed contrastive fine-tuning on hard examples"
