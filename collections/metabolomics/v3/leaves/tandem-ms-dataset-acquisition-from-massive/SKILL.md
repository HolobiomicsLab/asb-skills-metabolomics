---
name: tandem-ms-dataset-acquisition-from-massive
description: Use when a downstream annotation/prioritization task needs the P. mutica
  MIA alkaloid extract MS/MS data referenced by MassIVE accession MSV95596, or when
  reproducing the article's 754-feature k-ranking.
when_to_use_negative:
- The tandem MS data are already available locally as MGF plus quantitative .csv —
  proceed directly to annotation and K_estimation instead of re-downloading.
- The analysis targets a different organism/extract or a synthetic benchmark dataset
  rather than the P. mutica real-case dataset under accession MSV95596.
- Only raw vendor instrument files without the MZmine FBMN export (.csv quantitative
  table) are needed for a workflow outside the MS2DECIDE input format.
edam_topics:
- http://edamontology.org/topic_3521
- http://edamontology.org/topic_3520
tools:
- name: MS2DECIDE
  role: 'Downstream consumer of the acquired dataset: its K_estimation function ingests
    the downloaded .csv quantitative table and .mgf tandem MS file to produce the
    k-ranked annotation output'
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Feature-based molecular networking platform to which the acquired MGF and
    quantitative files are uploaded for library-search annotation of the ion features
- name: Sirius
  role: Runs batch-mode Sirius 6 annotation (structure_identifications.tsv) on the
    features extracted from the acquired dataset
- name: ISDB-LOTUS
  role: Spectral library annotation (via spectral_lib_matcher/MatchMS, LOTUS data
    at zenodo.org/records/8287341) applied to the MGF spectra from the acquired dataset
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:254ca20c9c651f68a63b44c2ad885426369416d3946b6294ad026fca61f8535e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3521
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tandem-ms-dataset-acquisition-from-massive

## Summary

Retrieves the real-case Pleiocarpa mutica LC-MS/MS alkaloid extract tandem mass spectrometry dataset from the MassIVE repository (accession MSV95596) so that the MS2DECIDE k pipeline (GNPS + Sirius + ISDB-LOTUS) can be applied to its 754 ion features.

## When to use

Use when a downstream annotation/prioritization task needs the P. mutica MIA alkaloid extract MS/MS data referenced by MassIVE accession MSV95596, or when reproducing the article's 754-feature k-ranking.

## When NOT to use

- The tandem MS data are already available locally as MGF plus quantitative .csv — proceed directly to annotation and K_estimation instead of re-downloading.
- The analysis targets a different organism/extract or a synthetic benchmark dataset rather than the P. mutica real-case dataset under accession MSV95596.
- Only raw vendor instrument files without the MZmine FBMN export (.csv quantitative table) are needed for a workflow outside the MS2DECIDE input format.

## Inputs

- MassIVE repository accession MSV95596 (P. mutica real-case LC-MS/MS alkaloid extract dataset)
- Internet connectivity for repository access

## Outputs

- MGF file of tandem mass spectrometry (MS/MS) spectra for the P. mutica extract
- Quantitative feature table as .csv in MZmine (>2.53) FBMN export format
- Feature list of 754 ion features ready for GNPS FBMN, Sirius 6, and ISDB-LOTUS annotation

## How to apply

Locate the dataset via its MassIVE accession MSV95596 (the P. mutica real-case LC-MS/MS alkaloid extract tandem MS dataset) and download the raw feature data. The downstream MS2DECIDE workflow consumes two specific artifacts: an MGF file containing the tandem mass spectrometry data and a .csv quantitative table exported with the MZmine (>2.53) export module for FBMN. Confirm the download is complete and format-conformant before feeding the files into the K_estimation pipeline, since the quantitative file must end in .csv and the spectra file in .mgf. Success is anchored on the reproducibility target: the dataset must support a ranking over 754 ion features in which pleiomutine (node at m/z 631.3986) is ranked 525/754.

## Related tools

- **MS2DECIDE** (Downstream consumer of the acquired dataset: its K_estimation function ingests the downloaded .csv quantitative table and .mgf tandem MS file to produce the k-ranked annotation output) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Feature-based molecular networking platform to which the acquired MGF and quantitative files are uploaded for library-search annotation of the ion features)
- **Sirius** (Runs batch-mode Sirius 6 annotation (structure_identifications.tsv) on the features extracted from the acquired dataset)
- **ISDB-LOTUS** (Spectral library annotation (via spectral_lib_matcher/MatchMS, LOTUS data at zenodo.org/records/8287341) applied to the MGF spectra from the acquired dataset)

## Examples

```
pip install git+https://github.com/MejriY/MS2DECIDE.git
```

## Evaluation signals

- The retrieved dataset accession matches MSV95596 and contains tandem MS/MS spectra for the P. mutica alkaloid extract.
- The spectra file is in MGF format and the quantitative table is a .csv exported with the MZmine (>2.53) export module for FBMN, as required by K_estimation inputs.
- Downstream processing of the acquired files yields a ranking over exactly 754 ion features.
- Applying the reconstructed k pipeline to the acquired data reproduces the reported rank of pleiomutine (node at m/z 631.3986) as 525/754, matching the reference value from the article's Results section.
- The specifically reported nodes show their reference ranks: id 92 at rank 326, m/z 817.4137 at rank 92, and pleiokomenine A at rank 741.

## Limitations

- The skill is tied to a single real-case dataset (MSV95596, P. mutica); it does not generalize to arbitrary MassIVE accessions without re-validating the 754-feature ranking expectations.
- The downstream pipeline expects FBMN-style exports; using the MZmine export file module (>2.53) for FBMN is stated as the accepted format, so other feature-table formats may fail.
- GNPS access requires credentials (username, password, email) and internet connectivity, so acquisition/annotation cannot run fully offline.

## Evidence

- [other] Download the P. mutica real-case tandem MS dataset from the MassIVE repository accession MSV95596.: [evidence span withheld — anchor status: not_found]
- [readme] Required input formats for the acquired dataset: [evidence span withheld — anchor status: external_doc]
- [results] Dataset scope in the k-ranking validation: [evidence span withheld — anchor status: not_found]
