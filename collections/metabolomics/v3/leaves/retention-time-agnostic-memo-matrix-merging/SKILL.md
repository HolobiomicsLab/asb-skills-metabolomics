---
name: retention-time-agnostic-memo-matrix-merging
description: Use when new LC-MS/MS samples must be added to an existing metabolomics
  sample-comparison matrix despite different LC gradients (e.g. 40 min vs 8 min) or
  different MS instruments (QE-Plus vs QE-Focus).
when_to_use_negative:
- When samples were acquired on the same LC gradient and instrument and a standard
  feature-alignment-based comparison (e.g. feature-table ordination) is sufficient
  — MEMO's RT-agnostic fingerprint counting is designed for chemodiverse or platform-shifted
  data
- When only MS1 feature intensities (no MS2 fragmentation spectra) are available,
  since MEMO fingerprints are built by counting MS2 peaks and neutral losses
- When the goal is chemical structural similarity from molecular networking cosine
  scores (e.g. the CSCS metric of q2-cscs) rather than MS2-profile sample vectorization
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: memo-ms
  role: 'Core package: imports feature tables/spectra via memo_ms.import_data, builds
    MS2 fingerprints (SpectraDocuments) and aligns them into a MemoMatrix without
    prior feature alignment'
  repo: https://github.com/mandelbrot-project/memo_publication_examples
- name: numpy
  role: Dependency installed before memo-ms in the dedicated conda environment
- name: matchms
  role: Handles the MS2 spectra used to generate sample fingerprints
  repo: https://github.com/matchms/matchms
- name: spec2vec
  role: Converts MS2 spectra into documents for fingerprint generation
  repo: https://github.com/iomega/spec2vec
- name: scikit-bio
  role: Conda-forge dependency of the memo environment for downstream ordination/distance
    analyses
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:81d002f49e7e9562ac4f47bfccf90ac9ef2434fa121218068b788bca20deb2c4
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Retention-time-agnostic MEMO matrix merging

## Summary

Append new samples' MS2-based MEMO fingerprints (e.g. Waltheria indica profiles from different LC gradients and MS instruments) to an existing MemoMatrix without any prior feature alignment, then re-project to check co-clustering with reference samples.

## When to use

Use when new LC-MS/MS samples must be added to an existing metabolomics sample-comparison matrix despite different LC gradients (e.g. 40 min vs 8 min) or different MS instruments (QE-Plus vs QE-Focus).

## When NOT to use

- When samples were acquired on the same LC gradient and instrument and a standard feature-alignment-based comparison (e.g. feature-table ordination) is sufficient — MEMO's RT-agnostic fingerprint counting is designed for chemodiverse or platform-shifted data
- When only MS1 feature intensities (no MS2 fragmentation spectra) are available, since MEMO fingerprints are built by counting MS2 peaks and neutral losses
- When the goal is chemical structural similarity from molecular networking cosine scores (e.g. the CSCS metric of q2-cscs) rather than MS2-profile sample vectorization

## Inputs

- MS2 spectra / feature tables of new samples to append (e.g. MassIVE MSV000088521 Waltheria indica 2014 aerial parts and 2018 roots and aerial parts files)
- Existing plant dataset feature tables/spectra files already forming the MEMO matrix
- Blank files for optional blank-filtering of peaks/losses

## Outputs

- Merged MemoMatrix containing appended sample fingerprint rows/columns
- UMAP embedding of the merged MEMO matrix
- TMAP plot showing positions of appended samples relative to reference extracts

## How to apply

Create a dedicated conda environment with Python 3.8 and install numpy plus memo-ms. Import the new samples' feature tables/spectra (e.g. Waltheria indica files from MassIVE accession MSV000088521) with memo_ms.import_data alongside the existing plant dataset files. Convert each sample's MS2 spectra into documents with matchms/spec2vec and count the occurrence of MS2 peaks and neutral losses to the precursor to build SpectraDocuments MS2 fingerprints. Align the new fingerprints with the existing fingerprints RT-agnostically via the MemoMatrix class — critically, no prior feature alignment step is performed, which is what allows samples analyzed on different chromatographic or mass spectrometry platforms to be compared. Optionally filter fingerprints (e.g. remove peaks/losses attributed to blanks) before merging, then project the merged matrix with UMAP and/or TMAP and export the embedding showing the appended samples' positions relative to reference samples such as the trypanocidal Melochia umbellata green stems extract.

## Related tools

- **memo-ms** (Core package: imports feature tables/spectra via memo_ms.import_data, builds MS2 fingerprints (SpectraDocuments) and aligns them into a MemoMatrix without prior feature alignment) — https://github.com/mandelbrot-project/memo_publication_examples
- **numpy** (Dependency installed before memo-ms in the dedicated conda environment)
- **matchms** (Handles the MS2 spectra used to generate sample fingerprints) — https://github.com/matchms/matchms
- **spec2vec** (Converts MS2 spectra into documents for fingerprint generation) — https://github.com/iomega/spec2vec
- **scikit-bio** (Conda-forge dependency of the memo environment for downstream ordination/distance analyses)

## Examples

```
conda create --name memo python=3.8 && conda activate memo && pip install numpy && pip install memo-ms
```

## Evaluation signals

- The merged MEMO matrix contains the three appended Waltheria indica sample columns/rows in addition to the original plant dataset samples
- The UMAP plot shows all three Waltheria indica extracts (2014 aerial parts, 2018 roots, 2018 aerial parts) clustering together with the trypanocidal Melochia umbellata green stems extract
- The TMAP visualization confirms the same co-clustering as the UMAP embedding
- No feature alignment step was performed prior to matrix merging (retention-time-agnostic alignment verified)
- The run completes in the dedicated python=3.8 conda environment with memo-ms installed

## Limitations

- New samples profiled on different LC gradients (40 min vs 8 min runs) and different MS instruments (QE-Plus vs QE-Focus) can only be compared via MS2 fingerprint overlap, not by RT-aligned features
- MEMO is best suited to chemodiverse samples with poor feature overlap; for well-aligned feature tables other metrics may be preferable
- memo-ms version history shows fragility: the MemoMatrix class replaced the former MemoContainer, python requirement changed from >3.8 to >3.7 for TMAP compatibility, and filtering in classes.py was changed to avoid erasing data — pin versions carefully

## Evidence

- [abstract] RT-agnostic merging without feature alignment: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [abstract] Cross-platform comparison rationale: "MEMO thus allows the comparison of vast ensembles of samples, even when analyzed over long periods of time, and on different chromatographic or mass spectrometry platforms."
- [methods] Fingerprint generation method: "The occurrence of MS2 peaks and neutral losses (to the precursor) in each sample is counted
and used to generate an *MS2 fingerprint* of the sample"
- [methods] Environment setup: "conda create --name memo python=3.8
   conda activate memo"
- [methods] Filtering step before merging: "different filtering (remove peaks/losses from
blanks for example)"
- [intro] Suitability for platform-shifted data: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift"
- [readme] Package API and installation: [evidence span withheld — anchor status: not_found]
