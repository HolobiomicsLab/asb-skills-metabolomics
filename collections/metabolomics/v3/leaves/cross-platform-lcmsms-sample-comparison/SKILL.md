---
name: cross-platform-lcmsms-sample-comparison
description: Use when LC-MS/MS samples were profiled on different instruments (e.g.
  Q-Exactive Plus vs Q-Exactive Focus) or with different LC gradients, exhibit strong
  RT shifts or poor feature overlap, and you need to cluster/compare them together.
when_to_use_negative:
- When all samples were acquired on the same platform with comparable gradients and
  standard feature alignment plus conventional clustering (e.g. Bray-Curtis on a feature
  table) is already reliable
- When samples lack MS2 fragmentation spectra — MEMO fingerprints are built from MS2
  peaks and neutral losses, not MS1 features
- When the analysis requires retention-time-resolved information, since MEMO deliberately
  performs a retention time agnostic alignment
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: memo-ms
  role: Python package performing MS2 sample vectorization (SpectraDocuments) and
    MemoMatrix fingerprint alignment; installed via pip in a Python 3.8 conda environment
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Handles the MS2 spectra import/processing underlying fingerprint generation
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Converts MS2 spectra into documents used to count peaks and neutral losses
    for the sample fingerprints
  repo: https://github.com/iomega/spec2vec
- name: numpy
  role: Dependency installed alongside memo-ms
- name: scikit-bio
  role: Installed in the MEMO environment for downstream ordination/distance analyses
    of the aligned matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:87fc5572f35a0476568a1b25f77c1b7335a515c711f4d92be124fc0601de43f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - Python
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  - spec2vec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# cross-platform-lcmsms-sample-comparison

## Summary

Vectorize LC-MS/MS samples into MS2 fingerprints (counts of MS2 peaks and neutral losses to the precursor) using MEMO, then align and co-cluster samples acquired on different LC gradients and MS instruments — e.g. 40 min QE-Plus vs 8 min QE-Focus runs — without any prior feature alignment step.

## When to use

Use when LC-MS/MS samples were profiled on different instruments (e.g. Q-Exactive Plus vs Q-Exactive Focus) or with different LC gradients, exhibit strong RT shifts or poor feature overlap, and you need to cluster/compare them together.

## When NOT to use

- When all samples were acquired on the same platform with comparable gradients and standard feature alignment plus conventional clustering (e.g. Bray-Curtis on a feature table) is already reliable
- When samples lack MS2 fragmentation spectra — MEMO fingerprints are built from MS2 peaks and neutral losses, not MS1 features
- When the analysis requires retention-time-resolved information, since MEMO deliberately performs a retention time agnostic alignment

## Inputs

- Feature tables and MS2 spectra files (e.g. from MassIVE accession MSV000088521) for each sample, including samples from different LC gradients and MS instruments
- Blank runs/peak lists (for optional blank filtering)
- Existing MEMO fingerprint matrix of the reference sample set (e.g. plant dataset), if appending new samples

## Outputs

- MemoMatrix: aligned MS2-fingerprint sample matrix (samples × peaks/losses counts) with new samples appended
- UMAP/TMAP (or MDS/PCoA, Heatmap) embeddings and plots showing sample co-clustering
- Distance/dissimilarity structure between old and new samples usable for downstream clustering

## How to apply

Create a dedicated conda environment (Python 3.8) and install memo-ms with its numpy dependency. Load feature tables/spectra (e.g. from MassIVE MSV000088521) with memo_ms.import_data, then use matchms/spec2vec to convert each sample's MS2 spectra into documents, counting the occurrence of MS2 peaks and neutral losses (to the precursor) to build the MS2 fingerprint of each sample. Align fingerprints RT-agnostically into a MemoMatrix, appending new sample vectors to an existing MEMO matrix with no prior feature alignment step — this is what makes batch/platform-agnostic comparison possible. Optionally filter the fingerprints (e.g. remove peaks/losses attributed to blanks) before merging. Project the merged matrix with UMAP or TMAP (or MDS/PCoA, Heatmap) to visualize whether samples such as Waltheria indica 2014/2018 extracts co-cluster with the trypanocidal Melochia umbellata green stems extract.

## Related tools

- **memo-ms** (Python package performing MS2 sample vectorization (SpectraDocuments) and MemoMatrix fingerprint alignment; installed via pip in a Python 3.8 conda environment) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Handles the MS2 spectra import/processing underlying fingerprint generation) — https://github.com/matchms/matchms
- **spec2vec** (Converts MS2 spectra into documents used to count peaks and neutral losses for the sample fingerprints) — https://github.com/iomega/spec2vec
- **numpy** (Dependency installed alongside memo-ms)
- **scikit-bio** (Installed in the MEMO environment for downstream ordination/distance analyses of the aligned matrix)

## Examples

```
conda create --name memo python=3.8 && conda activate memo && pip install numpy && pip install memo-ms
```

## Evaluation signals

- The merged MemoMatrix contains the appended new samples (e.g. the three Waltheria indica 2014/2018 columns/rows) alongside the original plant dataset fingerprints
- UMAP and TMAP visualizations show all newly vectorized extracts co-clustering with the expected reference sample (Melochia umbellata green stems extract)
- No prior feature alignment step was required before computing sample similarity, and the alignment completes in short computational time
- Blank-attributed peaks/losses are removed when the optional filtering step is applied
- Fingerprints are validated by checking that each sample vector contains counts of MS2 peaks and neutral losses to the precursor

## Limitations

- Comparison is retention time agnostic — RT-shifted samples can be aligned, but RT-resolved structure is not preserved in the fingerprints
- Suits chemodiverse samples with poor feature overlap; it may not outperform feature-alignment-based metrics when feature overlap is high
- Environment fragility: Python version requirements interact with TMAP compatibility (>3.8 relaxed to >3.7), and filtering code changes (classes.py 'avoid erasing when filtering') indicate earlier versions could corrupt filtered matrices

## Evidence

- [abstract] Cross-platform clustering rationale: "MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms."
- [abstract] No alignment step needed: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [methods] Fingerprint construction: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [methods] Core dependencies: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling
the MS2 spectra"
- [methods] Optional blank filtering: "different filtering (remove peaks/losses from
blanks for example)"
- [readme] Ideal use case: chemodiverse/cross-platform samples: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples"
- [methods] Environment setup: "conda create --name memo python=3.8"
