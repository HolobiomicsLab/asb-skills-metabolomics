---
name: metabolomics-repository-routing
description: Use when input consists of USI strings from metabolomics resources (MassBank,
  MOTIFDB, GNPS tasks/libraries, MSV/PXD/MetaboLights/Workbench accessions) that must
  be decomposed and mapped to the correct data service.
when_to_use_negative:
- The identifier is a SPLASH (SPectraL hASH) identifier, which is database-independent
  and produced by complex hashing rather than colon-delimited USI fields.
- The USI is a native proteomics USI (ProteomeXchange-style) with no metabolomics
  extension, where the standard psidev.info USI formatting applies instead of the
  metabolomics-specific patterns.
- Input is already parsed spectrum data (e.g. mgf peaks or spectral JSON), where routing
  is unnecessary and rendering/analysis endpoints should be used instead.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service whose USI extension implements the collection classification and
    routing logic; endpoints /spectrum/, /svg/, /png/, /json/, /proxi/v0.1/spectra,
    /csv/, /qrcode/, /mirror/ accept the routed USIs
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: USI
  role: HUPO-PSI Universal Spectrum Identifier specification, extended for metabolomics
    use cases, that defines the identifier structure being parsed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:a441baf16c924038fc34a0491fc2dc75f45baedfc260ff6fce0c40d810fffbe9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  repo_url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  tools:
  - Metabolomics Spectrum Resolver
  - USI
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolomics-repository-routing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses metabolomics-extended Universal Spectrum Identifiers (mzspec:<collection>:<msRun>:<indexType>:<indexNumber>) into their four colon-delimited fields and routes each identifier to its backing resource (GNPS, MassBank, MS2LDA/MOTIFDB, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange). This is the collection-classification logic implemented by the Metabolomics Spectrum Resolver's USI extension.

## When to use

Use when input consists of USI strings from metabolomics resources (MassBank, MOTIFDB, GNPS tasks/libraries, MSV/PXD/MetaboLights/Workbench accessions) that must be decomposed and mapped to the correct data service.

## When NOT to use

- The identifier is a SPLASH (SPectraL hASH) identifier, which is database-independent and produced by complex hashing rather than colon-delimited USI fields.
- The USI is a native proteomics USI (ProteomeXchange-style) with no metabolomics extension, where the standard psidev.info USI formatting applies instead of the metabolomics-specific patterns.
- Input is already parsed spectrum data (e.g. mgf peaks or spectral JSON), where routing is unnecessary and rendering/analysis endpoints should be used instead.

## Inputs

- USI strings of the form mzspec:<collection>:<msRun>:<indexType>:<indexNumber> (e.g. mzspec:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra:scan:1943, mzspec:MASSBANK::accession:SM858102, mzspec:MOTIFDB::accession:171163, mzspec:PXD000561:Adult_Frontalcortex_bRP_Elite_85_f09:scan:17555)

## Outputs

- Parsed collection, msRun, indexType, and indexNumber components for each USI
- Resolved backing resource per USI (data repository vs. reference spectral library) as rows of usi_collection_map.csv

## How to apply

Split each USI on colons into the mzspec prefix, collection, msRun, indexType, and indexNumber fields, honoring the documented special cases: MassBank ('mzspec:MASSBANK::accession:<MassBank Accession>') and MOTIFDB ('mzspec:MOTIFDB::accession:<Motif DB accession>') leave the msRun field empty, while GNPS library spectra ('mzspec:GNPS:<GNPS library name (ignored)>:accession:<GNPS Library Accession>') carry a library name that is ignored. Classify the collection by its documented formatting pattern: TASK-<GNPS Task ID>-<File name in task> marks a GNPS/MassIVE analysis task spectrum, CCMSLIB accessions mark GNPS library spectra, MassBank accessions mark MassBank library spectra, and MSV/PXD accessions mark repository data (MassIVE/GNPS, MetaboLights, Metabolomics Workbench, or ProteomeXchange). Map each collection type to its backing resource consistent with the Metabolomics Spectrum Resolver's USI extension, and emit parsed components plus resolved resource for every input USI. Note the README caveat that these identifiers are draft ('mzdraft') metabolomics USIs subject to change.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service whose USI extension implements the collection classification and routing logic; endpoints /spectrum/, /svg/, /png/, /json/, /proxi/v0.1/spectra, /csv/, /qrcode/, /mirror/ accept the routed USIs) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **USI** (HUPO-PSI Universal Spectrum Identifier specification, extended for metabolomics use cases, that defines the identifier structure being parsed)

## Examples

```
curl 'https://metabolomics-usi.ucsd.edu/json/?usi=mzspec:MASSBANK::accession:SM858102'
```

## Evaluation signals

- Every USI listed in the README's Example USI URLs section must be present in the output table and classified per the documented formatting patterns.
- Empty-msRun handling is correct: MassBank (mzspec:MASSBANK::accession:SM858102) and MOTIFDB (mzspec:MOTIFDB::accession:171163) USIs parse with a blank msRun field.
- GNPS library USIs (mzspec:GNPS:GNPS-LIBRARY:accession:CCMSLIB00005436077) are classified with the msRun field (library name) ignored, per the documented pattern.
- Task spectra vs. repository data are distinguished: TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra(/specs_ms.mgf) maps to a GNPS/MassIVE analysis task, while MSV000079514, MSV000082791, and MSV000082680 map to MassIVE, MetaboLights, and Metabolomics Workbench repositories respectively.
- Every collection resolves to a backing resource with no unresolved or misclassified entries.

## Limitations

- These identifiers are based on draft USI and draft Metabolomics USI identifiers; they are subject to change and specified as 'mzdraft' instead of 'mzspec' in the first block for the moment.
- The USI standard currently mainly supports proteomics spectral data deposited via ProteomeXchange and does not natively support metabolomics repositories, which is why the metabolomics extension is required.
- For MassIVE/GNPS data repository spectra, formatting follows the proteomics USI standard (psidev.info/usi) rather than a metabolomics-specific pattern.
- One README example uses the inconsistent prefix 'mzdata' (mzdata:MSV000078547:120228_nbut_3610_it_it_take2:scan:389), so parsers may need to tolerate prefix variants.

## Evidence

- [intro] Routing targets named in article: [evidence span withheld — anchor status: not_found]
- [intro] Gap motivating the metabolomics extension: [evidence span withheld — anchor status: not_found]
