---
name: tanimoto-fingerprint-correlation-analysis
description: Use when comparing spectral similarity measures (e.g., DreaMS embedding
  cosine, MS2DeepScore, spectral entropy) against molecular structural similarity
  on tandem MS/MS spectra, or when building a library-retrieval benchmark with precursor
  m/z constraints.
when_to_use_negative:
- When the goal is de novo structure annotation or fingerprint prediction from a single
  spectrum rather than similarity/benchmarking — use DreaMS fine-tuned prediction,
  MIST, or SIRIUS instead.
- When spectra are in negative ionization mode or from multiply charged precursors
  — DreaMS training did not incorporate such data.
- When the task requires MS1 isotopic patterns or adduct distributions — this method
  focuses solely on tandem mass spectra.
tools:
- name: DreaMS
  role: Provides the 1,024-dimensional embeddings whose pairwise cosine similarities
    are benchmarked against Tanimoto structural similarity (zero-shot and after contrastive
    fine-tuning on hard triplets)
  repo: pluskal-lab/DreaMS
- name: MS2DeepScore
  role: Supervised baseline similarity measure; run via its official implementation
    for pairwise spectral similarity on NIST20
  repo: https://github.com/matchms/ms2deepscore
- name: SpectralEntropy
  role: Source of spectral entropy and 43 additional standard spectral similarity
    measures used as the 44-measure baseline comparison
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: MoNA
  role: Source of the fine-tuning spectral library (~5,500 molecules) used to construct
    hard triplets for contrastive fine-tuning
- name: NIST20
  role: Source of the evaluation spectra on which all pairwise similarity measures
    are computed and benchmarked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:7a0e7f150b57cd67af7a3cfabcd688a0d3605b7024e22fffbca611f7be4904ec
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tanimoto-fingerprint-correlation-analysis

## Summary

Benchmarks spectral similarity measures (embeddings, supervised models, and 44 standard measures) by correlating them with Tanimoto similarity of molecular fingerprints and by AUROC for same-molecule/analog retrieval, reproducing the DreaMS Fig. 4a–c comparison.

## When to use

Use when comparing spectral similarity measures (e.g., DreaMS embedding cosine, MS2DeepScore, spectral entropy) against molecular structural similarity on tandem MS/MS spectra, or when building a library-retrieval benchmark with precursor m/z constraints.

## When NOT to use

- When the goal is de novo structure annotation or fingerprint prediction from a single spectrum rather than similarity/benchmarking — use DreaMS fine-tuned prediction, MIST, or SIRIUS instead.
- When spectra are in negative ionization mode or from multiply charged precursors — DreaMS training did not incorporate such data.
- When the task requires MS1 isotopic patterns or adduct distributions — this method focuses solely on tandem mass spectra.

## Inputs

- NIST20 evaluation MS/MS spectra (preprocessed)
- MoNA-derived fine-tuning spectral library (~5,500 molecules with preprocessed MS/MS spectra)
- Molecular fingerprints (e.g., for Tanimoto similarity computation) for each evaluated molecule
- Pretrained DreaMS model weights (Zenodo 10997887 / 13843034)
- Spectra stored in the compact HDF5-based binary format used for deep learning

## Outputs

- CSV table of per-measure Pearson correlations against Tanimoto fingerprint similarity, retrieval AUROC (10-ppm precursor m/z window) and analog AUROC (MCES k = 0..7), reproducing Fig. 4a–c
- Pairwise similarity matrices for each evaluated spectral similarity measure on NIST20

## How to apply

Embed the NIST20 evaluation spectra with the DreaMS encoder (both zero-shot and fine-tuned variants) and compute pairwise cosine similarities in the 1,024-dimensional embedding space. Compute baseline pairwise spectral similarities with MS2DeepScore from its official implementation, plus spectral entropy and 43 additional standard measures from the official SpectralEntropy repository (44 measures in total). Evaluate each measure by Pearson correlation against Tanimoto similarity of the molecules' fingerprints, and by AUROC for classifying same-molecule retrieval pairs restricted to a 10-ppm precursor m/z difference window. For analog search, report AUROC at maximum common edge substructure (MCES) distance thresholds k = 0..7. Decision point: contrastive fine-tuning on ~5,500 MoNA molecules with a triplet margin loss (hard triplets with positive/negative pairs constrained to precursor mass differences within 0.05 Da) is what drives the fine-tuned model beyond the zero-shot embedding quality. Assemble the per-measure metrics into a CSV reproducing the Fig. 4a–c comparison with columns for correlation, retrieval AUROC and analog AUROC per measure.

## Related tools

- **DreaMS** (Provides the 1,024-dimensional embeddings whose pairwise cosine similarities are benchmarked against Tanimoto structural similarity (zero-shot and after contrastive fine-tuning on hard triplets)) — pluskal-lab/DreaMS
- **MS2DeepScore** (Supervised baseline similarity measure; run via its official implementation for pairwise spectral similarity on NIST20) — https://github.com/matchms/ms2deepscore
- **SpectralEntropy** (Source of spectral entropy and 43 additional standard spectral similarity measures used as the 44-measure baseline comparison) — https://github.com/YuanyueLi/SpectralEntropy
- **MoNA** (Source of the fine-tuning spectral library (~5,500 molecules) used to construct hard triplets for contrastive fine-tuning)
- **NIST20** (Source of the evaluation spectra on which all pairwise similarity measures are computed and benchmarked)

## Examples

```
from scipy.stats import pearsonr; r, _ = pearsonr(spectral_similarity_pairs, tanimoto_fingerprint_pairs)
```

## Evaluation signals

- Fine-tuned DreaMS cosine similarity achieves higher Pearson correlation with Tanimoto fingerprint similarity than MS2DeepScore on NIST20.
- Fine-tuned model improves global correlation beyond the zero-shot DreaMS embedding.
- Retrieval AUROC within the 10-ppm precursor m/z difference window for fine-tuned DreaMS substantially exceeds all 44 standard spectral similarity measures.
- All correlations fall in [-1, 1] and all AUROC values in [0, 1]; the assembled CSV matches the measure set and column schema of Fig. 4a–c.
- Analog AUROC increases monotonically as the MCES threshold k is relaxed from 0 toward 7.

## Limitations

- The contrastive fine-tuning evaluation covers only a subset of mass spectra and the full potential of training on raw unannotated spectra remains to be unlocked.
- Tanimoto fingerprint similarity is a proxy for structural similarity; analog retrieval is additionally quantified via MCES distance thresholds rather than fingerprints alone.
- DreaMS disregards MS1 isotopic patterns and adduct distributions, which can affect similarity judgments for molecules distinguishable only at the MS1 level.

## Evidence

- [abstract] Zero-shot cosine similarity outperforms MS2DeepScore: "even before any fine-tuning, cosine similarity in the embedding space outperforms the cutting-edge supervised algorithm MS2DeepScore13"
- [abstract] Fine-tuned DreaMS beats 44 similarity measures: "fine-tuned DreaMS substantially outperforms 44 standard spectral similarity measures9 (Fig. 4b)"
- [discussion] MS2DeepScore official implementation: "the official implementation (https://github.com/matchms/ms2deepscore) of the state-of-the-art MS2DeepScore model13"
- [discussion] SpectralEntropy official implementation: "the implementation of all the methods from the official spectral entropy GitHub repository (https://github.com/YuanyueLi/SpectralEntropy)"
- [abstract] Hard triplet contrastive fine-tuning design: "we disentangle the embeddings of similar molecules through a short but carefully designed contrastive fine-tuning on hard examples"
