---
name: molecular-network-edge-filtering
description: Use when preparing a filtered .mgf input for microbeMASST batch search
  from a deposited GNPS molecular networking job, and you need spectra unique to one
  condition (e.g. SPF) excluding GF-linked spectra.
when_to_use_negative:
- Input is already a filtered .mgf of condition-unique spectra — re-filtering would
  drop additional edges and break the expected count
- You are running a single-spectrum microbeMASST web-app search rather than batch
  input preparation
- The molecular networking job lacks an edge table with cosine similarity and delta
  m/z, so GF-linked analogs cannot be identified
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzmine
  role: Produces the .mgf molecular networking input file from LC-MS/MS data; the
    filtered spectra set is generated from the GNPS molecular networking workflow
    outputs it feeds
  repo: https://github.com/mzmine/mzmine
- name: microbeMASST
  role: Downstream consumer of the filtered .mgf — batch searches the resulting SPF-unique
    spectra against microbial metabolomics data
  repo: https://masst.gnps2.org/microbemasst/
- name: Python 3.10
  role: Used to parse GNPS job metadata and edge files and write the filtered .mgf
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:cae81df8ee0c19847352a74363912e9fbe22bb6ee45b1d8aa4d45d875bc3199e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - MZmine
  - Python 3.10
  - microbeMASST
  - mzmine
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# molecular-network-edge-filtering

## Summary

Filters MS/MS spectra from a GNPS molecular networking job to isolate microbe-consensus spectra before microbeMASST searching: retain spectra present in SPF animals, then remove any spectrum that has a molecular-network edge with cosine similarity > 0.7 and delta parent mass ± 0.02 Da to a GF-only spectrum.

## When to use

Use when preparing a filtered .mgf input for microbeMASST batch search from a deposited GNPS molecular networking job, and you need spectra unique to one condition (e.g. SPF) excluding GF-linked spectra.

## When NOT to use

- Input is already a filtered .mgf of condition-unique spectra — re-filtering would drop additional edges and break the expected count
- You are running a single-spectrum microbeMASST web-app search rather than batch input preparation
- The molecular networking job lacks an edge table with cosine similarity and delta m/z, so GF-linked analogs cannot be identified

## Inputs

- Molecular networking spectra file (.mgf) from a GNPS job
- Molecular networking edge table (cosine similarity and delta m/z per edge)
- Sample metadata mapping scans to conditions (GF vs SPF)

## Outputs

- Filtered .mgf containing condition-unique consensus MS/MS spectra (10,047 spectra in the paper)
- Set of removed spectra (GF-linked) for downstream provenance

## How to apply

Download the spectra (.mgf) and molecular networking edge files from the deposited GNPS job (e.g. 893fd89b52dc4c07a292485404f97723) and parse the metadata to identify which scans are present in SPF animals versus GF-only. Using the molecular networking edge table, remove any SPF spectrum that has an edge to a GF-only spectrum with cosine similarity > 0.7 and a delta parent ion mass of ± 0.02 Da. Retain the remaining SPF-present spectra and write them to a single .mgf file. Rationale: these thresholds are exactly the molecular networking criteria used in the paper, so filtering by them removes GF-linked analogs while keeping condition-unique consensus spectra; the resulting file is directly consumable as a batch microbeMASST input generated via MZmine. In the paper's case this yields 10,047 SPF-unique spectra, which can be used as a validation count.

## Related tools

- **mzmine** (Produces the .mgf molecular networking input file from LC-MS/MS data; the filtered spectra set is generated from the GNPS molecular networking workflow outputs it feeds) — https://github.com/mzmine/mzmine
- **microbeMASST** (Downstream consumer of the filtered .mgf — batch searches the resulting SPF-unique spectra against microbial metabolomics data) — https://masst.gnps2.org/microbemasst/
- **Python 3.10** (Used to parse GNPS job metadata and edge files and write the filtered .mgf)

## Examples

```
python jobs.py --input filtered_spf.mgf --output_prefix spf_masst
```

## Evaluation signals

- Output .mgf must contain exactly 10,047 spectra, matching the paper's filtered SPF-unique set
- Every retained scan must be present in SPF animals per the parsed metadata
- Every retained scan must have no edge to a GF-only spectrum with cosine > 0.7 and Δm/z within 0.02 Da in the edge table
- Edge criterion must use the same cosine and mass tolerance as the molecular networking job (± 0.02 Da, cosine > 0.7)
- Filtered file must load as a valid .mgf batch input for microbeMASST generated via MZmine

## Limitations

- The Fast Search API used downstream can fail entries; microbe_masst jobs.py must be re-run with skip_existing=True until no new output is generated
- Python 3.10 is explicitly required for the microbeMASST batch workflow
- Filtering depends on the deposited GNPS job's edge table and metadata; the paper's criteria are cosine > 0.7 and ± 0.02 Da and would need re-derivation for other networks

## Evidence

- [results] From the GNPS molecular networking job of the GF/SPF mouse study, spectra present in SPF animals (n = 10,047) were retained, and all MS/MS spectra with an edge (cosine similarity > 0.7) and a delta parent ion mass of ± 0.02 Da with MS/MS spectra present in GF animals were removed.: [evidence span withheld — anchor status: not_found]
- [methods] microbeMASST batch search consumes an .mgf input and optional USI list; the workflow requires re-runs because the Fast Search API may fail entries: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [readme] The README specifies the input formats and runtime for the batch search that consumes this filtered set: "You can run either a single .mgf file generated via [MZmine](https://github.com/mzmine/mzmine), from the molecular networking in GNPS workflow, or a list of [USIs]"
