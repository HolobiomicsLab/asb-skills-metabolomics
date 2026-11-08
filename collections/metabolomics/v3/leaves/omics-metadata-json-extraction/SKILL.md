---
name: omics-metadata-json-extraction
description: Use when you need to tabulate and verify paired-omics link metadata from
  Zenodo-archived PoDP JSON project files, e.g., reproducing registered genome–metabolome
  and BGC–MS/MS link structures and counts.
when_to_use_negative:
- You need the underlying metabolomics or genomics datasets themselves — the PoDP
  does not store any metabolomics or genomics datasets, only metadata defining pairs
  of omics datasets in public databases such as MassIVE, MetaboLights, and NCBI GenBank
- The metadata you need was entered as free text in the 'Other' boxes, which is inherently
  not machine-readable and cannot be reliably extracted
- Your goal is MS/MS spectral matching or molecular networking on the raw spectra
  — this skill only extracts and validates registered link metadata (labels, URLs,
  MIBiG identifiers, scan numbers)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: Paired Omics Data Platform
  role: Source of the community-curated project metadata; its web service stores each
    project as a JSON file and uploads the projects to Zenodo each month
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived PoDP project JSON dataset that is downloaded
    and parsed
- name: MIBiG
  role: Provides the BGC identifiers used in BGC–MS/MS link records
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:c0976719215a6c175c52a0de65c166754f4fd2c694027b66e52dad3d7353a75b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - MIBiG
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# omics-metadata-json-extraction

## Summary

Extracts genome–metabolome link records (genome label, user-specified metadata labels, MS spectra URL) and BGC–MS/MS link records (MIBiG identifier, MS/MS URL, scan number) from PoDP project JSON documents archived on Zenodo, then validates them against the paper's reported totals (e.g., 114 BGC–MS/MS links).

## When to use

Use when you need to tabulate and verify paired-omics link metadata from Zenodo-archived PoDP JSON project files, e.g., reproducing registered genome–metabolome and BGC–MS/MS link structures and counts.

## When NOT to use

- You need the underlying metabolomics or genomics datasets themselves — the PoDP does not store any metabolomics or genomics datasets, only metadata defining pairs of omics datasets in public databases such as MassIVE, MetaboLights, and NCBI GenBank
- The metadata you need was entered as free text in the 'Other' boxes, which is inherently not machine-readable and cannot be reliably extracted
- Your goal is MS/MS spectral matching or molecular networking on the raw spectra — this skill only extracts and validates registered link metadata (labels, URLs, MIBiG identifiers, scan numbers)

## Inputs

- Zenodo-archived PoDP project JSON documents covering all 70 projects (Zenodo deposition derived from the platform's scheduled monthly upload)
- JSON schema app/public/schema.json from github.com/iomega/paired-data-form
- Expected reference totals from the article (70 projects; 114 BGC–MS/MS links; over 4,800 genome–metabolome links)

## Outputs

- podp_link_records.json containing the tabulated genome–metabolome link records (genome label, metadata labels, spectra URL)
- podp_link_records.json containing the tabulated BGC–MS/MS link records (MIBiG identifier, MS/MS URL, scan number)
- Validation results confirming record completeness and the 114 BGC–MS/MS link count

## How to apply

Download the regularly archived PoDP JSON dataset from Zenodo, which contains the JSON documents for all 70 registered projects. For each project JSON, parse the genome–metabolome link records capturing the genome label, the user-specified metadata labels, and the URL for the specific set of MS spectra; then parse the BGC–MS/MS link records capturing the MIBiG identifier, the MS/MS URL, and the scan number. Validate each genome–metabolome link for presence of genome label, spectra URL, and metadata labels, and confirm the total BGC–MS/MS link count equals the registered 114. Write the two tabulated record sets plus validation results to a single JSON output (podp_link_records.json). The JSON schema published at app/public/schema.json in the iomega/paired-data-form repo defines the expected project format, so schema validation against it is the primary correctness check.

## Related tools

- **Paired Omics Data Platform** (Source of the community-curated project metadata; its web service stores each project as a JSON file and uploads the projects to Zenodo each month) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived PoDP project JSON dataset that is downloaded and parsed)
- **MIBiG** (Provides the BGC identifiers used in BGC–MS/MS link records)

## Examples

```
python extract_podp_links.py --zenodo-doi 10.5281/zenodo.6091663 --projects-dir data/ --expected-bgc-links 114 --schema app/public/schema.json --out podp_link_records.json
```

## Evaluation signals

- Total extracted BGC–MS/MS link records across all project JSONs equals exactly 114
- Every extracted genome–metabolome link record contains a genome label, a spectra URL, and user-specified metadata labels
- Every extracted BGC–MS/MS link record contains a MIBiG identifier, an MS/MS URL, and a scan number
- Project JSONs validate against the official schema (app/public/schema.json from iomega/paired-data-form)
- Number of distinct projects parsed equals the 70 registered PoDP datasets

## Limitations

- Free text entered in 'Other' boxes is inherently not machine-readable, so some customization metadata cannot be extracted programmatically
- Capturing the full range of relevant variables in standardized machine-readable format was deliberately avoided to keep data entry tractable, so extracted records are minimal metadata only (sample preparation, extraction method, instrumentation method)
- The platform does not store metabolomics or genomics datasets; only metadata and links to public database identifiers (MassIVE accession, MetaboLights study identifier, NCBI GenBank accession, JGI Genome ID, BioSample accession) are captured

## Evidence

- [abstract] a URL for a specific set of MS spectra is linked with the genome label: "User-specified metadata labels are also used for easy recall in the linking step, in which a URL for a specific set of MS spectra is linked with the genome label"
- [abstract] BGC–MS/MS link structure via MIBiG identifier, MS/MS URL and scan number: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [abstract] Zenodo archives of PoDP JSON data: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] JSON schema defining project format: "The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project."
- [readme] Project JSON storage and Zenodo upload: [evidence span withheld — anchor status: external_doc]
