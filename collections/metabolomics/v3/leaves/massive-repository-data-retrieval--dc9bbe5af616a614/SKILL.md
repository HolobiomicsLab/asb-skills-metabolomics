---
name: massive-repository-data-retrieval
description: Use when an externally published metabolomics dataset (e.g., a MassIVE
  accession like MSV00008616/MSV000093464) must be downloaded and merged with in-house
  LC-MS2 data into a shared analysis or knowledge graph.
when_to_use_negative:
- The data are already local and organized per-sample (i.e., you are past acquisition/retrieval
  and ready for MZmine feature detection or graph building).
- The target dataset lacks publicly deposited raw or feature-level files on MassIVE
  and cannot be retrieved by accession.
- You need real-time instrument acquisition rather than reanalysis of a published,
  repository-hosted dataset.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: MassIVE
  role: Public repository from which the LC-MS2 (.mzML) and .mgf files of the Korean
    medicinal plants dataset are retrieved; also used to upload MS data files in the
    ENPKG workflow
- name: ENPKG
  role: Downstream workflow whose 01_enpkg_data_organization scripts consume the retrieved
    files, arranging them in a sample-wise folder architecture
  repo: https://github.com/enpkg/enpkg_full/tree/main/01_enpkg_data_organization
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:5c5878ac6863beb41ec1fe8a24dbc56bb89dbb5b81dcefdf563674102eee1050
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ENPKG
  - GraphDB
  - MassIVE
  - RDF
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# massive-repository-data-retrieval

## Summary

Retrieve published LC-MS2 metabolomics data (.mzML) and deconvoluted feature spectra (.mgf) for a public dataset (e.g., MSV000093464, 337 Korean Pharmacopoeia plant extracts) from the MassIVE repository so it can be reprocessed and integrated into the ENPKG knowledge graph.

## When to use

Use when an externally published metabolomics dataset (e.g., a MassIVE accession like MSV00008616/MSV000093464) must be downloaded and merged with in-house LC-MS2 data into a shared analysis or knowledge graph.

## When NOT to use

- The data are already local and organized per-sample (i.e., you are past acquisition/retrieval and ready for MZmine feature detection or graph building).
- The target dataset lacks publicly deposited raw or feature-level files on MassIVE and cannot be retrieved by accession.
- You need real-time instrument acquisition rather than reanalysis of a published, repository-hosted dataset.

## Inputs

- MassIVE repository accession (e.g., MSV000093464 / MSV00008616)
- LC-MS2 raw/processed data (.mzML)
- Deconvoluted feature spectra (.mgf)
- Feature quantification tables and sample metadata (producing organism, etc.)

## Outputs

- Locally staged sample-wise folder architecture with per-sample .mzML, .mgf, feature tables, and metadata ready for ENPKG data-organization scripts

## How to apply

Identify the MassIVE record for the target study (here, MSV000093464 covering 337 methanolic extracts of Korean Pharmacopoeia plants profiled in PI mode on a Q-ToF spectrometer, also referenced as MSV00008616). Download the LC-MS2 data (.mzML) and the deconvoluted feature spectra (.mgf) from the MassIVE repository. Organize the exported feature quantification tables, MS2 spectra, and metadata (producing organism, etc.) in a sample-wise folder architecture using the ENPKG data-organization scripts (01_enpkg_data_organization), since the ENPKG workflow processes each sample individually. This retrieval step is the entry point for integrating heterogeneous external data — differing in platform, run time, and solvent — into the same knowledge graph as in-house data.

## Related tools

- **MassIVE** (Public repository from which the LC-MS2 (.mzML) and .mgf files of the Korean medicinal plants dataset are retrieved; also used to upload MS data files in the ENPKG workflow)
- **ENPKG** (Downstream workflow whose 01_enpkg_data_organization scripts consume the retrieved files, arranging them in a sample-wise folder architecture) — https://github.com/enpkg/enpkg_full/tree/main/01_enpkg_data_organization

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git && cd enpkg_full; # place retrieved MassIVE MSV000093464 .mzML/.mgf per-sample, then: sh workflow/00_workflow_all.sh
```

## Evaluation signals

- The downloaded package contains both LC-MS2 .mzML files and deconvoluted feature .mgf spectra for all 337 samples of the Korean Pharmacopoeia dataset.
- After running 01_enpkg_data_organization, each sample has its own folder containing the .mzML, .mgf, feature quantification table, and metadata (producing organism).
- The retrieved data can be processed by ENPKG and the resulting triples imported into GraphDB and queried at https://enpkg.commons-lab.org/graphdb/.
- Cross-dataset SPARQL queries over the integrated graph retrieve spectral analog matches against the in-house 1,600 plant extract dataset (e.g., scopolamine analogs in Datura metel and Scopolia japonica), demonstrating successful integration despite platform, run-time, and solvent differences.

## Limitations

- The external dataset was acquired under different experimental conditions (different platform, run time, and solvent) from the in-house data; integration relies on the sample-centric KG approach, not RT-based feature alignment, which fails across batches/instruments.
- The workflow targets LC-MS/MS DDA metabolomics data (positive and/or negative ionization modes); data not matching this structure may not be directly ingestable.
- PI-mode LC-MS2 acquisition parameters for the Korean dataset are deferred to the original reference, so full method detail must be sourced externally.

## Evidence

- [other] Retrieve the LC-MS2 data (.mzML) and deconvoluted feature spectra (.mgf) for the Korean medicinal plants data set from the MassIVE repository (MSV000093464).: [evidence span withheld — anchor status: not_found]
- [results] 337 methanolic extracts of Korean Pharmacopoeia plants profiled in PI mode on a Q-ToF spectrometer (MSV00008616): "337 methanolic extracts of Korean Pharmacopoeia
plants profiled in PI mode on a Q-ToF spectrometer (MSV00008616)"
- [methods] organized in a sample-wise folder architecture using scripts available at https://github.com/enpkg/enpkg_full/tree/main/01_enpkg_data_organization: "organized in a sample-wise folder architecture using scripts available at https://github.com/enpkg/enpkg_full/tree/main/01_enpkg_data_organization"
- [abstract] current processing workflows require feature-alignment steps based on retention time which hinders the comparison of samples originating from different batches or analyzed using different: "current processing workflows require feature-alignment steps based on retention time which hinders the comparison of samples originating from different batches or analyzed using different"
- [readme] This KG also integrates data from a metabolomics study led over 337 medicinal plants of the Korean Pharmacopeia (https://doi.org/10.1038/s41597-022-01662-2).: "This KG also integrates data from a metabolomics study led over 337 medicinal plants of the Korean Pharmacopeia (https://doi.org/10.1038/s41597-022-01662-2)."
