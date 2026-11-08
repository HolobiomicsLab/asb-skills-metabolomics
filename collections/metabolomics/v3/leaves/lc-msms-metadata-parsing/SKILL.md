---
name: lc-msms-metadata-parsing
description: Use when constructing a large unannotated MS/MS corpus from raw metabolomics
  repository files (MassIVE GNPS .mzML/.mzXML) and you need per-spectrum LC–MS/MS
  metadata preserved alongside peak lists for downstream ML.
when_to_use_negative:
- Your spectra are already annotated records from reference spectral libraries (MoNA,
  NIST20, GNPS library spectra) — repository-scale mining of unannotated .mzML files
  is unnecessary.
- You only need a small curated benchmark (e.g. MassSpecGym or MSV00008490 food-sample
  data with 6 million spectra) rather than a repository-scale corpus; direct download
  of the ready-made subset is cheaper.
- Your analysis needs MS1 isotopic patterns, adduct distributions, or negative-ionization/multiply-charged
  spectra — the GeMS parsing pipeline stores only tandem MS/MS spectra with a selected
  metadata subset.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HDF5
  role: 'Serialization backend: the mined spectra and their selected LC–MS/MS metadata
    are stored in the compact HDF5-based binary format designed for deep learning.'
- name: DreaMS (pluskal-lab/DreaMS)
  role: Reference implementation and release home of the five-step MassIVE GNPS mining
    pipeline and the resulting GeMS dataset.
  repo: https://github.com/pluskal-lab/DreaMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:c51ea763f562ab53f7d224cfc9e2fa62594703e6aa8e6f372043e3f53212f52d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS (pluskal-lab/DreaMS)
  - HDF5
  - MassIVE GNPS repository
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-msms-metadata-parsing

## Summary

Mining raw LC–MS/MS experiment files (.mzML/.mzXML) from the MassIVE GNPS repository and parsing each spectrum together with selected LC–MS/MS metadata — precursor m/z, charge, retention time and instrument type — to build the repository-scale GeMS corpus (~700 million MS/MS spectra) in a compact HDF5-based binary format designed for deep learning.

## When to use

Use when constructing a large unannotated MS/MS corpus from raw metabolomics repository files (MassIVE GNPS .mzML/.mzXML) and you need per-spectrum LC–MS/MS metadata preserved alongside peak lists for downstream ML.

## When NOT to use

- Your spectra are already annotated records from reference spectral libraries (MoNA, NIST20, GNPS library spectra) — repository-scale mining of unannotated .mzML files is unnecessary.
- You only need a small curated benchmark (e.g. MassSpecGym or MSV00008490 food-sample data with 6 million spectra) rather than a repository-scale corpus; direct download of the ready-made subset is cheaper.
- Your analysis needs MS1 isotopic patterns, adduct distributions, or negative-ionization/multiply-charged spectra — the GeMS parsing pipeline stores only tandem MS/MS spectra with a selected metadata subset.

## Inputs

- LC–MS/MS experiment files (.mzML/.mzXML) downloaded from the MassIVE GNPS repository
- List/set of MassIVE GNPS metabolomics datasets to mine (~250,000 LC–MS/MS experiments)
- Quality-control criteria defining GeMS-A, GeMS-B, GeMS-C subsets

## Outputs

- GeMS corpus (~700 million / ~714 million MS/MS spectra) with selected LC–MS/MS metadata (precursor m/z, charge, retention time, instrument type)
- Compact HDF5-based binary file(s) designed for deep learning, containing GeMS-A, GeMS-B and GeMS-C subsets
- LSH-cluster assignments used to limit spectral redundancy

## How to apply

Collect ~250,000 LC–MS/MS experiment files (.mzML/.mzXML) from MassIVE GNPS, then parse each file to extract tandem MS/MS spectra together with selected LC–MS/MS metadata fields: precursor m/z, charge, retention time and instrument type. Apply the DreaMS quality-control pipeline to filter extracted spectra into three subsets (GeMS-A, GeMS-B, GeMS-C) based on criteria such as instrument m/z accuracy and number of high-intensity signals, and address redundancy via locality-sensitive hashing (LSH) clustering with random sampling to cap cluster sizes (e.g. 10 or 1,000 spectra per cluster). Finally, serialize the curated spectra plus metadata into the compact HDF5-based binary format designed for deep learning, so that the pre-training corpus can be loaded directly by models such as DreaMS. Correctness is judged by reproducing the reported scale (~714 million spectra mined) and verifying the GeMS subset structure survives serialization.

## Related tools

- **HDF5** (Serialization backend: the mined spectra and their selected LC–MS/MS metadata are stored in the compact HDF5-based binary format designed for deep learning.)
- **DreaMS (pluskal-lab/DreaMS)** (Reference implementation and release home of the five-step MassIVE GNPS mining pipeline and the resulting GeMS dataset.) — https://github.com/pluskal-lab/DreaMS

## Evaluation signals

- Mined spectrum count is on the order of the paper's reported ~714 million MS/MS spectra (approximately 700 million) across the mined MassIVE GNPS collection.
- Extracted metadata table contains non-null values for the selected LC–MS/MS fields: precursor m/z, charge, retention time, and instrument type.
- Output HDF5 file loads with spectra organized into the three QC subsets GeMS-A, GeMS-B, and GeMS-C.
- LSH clustering limits redundancy: cluster sizes capped by random sampling at the chosen cap (e.g. 10 or 1,000 spectra per cluster).
- Serialized corpus is directly consumable for DreaMS-style self-supervised pre-training (masked peak prediction / retention order prediction) without re-parsing .mzML.

## Limitations

- The stored metadata is only a selected subset of LC–MS/MS fields; MS1 isotopic patterns and adduct distributions are disregarded by the DreaMS approach.
- The GeMS collection focuses on positive ionization mode and singly charged precursor ions; more diverse MS data are not incorporated.
- Only a subset of available mass spectra was used for DreaMS training — the parsed corpus was further subsampled, so full repository scale is not exhausted.

## Evidence

- [abstract] we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments: "Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments"
- [abstract] we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [abstract] we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH): "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
