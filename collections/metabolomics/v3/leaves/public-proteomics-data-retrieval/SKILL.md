---
name: public-proteomics-data-retrieval
description: Use when a study specifies a public accession (PRIDE project, MassIVE
  reanalysis, or Zenodo DOI) and the analysis requires the underlying raw spectra,
  PSMs, or result files; e.g. clustering 25 million MS/MS spectra from PXD000561.
when_to_use_negative:
- The spectra or result files are already available locally as mzML/MGF peak files
  — no repository retrieval or conversion is needed.
- The study provides no public accession (no PRIDE project, MassIVE reanalysis, or
  Zenodo deposit) for the required data.
- You need newly generated instrument runs rather than reanalysis of a published public
  dataset.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ThermoRawFileParser
  role: Converts downloaded Thermo raw files to MGF peak files consumed by downstream
    clustering
- name: pandas
  role: Loads and filters the downloaded peptide-spectrum match identifications (e.g.
    to peptide VATVSIPR, charge 2)
- name: falcon
  role: Downstream consumer of the retrieved and converted peak files; accepts mzML,
    mzXML, or MGF input
  repo: https://github.com/bittremieux/falcon
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:6b76c7af6da028baa2016782d421083f76c899c6753aedcbba7695cbf8b83161
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Faiss
  - MaRaCluster
  - Pyteomics
  - Python
  - Python 3.8
  - Scikit-Learn
  - ThermoRawFileParser
  - falcon
  - fastcluster
  - msCRUSH
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# public-proteomics-data-retrieval

## Summary

Retrieve public proteomics inputs — vendor raw files, spectrum identifications, and precomputed clustering results — from public repositories (PRIDE, MassIVE reanalysis, Zenodo) and convert them into analysis-ready formats (e.g. MGF peak files via ThermoRawFileParser). This is the entry step that makes large-scale downstream analyses, such as falcon spectrum clustering on 25M spectra, reproducible.

## When to use

Use when a study specifies a public accession (PRIDE project, MassIVE reanalysis, or Zenodo DOI) and the analysis requires the underlying raw spectra, PSMs, or result files; e.g. clustering 25 million MS/MS spectra from PXD000561.

## When NOT to use

- The spectra or result files are already available locally as mzML/MGF peak files — no repository retrieval or conversion is needed.
- The study provides no public accession (no PRIDE project, MassIVE reanalysis, or Zenodo deposit) for the required data.
- You need newly generated instrument runs rather than reanalysis of a published public dataset.

## Inputs

- Public repository accessions (PRIDE project PXD000561, MassIVE reanalysis RMSV000000091.3, Zenodo doi:10.5281/zenodo.4721496)
- Vendor raw mass spectrometry files (2212 Thermo raw files, 25 million MS/MS spectra)
- Peptide-spectrum match (PSM) identification files filtered at 1% FDR (10,487,235 identifications)
- Reference proteome FASTA (UniProtKB/Swiss-Prot human reference proteome)

## Outputs

- MGF peak files converted from raw files (loadable by falcon, which accepts mzML, mzXML, or MGF)
- Loaded PSM table (e.g. via pandas) filterable to specific peptides such as VATVSIPR/2
- Downloaded per-tool clustering result files (cluster labels per spectrum)

## How to apply

Resolve each accession to its repository: download raw files from PRIDE by project identifier (here PRIDE project PXD000561, 2212 raw files containing 25 million MS/MS spectra from the human draft proteome dataset by Kim et al.), spectrum identifications from the MassIVE reanalysis identifier (RMSV000000091.3, PSMs filtered at 1% FDR yielding 10,487,235 identifications), and precomputed clustering results from the Zenodo deposit (doi:10.5281/zenodo.4721496). Convert downloaded Thermo raw files to MGF peak files using ThermoRawFileParser (version 1.2.3), since falcon accepts mzML, mzXML, or MGF inputs. For peptide-level follow-up, search the spectra against a reference proteome — the paper used the UniProtKB/Swiss-Prot human reference proteome downloaded 2016/05/23 — and filter PSMs at 1% false discovery rate. Verify completeness by checking that the file count and spectrum count match the published figures (2212 files, 25 million spectra) and that identifications resolve to expected entities (e.g. peptide VATVSIPR/2 from the pig trypsin contaminant protein P00761, observed 18,657 times).

## Related tools

- **ThermoRawFileParser** (Converts downloaded Thermo raw files to MGF peak files consumed by downstream clustering)
- **pandas** (Loads and filters the downloaded peptide-spectrum match identifications (e.g. to peptide VATVSIPR, charge 2))
- **falcon** (Downstream consumer of the retrieved and converted peak files; accepts mzML, mzXML, or MGF input) — https://github.com/bittremieux/falcon

## Examples

```
import pandas as pd; psms = pd.read_table('RMSV000000091.3/psm.tsv'); sub = psms[(psms['peptide']=='VATVSIPR') & (psms['charge']==2)]
```

## Evaluation signals

- The downloaded PRIDE project PXD000561 contains 2212 raw files corresponding to the published 25 million MS/MS spectra.
- PSM identifications from MassIVE reanalysis RMSV000000091.3, filtered at 1% FDR, yield 10,487,235 spectrum identifications.
- Converted MGF files load in falcon; falcon.csv contains a cluster label line for each input spectrum.
- The most frequently identified peptide VATVSIPR/2 (P00761, pig trypsin contaminant) is present and observed 18,657 times, confirming identification data integrity.
- All five tools' clustering results from Zenodo doi:10.5281/zenodo.4721496 are present and cover the same set of spectra.

## Limitations

- Repository holdings can change: the UniProtKB/Swiss-Prot reference proteome must be pinned to a download date (the paper used 2016/05/23) for reproducible searches.
- Large raw-file downloads (2212 files, 25M spectra) are time-consuming; clustering the full dataset took msCRUSH four hours and falcon five hours, so retrieval and conversion costs scale similarly.
- Identifications and clustering results are reanalysis products; their quality depends on the original authors' search parameters and 1% FDR filtering, not just the raw data.

## Evidence

- [methods] Raw files were downloaded from PRIDE (project PXD000561): [evidence span withheld — anchor status: not_found]
