---
name: spectral-quality-control-filtering
description: Use when processing tandem mass spectra (e.g. large mzML/mzXML collections
  or mined GNPS data) into quality-stratified subsets—by instrument m/z accuracy,
  high-intensity signal counts, and redundancy—to form corpora for self-supervised
  ML.
when_to_use_negative:
- Input spectra are already part of a curated paired reference library (e.g. MoNA
  or NIST20 with known structures) — annotation-focused training should not discard
  or re-tier them.
- The downstream task involves multiply charged precursor ions or negative ionization
  mode spectra that the GeMS mining scope (positive mode, singly charged precursors)
  does not cover.
- You need MS1-level features — the pipeline targets tandem MS/MS spectra and disregards
  MS1 isotopic patterns and adduct distributions.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: HDF5
  role: Used to serialize the quality-filtered GeMS spectra, including selected LC-MS/MS
    metadata, into a compact HDF5-based binary format designed for deep learning.
- name: MassIVE GNPS respositorysitory
  role: Source repository from which the raw LC-MS/MS experiments (.mzML/.mzXML) are
    mined before quality-control filtering.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:e6249c70b5ae63dec44bec45e9cdc592f67a82b2b755ff20d97017f68b42cede
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HDF5
  - MassIVE GNPS repository
  - MassIVE GNPS respositorysitory
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# spectral-quality-control-filtering

## Summary

A QC pipeline from the DreaMS paper that filters raw MS/MS spectra mined from MassIVE GNPS into three quality tiers—GeMS-A, GeMS-B and GeMS-C—based on criteria such as instrument m/z accuracy and number of high-intensity signals, producing curated corpora suitable for deep learning.

## When to use

Use when processing tandem mass spectra (e.g. large mzML/mzXML collections or mined GNPS data) into quality-stratified subsets—by instrument m/z accuracy, high-intensity signal counts, and redundancy—to form corpora for self-supervised ML.

## When NOT to use

- Input spectra are already part of a curated paired reference library (e.g. MoNA or NIST20 with known structures) — annotation-focused training should not discard or re-tier them.
- The downstream task involves multiply charged precursor ions or negative ionization mode spectra that the GeMS mining scope (positive mode, singly charged precursors) does not cover.
- You need MS1-level features — the pipeline targets tandem MS/MS spectra and disregards MS1 isotopic patterns and adduct distributions.

## Inputs

- LC-MS/MS experiment files (.mzML/.mzXML) downloaded from the MassIVE GNPS repository
- Extracted tandem MS/MS spectra with selected LC-MS/MS metadata (precursor m/z, charge, retention time, instrument type)

## Outputs

- Three quality-filtered spectral subsets: GeMS-A, GeMS-B and GeMS-C
- LSH clusters of similar spectra with per-cluster random subsampling (e.g. 10 or 1,000 spectra per cluster) removing redundancy
- Compact HDF5-based binary format storing curated spectra plus selected LC-MS/MS metadata for deep learning

## How to apply

First collect the LC-MS/MS experiments to be curated — in the DreaMS paper, 250,000 LC-MS/MS experiments downloaded from MassIVE GNPS as .mzML/.mzXML files. Parse each experiment to extract tandem MS/MS spectra together with selected LC-MS/MS metadata (precursor m/z, charge, retention time, instrument type). Run the quality-control pipeline that evaluates spectra-specific criteria such as instrument m/z accuracy and the number of high-intensity signals, and use the results to assign each spectrum to one of the three tiers: GeMS-A, GeMS-B or GeMS-C. Address redundancy by clustering similar spectra with locality-sensitive hashing (LSH), limiting cluster sizes by random sampling (e.g. retaining 10 or 1,000 spectra per cluster) so training corpora are not dominated by repeated highly similar spectra. Serialize the curated spectra plus metadata into a compact HDF5-based binary format designed for deep learning. Judge success by checking that tier assignment is consistent with the m/z-accuracy and high-intensity-signal criteria, and that the reconstructed corpus reaches the paper's scale (roughly 714 million / ~700 million MS/MS spectra mined overall).

## Related tools

- **HDF5** (Used to serialize the quality-filtered GeMS spectra, including selected LC-MS/MS metadata, into a compact HDF5-based binary format designed for deep learning.)
- **MassIVE GNPS respositorysitory** (Source repository from which the raw LC-MS/MS experiments (.mzML/.mzXML) are mined before quality-control filtering.)

## Examples

```
python geMS_mine.py --download massive_gnps --qc_filter --tiers A,B,C --store hdf5 --out GeMS.hdf5
```

## Evaluation signals

- Each spectrum is assigned to exactly one of GeMS-A, GeMS-B or GeMS-C, with assignments traceable to the instrument m/z accuracy and high-intensity-signal-count criteria.
- Locality-sensitive hashing clusters are capped as specified (e.g. 10 or 1,000 randomly sampled spectra per cluster), confirming redundancy removal.
- Serialized output is a valid HDF5 file containing both peak arrays and metadata fields (precursor m/z, charge, retention time, instrument type).
- Reconstructed dataset scale matches the reported mining output of roughly 714 million (~700 million) MS/MS spectra from MassIVE GNPS.

## Limitations

- The mining scope covers positive ionization mode with singly charged precursor ions; more diverse data (negative mode, multiply charged precursors) were not incorporated.
- The pipeline focuses solely on tandem mass spectra, disregarding MS1 isotopic patterns and adduct distributions.
- Only a subset of available mass spectra was used for training, so quality filtering necessarily discards spectra the full potential of which remains to be unlocked.

## Evidence

- [abstract] we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [abstract] LSH clustering and subsampling: "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
- [abstract] HDF5 storage: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments: "Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments"
- [discussion] ionization-mode scope: "including spectra beyond positive ionization mode or singly charged precursor ions"
- [discussion] MS1 features: "our method is focused solely on tandem mass spectra, disregarding other important features such as MS1 isotopic patterns or adduct distributions"
