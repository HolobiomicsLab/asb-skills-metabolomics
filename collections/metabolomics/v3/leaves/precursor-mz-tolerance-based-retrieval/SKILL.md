---
name: precursor-mz-tolerance-based-retrieval
description: Use when evaluating whether a spectral similarity metric (embeddings,
  supervised models, or 44 classical measures) retrieves the correct molecular structure
  from a spectral library, with candidates restricted by precursor m/z tolerance such
  as 10 ppm.
when_to_use_negative:
- Task is de novo structure annotation or fingerprint prediction (use MIST/SIRIUS-style
  annotation skills) rather than library retrieval
- Precursor m/z is unavailable, low-accuracy, or the instrument's m/z accuracy makes
  a 10-ppm window meaningless
- Evaluating spectra outside the paper's scope, e.g., negative ionization mode or
  multiply charged precursor ions
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DreaMS
  role: Provides the embedding encoder whose cosine similarities are benchmarked for
    precursor-constrained library retrieval
  repo: https://github.com/pluskal-lab/DreaMS
- name: MS2DeepScore
  role: Supervised spectral-similarity baseline for the retrieval comparison
  repo: https://github.com/matchms/ms2deepscore
- name: SpectralEntropy
  role: Official implementation of the 44 standard spectral similarity measures used
    as baselines
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:48a226983eeee598972e2556ad96e502782f1cab5986d1a7b51587936f4ceaca
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# precursor-mz-tolerance-based-retrieval

## Summary

Benchmark protocol for library retrieval of MS/MS spectra in which candidate correct matches are constrained to a precursor m/z window (10 ppm in the DreaMS paper), scoring each spectral similarity measure by same-molecule retrieval AUROC and analog AUROC at MCES distance thresholds.

## When to use

Use when evaluating whether a spectral similarity metric (embeddings, supervised models, or 44 classical measures) retrieves the correct molecular structure from a spectral library, with candidates restricted by precursor m/z tolerance such as 10 ppm.

## When NOT to use

- Task is de novo structure annotation or fingerprint prediction (use MIST/SIRIUS-style annotation skills) rather than library retrieval
- Precursor m/z is unavailable, low-accuracy, or the instrument's m/z accuracy makes a 10-ppm window meaningless
- Evaluating spectra outside the paper's scope, e.g., negative ionization mode or multiply charged precursor ions

## Inputs

- MS/MS spectra with precursor m/z values (e.g., NIST20 evaluation set; MoNA-derived fine-tuning library of ~5,500 molecules)
- Pre-trained and contrastively fine-tuned DreaMS transformer embeddings
- Baseline spectral similarity scores from MS2DeepScore and SpectralEntropy (44 measures)
- Molecular fingerprints / structures for Tanimoto and MCES distance computation

## Outputs

- Per-measure table (CSV) of Pearson correlation vs Tanimoto similarity, same-molecule retrieval AUROC within 10-ppm precursor m/z difference, and analog-retrieval AUROC at MCES thresholds k = 0..7
- Fig. 4a–c-style comparison of DreaMS vs MS2DeepScore, spectral entropy, and 43 other similarity measures

## How to apply

Embed query and library spectra (NIST20 evaluation set) with a spectral encoder — e.g., the fine-tuned DreaMS transformer — and compute pairwise cosine similarities in the 1,024-dimensional embedding space. Compute baseline pairwise similarities for comparison using MS2DeepScore and the spectral entropy suite of 44 standard similarity measures from their official implementations. Restrict the retrieval task to spectra whose precursor m/z values differ by within 10 ppm, and score each measure by AUROC for classifying same-molecule pairs within that window. For analog search, additionally evaluate AUROC of retrieving structural analogs defined by maximum common edge substructure (MCES) distance thresholds k = 0..7. Assemble per-measure correlation, retrieval AUROC, and analog AUROC values into a CSV reproducing the Fig. 4a–c comparison.

## Related tools

- **DreaMS** (Provides the embedding encoder whose cosine similarities are benchmarked for precursor-constrained library retrieval) — https://github.com/pluskal-lab/DreaMS
- **MS2DeepScore** (Supervised spectral-similarity baseline for the retrieval comparison) — https://github.com/matchms/ms2deepscore
- **SpectralEntropy** (Official implementation of the 44 standard spectral similarity measures used as baselines) — https://github.com/YuanyueLi/SpectralEntropy

## Evaluation signals

- Every retrieval candidate pair is filtered to precursor m/z difference within 10 ppm before computing AUROC
- Correlations are computed against Tanimoto similarity of molecular fingerprints, not raw labels
- Analog AUROC is reported for the full sweep of MCES distance thresholds k = 0..7
- Fine-tuned DreaMS AUROC should exceed all 44 standard similarity measures and MS2DeepScore, and zero-shot DreaMS cosine similarity should already beat MS2DeepScore
- Output CSV columns include correlation, retrieval AUROC, and analog AUROC for each similarity measure, matching the Fig. 4a–c layout

## Limitations

- The 10-ppm precursor window assumes high-resolution, accurate precursor m/z; retrieval quality degrades on low-accuracy instruments
- Benchmark covers positive-mode, singly charged precursor spectra as in the DreaMS training/evaluation setup
- Analog-search AUROC depends on the chosen MCES distance threshold k, so conclusions are threshold-dependent

## Evidence

- [abstract] fine-tuned retrieval outperforms baselines: "fine-tuned DreaMS substantially outperforms 44 standard spectral similarity measures9 (Fig. 4b)"
- [abstract] zero-shot cosine similarity beats MS2DeepScore: "even before any fine-tuning, cosine similarity in the embedding space outperforms the cutting-edge supervised algorithm MS2DeepScore13"
- [abstract] embedding space organized by structural similarity: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules and are robust to mass spectrometry conditions"
- [readme] SpectralEntropy is the official source of the baseline measures: [evidence span withheld — anchor status: not_found]
