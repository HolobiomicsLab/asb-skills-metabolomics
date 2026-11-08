---
name: hdf5-binary-spectra-serialization
description: Use when building a repository-scale corpus of raw MS/MS spectra (e.g.
  from mzML/mzXML experiments in MassIVE GNPS) that must be stored compactly and streamed
  efficiently for neural network training.
when_to_use_negative:
- Input is already a curated annotated spectral library (e.g. MoNA, NIST20 reference
  libraries) rather than raw unannotated repository spectra to be mined
- The analysis requires MS1 isotopic patterns or adduct distributions — the paper's
  method 'is focused solely on tandem mass spectra, disregarding other important features
  such as MS1 isotopic patterns or adduct distributions'
- You need spectra beyond positive ionization mode or singly charged precursor ions,
  which the released corpus does not yet incorporate
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HDF5
  role: Storage library backing the compact binary format in which GeMS spectra and
    selected LC-MS/MS metadata are serialized for deep learning
- name: pluskal-lab/DreaMS
  role: Official repository providing the Python environment (requirements.txt) and
    code for the GeMS mining/HDF5 serialization pipeline
  repo: https://github.com/pluskal-lab/DreaMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:a51eb2f269c070386bf7ea25aaea2714d992164c8b162cb64f19d5a024d1a3a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - HDF5
  - MassIVE GNPS repository
  - pluskal-lab/DreaMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hdf5-binary-spectra-serialization

## Summary

Serialize mined MS/MS spectra and selected LC-MS/MS metadata (precursor m/z, charge, retention time, instrument type) into the DreaMS paper's compact HDF5-based binary format designed for deep learning, as used to store the ~700-million-spectrum GeMS dataset mined from MassIVE GNPS.

## When to use

Use when building a repository-scale corpus of raw MS/MS spectra (e.g. from mzML/mzXML experiments in MassIVE GNPS) that must be stored compactly and streamed efficiently for neural network training.

## When NOT to use

- Input is already a curated annotated spectral library (e.g. MoNA, NIST20 reference libraries) rather than raw unannotated repository spectra to be mined
- The analysis requires MS1 isotopic patterns or adduct distributions — the paper's method 'is focused solely on tandem mass spectra, disregarding other important features such as MS1 isotopic patterns or adduct distributions'
- You need spectra beyond positive ionization mode or singly charged precursor ions, which the released corpus does not yet incorporate

## Inputs

- LC-MS/MS experiment files (.mzML/.mzXML) downloaded from the MassIVE GNPS repository
- Tandem MS/MS spectra parsed from those experiment files
- Selected LC-MS/MS metadata fields: precursor m/z, charge, retention time, instrument type

## Outputs

- Compact HDF5-based binary file(s) storing GeMS spectra with selected LC-MS/MS metadata, ready for deep learning consumption
- Quality-filtered spectral subsets GeMS-A, GeMS-B and GeMS-C organized within the HDF5 store

## How to apply

First parse each LC-MS/MS experiment file (.mzML/.mzXML downloaded from the MassIVE GNPS repository) to extract tandem MS/MS spectra together with selected LC-MS/MS metadata fields (precursor m/z, charge, retention time, instrument type). Pass the extracted spectra through the paper's quality-control pipeline to organize them into the GeMS subsets GeMS-A, GeMS-B and GeMS-C based on criteria such as instrument m/z accuracy and number of high-intensity signals, and address redundancy by clustering similar spectra with locality-sensitive hashing (LSH), limiting cluster sizes by random sampling (e.g. 10 or 1,000 spectra per cluster). Then write the curated spectra plus metadata into the compact HDF5-based binary format designed for deep learning, using the HDF5 library (e.g. via the DreaMS repository's Python environment). The mining pipeline itself follows five steps starting with collection of 250,000 LC-MS/MS experiments; verify the result reproduces the reported scale of roughly 714 million MS/MS spectra.

## Related tools

- **HDF5** (Storage library backing the compact binary format in which GeMS spectra and selected LC-MS/MS metadata are serialized for deep learning)
- **pluskal-lab/DreaMS** (Official repository providing the Python environment (requirements.txt) and code for the GeMS mining/HDF5 serialization pipeline) — https://github.com/pluskal-lab/DreaMS

## Examples

```
git clone https://github.com/pluskal-lab/DreaMS && cd DreaMS && pip install -r requirements.txt
```

## Evaluation signals

- The serialized HDF5 store contains the quality-filtered subsets GeMS-A, GeMS-B and GeMS-C rather than a single undifferentiated pool
- Each stored spectrum carries the selected LC-MS/MS metadata fields (precursor m/z, charge, retention time, instrument type)
- Total scale matches the reported dataset size of roughly 700 million MS/MS spectra (approximately 714 million) mined from MassIVE GNPS
- Redundancy was controlled via LSH clustering with cluster sizes capped by random sampling (e.g. 10 or 1,000 spectra per cluster)
- The binary format loads correctly in the deep-learning training pipeline for which it was designed

## Limitations

- Only a subset of available mass spectra was used for training; the full potential of training on raw unannotated spectra remains to be unlocked
- The corpus does not yet include more diverse data such as negative ionization mode or multiply charged precursors
- Redundancy handling means the HDF5 store is not an exhaustive raw dump — similar spectra are clustered via LSH and cluster sizes are randomly subsampled

## Evidence

- [abstract] HDF5 storage format for GeMS: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Mining pipeline scale: "Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments"
- [abstract] Quality-control subsets: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [abstract] LSH deduplication: "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
- [readme] DreaMS repo environment setup: "# Install requirements
pip install -r requirements.txt"
