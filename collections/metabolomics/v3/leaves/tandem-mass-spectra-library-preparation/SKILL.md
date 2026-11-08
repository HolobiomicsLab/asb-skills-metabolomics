---
name: tandem-mass-spectra-library-preparation
description: Use when raw GNPS/MassIVE .mzML experiments must be converted into deduplicated,
  quality-filtered, paired or unpaired MS/MS spectral libraries (e.g. GeMS, MoNA,
  NIST20) for self-supervised pre-training or supervised fine-tuning of spectrum annotation
  models.
when_to_use_negative:
- The input is already a curated, paired spectral library in HDF5 or MGF format —
  re-running mining and LSH clustering would waste compute and could duplicate existing
  quality control.
- The analysis targets MS1 isotopic patterns, adduct distributions, negative ionization
  mode, or multiply charged precursor ions — the GeMS/DreaMS pipeline focuses solely
  on positive-mode, singly charged tandem mass spectra.
- The goal is to build spectral similarity measures (e.g. spectral entropy, MS2DeepScore)
  on already-clean spectra rather than to prepare raw repository-scale libraries.
tools:
- name: DreaMS
  role: Python package providing dreams.utils.data loading utilities, the compact
    HDF5-based binary format, and split construction for library preparation and downstream
    fine-tuning
  repo: github.com/pluskal-lab/DreaMS
- name: MassIVE GNPS repository
  role: Source repository of LC–MS/MS experiments mined to construct the GeMS datasets
- name: MoNA
  role: Standard paired spectral library merged with mined data for supervised fine-tuning
    and evaluation
- name: NIST20
  role: Standard paired spectral library merged with mined data for supervised fine-tuning
    and evaluation
- name: HDF5
  role: Storage backend for the compact deep-learning-ready binary spectral format
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:c49f309e52d156f856832670216293dcb8ae58a191653223703a32060ce679bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DreaMS
  - HDF5
  - MassIVE GNPS repository
  - MoNA
  - NIST20
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# tandem-mass-spectra-library-preparation

## Summary

Prepare curated tandem MS/MS training and evaluation libraries by mining LC–MS/MS experiments from the MassIVE GNPS repository, quality-filtering spectra into GeMS-A/B/C subsets, deduplicating via LSH clustering, and storing compact HDF5 files for downstream DreaMS fine-tuning.

## When to use

Use when raw GNPS/MassIVE .mzML experiments must be converted into deduplicated, quality-filtered, paired or unpaired MS/MS spectral libraries (e.g. GeMS, MoNA, NIST20) for self-supervised pre-training or supervised fine-tuning of spectrum annotation models.

## When NOT to use

- The input is already a curated, paired spectral library in HDF5 or MGF format — re-running mining and LSH clustering would waste compute and could duplicate existing quality control.
- The analysis targets MS1 isotopic patterns, adduct distributions, negative ionization mode, or multiply charged precursor ions — the GeMS/DreaMS pipeline focuses solely on positive-mode, singly charged tandem mass spectra.
- The goal is to build spectral similarity measures (e.g. spectral entropy, MS2DeepScore) on already-clean spectra rather than to prepare raw repository-scale libraries.

## Inputs

- LC–MS/MS experiments (.mzML files) mined from the MassIVE GNPS repository, e.g. 250,000 experiments / 250 million MS/MS spectra
- Standard spectral library files from MoNA and NIST20
- In-house or external MS/MS test sets such as the 17,052-spectrum fluorine test set (MassIVE accession MSV000094528)
- Pre-trained DreaMS checkpoint weights (116M-parameter transformer, Zenodo record 10997887)

## Outputs

- GeMS-A, GeMS-B, GeMS-C quality-filtered spectral subsets
- LSH-clustered, redundancy-reduced spectrum clusters with random per-cluster samples (10 or 1,000 spectra)
- HDF5 files in the DreaMS compact binary format holding spectra and selected LC–MS/MS metadata
- Murcko-histogram-disjoint train/val/test splits for supervised fine-tuning and evaluation

## How to apply

Mine the MassIVE GNPS repository by collecting 250,000 LC–MS/MS experiments, extract spectra together with selected LC–MS/MS metadata, and run the five-step mining pipeline (collection, quality filtering, clustering, storage). Apply the quality control algorithms to partition spectra into GeMS-A, GeMS-B and GeMS-C subsets based on criteria such as instrument m/z accuracy and number of high-intensity signals. Address redundancy by clustering similar spectra with locality-sensitive hashing (LSH), then limit cluster sizes by random sampling (e.g. 10 or 1,000 spectra per cluster) so frequent spectra do not dominate training. Store the resulting spectra and metadata in the DreaMS compact HDF5-based binary format for deep learning; load and split libraries with the DreaMS package utilities (dreams.utils.data), constructing Murcko-histogram-disjoint train/val/test splits so training and evaluation molecules share no Murcko scaffolds. For library retrieval fine-tuning, combine the GeMS-derived training data with standard spectral libraries MoNA and NIST20, which 'cover only a limited subset of known natural molecules' and therefore benefit from the scale of the mined GeMS data.

## Related tools

- **DreaMS** (Python package providing dreams.utils.data loading utilities, the compact HDF5-based binary format, and split construction for library preparation and downstream fine-tuning) — github.com/pluskal-lab/DreaMS
- **MassIVE GNPS repository** (Source repository of LC–MS/MS experiments mined to construct the GeMS datasets)
- **MoNA** (Standard paired spectral library merged with mined data for supervised fine-tuning and evaluation)
- **NIST20** (Standard paired spectral library merged with mined data for supervised fine-tuning and evaluation)
- **HDF5** (Storage backend for the compact deep-learning-ready binary spectral format)

## Examples

```
pip install -r requirements.txt
```

## Evaluation signals

- Spectra are stored in the DreaMS compact HDF5-based binary format and can be loaded via dreams.utils.data without errors.
- No Murcko scaffold overlap between train and val/test splits (Murcko-histogram-disjoint splitting verified).
- Each LSH cluster contributes at most the chosen cap (10 or 1,000) of randomly sampled spectra, so cluster-level redundancy is bounded.
- GeMS-A/B/C subset membership matches the QC criteria (instrument m/z accuracy, number of high-intensity signals).
- Total spectra count after mining approaches repository-scale coverage of positive-mode metabolomics spectra (e.g. 201 million MassIVE GNPS spectra for annotation).

## Limitations

- The pipeline focuses solely on tandem mass spectra, disregarding MS1 isotopic patterns and adduct distributions.
- Data mining covered positive ionization mode and singly charged precursor ions only; negative mode and multiply charged precursors are not incorporated.
- Training used only a subset of available mass spectra; the full potential of training on raw unannotated spectra remains to be unlocked.
- GeMS mining requires substantial storage and compute; the retrieved DreaMS README and article provide installation commands but not full parameter documentation.

## Evidence

- [abstract] Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments: "Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments"
- [abstract] we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [abstract] we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH): "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
- [abstract] we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] the standard training spectral libraries MoNA36 and NIST20 (ref. 37) cover only a limited subset of known natural molecules: "the standard training spectral libraries MoNA36 and NIST20 (ref. 37) cover only a limited subset of known natural molecules"
