---
name: large-scale-spectra-dataset-reconstruction
description: Use when building or reproducing a repository-scale collection of unannotated
  MS/MS spectra (e.g. mining MassIVE GNPS .mzML/.mzXML files into GeMS-style HDF5
  subsets) to serve as self-supervised pre-training data for spectral models.
when_to_use_negative:
- The input is already a curated reference spectral library (e.g. MoNA or NIST20)
  rather than raw repository-scale unannotated spectra — those libraries are annotation
  targets, not mining sources.
- You need spectra from conditions the mined collection does not cover, such as negative
  ionization mode or multiply charged precursor ions.
- You need MS1-level features such as isotopic patterns or adduct distributions —
  the mining targets tandem MS/MS spectra and metadata only.
tools:
- name: HDF5
  role: Serialization backend for storing the mined spectra and selected LC-MS/MS
    metadata in a compact binary format designed for deep learning
- name: pluskal-lab/DreaMS
  role: Reference implementation repository accompanying the paper describing and
    releasing the GeMS mining pipeline
  repo: https://github.com/pluskal-lab/DreaMS
- name: MassIVE GNPS repository
  role: Source repository of LC-MS/MS experiment files (.mzML/.mzXML) mined to construct
    GeMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:46196974f7756233f99ebafd3ac33f824ccdf2fedc977dae714b561f85553212
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - HDF5
  - MassIVE GNPS repository
  - pluskal-lab/DreaMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# large-scale-spectra-dataset-reconstruction

## Summary

Reconstructs the GeMS (GNPS Experimental Mass Spectra) dataset by mining ~250,000 LC-MS/MS experiments from the MassIVE GNPS repository into a compact HDF5-based binary format of roughly 714 million unannotated MS/MS spectra, organized into quality-filtered GeMS-A, GeMS-B and GeMS-C subsets for deep-learning consumption.

## When to use

Use when building or reproducing a repository-scale collection of unannotated MS/MS spectra (e.g. mining MassIVE GNPS .mzML/.mzXML files into GeMS-style HDF5 subsets) to serve as self-supervised pre-training data for spectral models.

## When NOT to use

- The input is already a curated reference spectral library (e.g. MoNA or NIST20) rather than raw repository-scale unannotated spectra — those libraries are annotation targets, not mining sources.
- You need spectra from conditions the mined collection does not cover, such as negative ionization mode or multiply charged precursor ions.
- You need MS1-level features such as isotopic patterns or adduct distributions — the mining targets tandem MS/MS spectra and metadata only.

## Inputs

- LC-MS/MS experiment files (.mzML/.mzXML) downloaded from the MassIVE GNPS repository
- Spectra-adjacent LC-MS/MS metadata fields: precursor m/z, charge, retention time, instrument type

## Outputs

- Compact HDF5-based binary dataset of MS/MS spectra with selected LC-MS/MS metadata (GeMS format)
- Three quality-filtered subsets: GeMS-A, GeMS-B, GeMS-C
- LSH-clustered, redundancy-controlled spectrum collection (~714 million MS/MS spectra at full scale)

## How to apply

Collect LC-MS/MS experiment files (.mzML/.mzXML) for the target mined collection — the paper's pipeline started from 250,000 LC-MS/MS experiments in MassIVE GNPS. Parse each experiment file to extract tandem MS/MS spectra together with selected LC-MS/MS metadata fields: precursor m/z, charge, retention time and instrument type. Run the quality-control pipeline to partition spectra into three subsets — GeMS-A, GeMS-B and GeMS-C — based on criteria such as instrument m/z accuracy and the number of high-intensity signals. Address redundancy by clustering similar spectra with locality-sensitive hashing (LSH), limiting cluster sizes by random sampling (e.g. 10 or 1,000 spectra per cluster) to avoid over-representing near-duplicate spectra. Serialize the curated spectra plus metadata into the compact HDF5-based binary format designed for deep learning, then verify the reconstruction reaches the reported scale of roughly 714 million MS/MS spectra.

## Related tools

- **HDF5** (Serialization backend for storing the mined spectra and selected LC-MS/MS metadata in a compact binary format designed for deep learning)
- **pluskal-lab/DreaMS** (Reference implementation repository accompanying the paper describing and releasing the GeMS mining pipeline) — https://github.com/pluskal-lab/DreaMS
- **MassIVE GNPS repository** (Source repository of LC-MS/MS experiment files (.mzML/.mzXML) mined to construct GeMS)

## Evaluation signals

- Reconstructed dataset size is on the order of the reported ~714 million MS/MS spectra across the mined MassIVE GNPS collection
- Output HDF5 files contain both peak lists and the selected LC-MS/MS metadata fields (precursor m/z, charge, retention time, instrument type)
- Spectra are partitioned into the three QC subsets GeMS-A, GeMS-B and GeMS-C defined by instrument m/z accuracy and high-intensity signal counts
- LSH clustering caps redundancy: no cluster exceeds the chosen random-sampling cap (e.g. 10 or 1,000 spectra per cluster)
- Subsets load directly into the downstream deep-learning training pipeline without a separate conversion step

## Limitations

- Training on the mined data used only a subset of the available mass spectra, so full reconstruction at the ~714 million scale is storage- and compute-intensive.
- The mined collection is centered on positive ionization mode and singly charged precursor ions; spectra beyond these conditions are not incorporated.
- The pipeline focuses solely on tandem mass spectra, disregarding MS1 isotopic patterns and adduct distributions.

## Evidence

- [abstract] Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments: "Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments"
- [abstract] we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [abstract] we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH): "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
- [other] The GeMS mining pipeline extracted approximately 700 million MS/MS spectra from MassIVE GNPS and stored them, with selected LC-MS/MS metadata, in a compact HDF5-based binary format designed for deep learning.: [evidence span withheld — anchor status: not_found]
