---
name: genome-metabolome-link-record-validation
description: Use when auditing archived PoDP project JSONs (Zenodo dumps of the 70
  paired datasets) to confirm that genome–metabolome and BGC–MS/MS link records are
  structurally complete and that totals match published counts (114 BGC–MS/MS links).
when_to_use_negative:
- When the goal is to search or browse PoDP projects interactively (e.g. keyword searches
  under the List tab for 'Streptomyces | Salinispora') rather than validate archived
  JSON dumps in bulk.
- When the input is raw metabolomics or genomics data — the PoDP does not store metabolomics
  or genomics datasets, only metadata defining pairs of omics datasets, so this skill
  cannot validate the underlying spectra or genome files themselves.
- When validating links to free-text 'Other' metadata boxes, which are inherently
  not machine-readable and cannot be structurally checked.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: Paired Omics Data Platform
  role: Source of the project JSON records being validated; its JSON schema defines
    the expected fields of each genome–metabolome and BGC–MS/MS link record
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived downloadable JSON dumps of PoDP projects that
    serve as the validation input
- name: MIBiG
  role: Provides the biosynthetic gene cluster identifiers that populate the BGC field
    of each BGC–MS/MS link record
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:56133c2d38aea4ff123922f3d4bf8cbe2a6d1faf6cffcfe1bd2f75f5fd97bc06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - MIBiG
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# genome-metabolome-link-record-validation

## Summary

Extracts and validates genome–metabolome and BGC–MS/MS link records from Paired Omics Data Platform (PoDP) project JSON documents archived on Zenodo, verifying that each genome–metabolome link carries a genome label, user-specified metadata labels, and an MS spectra URL, and that the BGC–MS/MS link count matches the expected 114 records.

## When to use

Use when auditing archived PoDP project JSONs (Zenodo dumps of the 70 paired datasets) to confirm that genome–metabolome and BGC–MS/MS link records are structurally complete and that totals match published counts (114 BGC–MS/MS links).

## When NOT to use

- When the goal is to search or browse PoDP projects interactively (e.g. keyword searches under the List tab for 'Streptomyces | Salinispora') rather than validate archived JSON dumps in bulk.
- When the input is raw metabolomics or genomics data — the PoDP does not store metabolomics or genomics datasets, only metadata defining pairs of omics datasets, so this skill cannot validate the underlying spectra or genome files themselves.
- When validating links to free-text 'Other' metadata boxes, which are inherently not machine-readable and cannot be structurally checked.

## Inputs

- Zenodo-archived PoDP project JSON documents for all 70 paired datasets (e.g. Zenodo deposition 10.5281/zenodo.6091663)
- PoDP JSON schema (app/public/schema.json in iomega/paired-data-form) defining valid project structure
- Expected reference counts from the published platform description (114 BGC–MS/MS links, >4,800 genome–metabolite links, 70 projects)

## Outputs

- podp_link_records.json containing tabulated genome–metabolome link records (genome label, metadata labels, spectra URL)
- Tabulated BGC–MS/MS link records (MIBiG identifier, MS/MS URL, scan number)
- Validation results confirming field completeness and the expected link counts

## How to apply

Load the Zenodo-archived PoDP project JSON documents for all 70 projects (the platform uploads projects to Zenodo each month, and the JSON schema at app/public/schema.json defines the project format). For each project JSON, extract genome–metabolome link records capturing the genome label, the user-specified metadata labels, and the URL for the specific set of MS spectra — these correspond to links where a URL for a specific set of MS spectra is linked with the genome label via user-specified metadata labels. Separately extract BGC–MS/MS link records capturing the MIBiG identifier, the MS/MS URL, and the scan number. Validate each genome–metabolome link for presence of all three required fields, and confirm the total BGC–MS/MS link count equals 114, the number registered in the PoDP. Write the two tabulated record sets plus validation results to podp_link_records.json.

## Related tools

- **Paired Omics Data Platform** (Source of the project JSON records being validated; its JSON schema defines the expected fields of each genome–metabolome and BGC–MS/MS link record) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived downloadable JSON dumps of PoDP projects that serve as the validation input)
- **MIBiG** (Provides the biosynthetic gene cluster identifiers that populate the BGC field of each BGC–MS/MS link record)

## Examples

```
python validate_podp_links.py --zenodo-doi 10.5281/zenodo.6091663 --schema app/public/schema.json --expected-bgc-links 114 --output podp_link_records.json
```

## Evaluation signals

- Every extracted genome–metabolome link record contains a non-empty genome label, a non-empty MS spectra URL, and user-specified metadata labels.
- Every extracted BGC–MS/MS link record contains a MIBiG identifier, an MS/MS URL, and a scan number.
- The total number of extracted BGC–MS/MS link records equals 114, matching the count registered in the PoDP.
- The number of projects parsed equals 70, matching the number of paired datasets reported for the platform.
- Output file podp_link_records.json validates against the project JSON structure described by app/public/schema.json.

## Limitations

- Free text entered in the 'Other' boxes is inherently not machine-readable, so links relying on such metadata cannot be fully validated programmatically.
- Capturing the full range of relevant variables in a standardized machine-readable format would lead to a very complex and tedious data entry process, so archived records may lack fields a validator would want.
- Counts are a moving target: the platform's community-curated data are regularly archived to Zenodo, so the 114 BGC–MS/MS link total reflects the published snapshot and must be re-confirmed against the specific Zenodo deposition being audited.

## Evidence

- [abstract] Link genome label and metadata labels with MS spectra URL: "User-specified metadata labels are also used for easy recall in the linking step, in which a URL for a specific set of MS spectra is linked with the genome label"
- [abstract] BGC–MS/MS link structure: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [abstract] Expected link counts: [evidence span withheld — anchor status: not_found]
- [abstract] Zenodo JSON archive: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] JSON schema: "The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project."
