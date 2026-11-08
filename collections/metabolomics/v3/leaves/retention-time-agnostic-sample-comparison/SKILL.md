---
name: retention-time-agnostic-sample-comparison
description: Use when comparing LC-MS/MS samples with poor feature overlap, strong
  RT shifts, samples acquired on different LC methods or MS platforms (e.g. Maxis
  Q-ToF vs Q-Exactive Orbitrap), or over long time periods where feature-table metrics
  inherit acquisition-batch artifacts.
when_to_use_negative:
- When the scientific question requires retention-time-resolved feature positions
  — MEMO deliberately discards RT information to be RT agnostic.
- When samples have good feature overlap, no RT shift, and were acquired on the same
  platform in one batch — a standard feature-table metric suffices and avoids the
  fingerprinting overhead.
- When quantitative intensities matter — the MEMO matrix uses only presence/absence
  counts of peaks and losses; the MS1 feature intensity is not used.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: memo-ms
  role: Imports the feature table and .mgf spectra, builds SpectraDocuments MS2 fingerprints
    and the MemoMatrix, exports memo_matrix.csv, and provides ordination visualization
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: matchms
  role: Underlies memo-ms for importing, processing and handling the MS2 spectra
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Underlies memo-ms for converting spectra into documents
  repo: https://github.com/iomega/spec2vec
- name: scikit-bio
  role: Runs classical multidimensional scaling (PCoA) on the numpy dissimilarity
    matrices
- name: numpy
  role: Array backend for the dissimilarity matrices passed to PCoA
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:760b5878abcd606308857a2a841d494b86f85137f7a3acaecf01f5977ecf1c1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mandelbrot-project/memo_publication_examples
  tools:
  - matchms
  - memo-ms
  - numpy
  - scikit-bio
  - spec2vec
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# retention-time-agnostic-sample-comparison

## Summary

Vectorize metabolomics samples into MS2 fingerprints (counts of MS2 peaks and neutral losses to the precursor) and align them to compare samples retention-time-agnostically, mitigating batch effects such as RT shifts and MS sensitivity drift that dominate feature-table based ordinations.

## When to use

Use when comparing LC-MS/MS samples with poor feature overlap, strong RT shifts, samples acquired on different LC methods or MS platforms (e.g. Maxis Q-ToF vs Q-Exactive Orbitrap), or over long time periods where feature-table metrics inherit acquisition-batch artifacts.

## When NOT to use

- When the scientific question requires retention-time-resolved feature positions — MEMO deliberately discards RT information to be RT agnostic.
- When samples have good feature overlap, no RT shift, and were acquired on the same platform in one batch — a standard feature-table metric suffices and avoids the fingerprinting overhead.
- When quantitative intensities matter — the MEMO matrix uses only presence/absence counts of peaks and losses; the MS1 feature intensity is not used.

## Inputs

- MS/MS spectra file (e.g. GNPS FBMN-exported .mgf) from dataset MSV000083306 or similar
- MZmine-exported aligned feature quantification table (.csv) (optional, for the aligned-sample MEMO variant and the feature-table baseline)
- Sample metadata with acquisition batch/date annotations

## Outputs

- MemoMatrix sample-by-feature count matrix (exportable as memo_matrix.csv)
- Pairwise sample dissimilarity matrices (MEMO aligned, MEMO unaligned, feature-table based)
- PCoA sample coordinate tables and ordination figures (e.g. pcoa_plant_extracts.png)

## How to apply

Load the MS/MS spectra (e.g. GNPS FBMN-exported .mgf paired with an MZmine-aligned feature table, as in MSV000083306) and convert each sample's spectra into a SpectraDocuments MS2 fingerprint by counting the occurrence of MS2 peaks and neutral losses relative to the precursor, using presence/absence only and ignoring MS1 intensities. Align the per-sample fingerprints into a MemoMatrix — retention-time agnostic, no prior feature alignment required — and compute a pairwise dissimilarity matrix. For comparison, also compute the feature-table-based dissimilarity and, optionally, a MEMO variant from a pre-aligned feature table. Run classical multidimensional scaling (PCoA via scikit-bio on numpy arrays) on each dissimilarity matrix and project samples onto the first axes, annotating by acquisition batch. Judge success by comparing ordinations: batch separation (e.g. samples splitting before vs after 2017-10-26 due to RT shift and MS sensitivity change) should be strong in the feature-table PCoA and mitigated in the MEMO PCoAs.

## Related tools

- **memo-ms** (Imports the feature table and .mgf spectra, builds SpectraDocuments MS2 fingerprints and the MemoMatrix, exports memo_matrix.csv, and provides ordination visualization) — https://github.com/mandelbrot-project/memo_publication_examples
- **matchms** (Underlies memo-ms for importing, processing and handling the MS2 spectra) — https://github.com/matchms/matchms
- **spec2vec** (Underlies memo-ms for converting spectra into documents) — https://github.com/iomega/spec2vec
- **scikit-bio** (Runs classical multidimensional scaling (PCoA) on the numpy dissimilarity matrices)
- **numpy** (Array backend for the dissimilarity matrices passed to PCoA)

## Examples

```
conda create --name memo python=3.8 && conda activate memo && pip install numpy memo-ms scikit-bio
```

## Evaluation signals

- memo_matrix.csv contains count/presence entries for every sample present in the input feature table
- Feature-table PCoA reproduces the reported pre-2017-10-03 vs post-2017-10-26 batch split driven by RT shift and MS sensitivity change
- MEMO-based PCoAs (aligned and unaligned fingerprints) show reduced batch-driven clustering compared with the feature-table PCoA
- MEMO clustering performance is comparable to state-of-the-art spectra-based metrics while requiring no prior feature alignment and shorter computational time

## Limitations

- Only presence/absence of MS2 peaks and neutral losses is used; MS1 feature intensities are discarded, so quantitative information is lost
- Filtering of fingerprints can erase data if misapplied (classes.py filtering fix noted in changelog)
- Alignment with the aligned-sample variant inherits MZmine join-aligner tolerances (40 ppm m/z, 0.2 min RT) from the upstream feature table

## Evidence

- [abstract] MEMO allows clustering large amounts of chemodiverse samples based on LC-MS/MS profiles in a retention time agnostic manner: [evidence span withheld — anchor status: not_found]
- [abstract] MEMO's performance was achieved without a prior feature alignment step and in significantly shorter computational time: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [intro] MEMO suits particularly well to compare chemodiverse samples, i.e. with a poor features overlap, or to compare samples with a strong RT shift: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift,
acquired using different LC methods or even different mass"
- [methods] MEMO is mainly built on matchms and spec2vec packages for handling the MS2 spectra: "MEMO is mainly built on `matchms`_ and `spec2vec`_ packages for handling
the MS2 spectra and converting them into documents"
- [readme] README: occurrence of MS2 peaks and neutral losses counted to generate MS2 fingerprint: "The occurence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample."
