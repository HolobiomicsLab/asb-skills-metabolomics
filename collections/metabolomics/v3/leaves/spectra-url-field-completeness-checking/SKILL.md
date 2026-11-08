---
name: spectra-url-field-completeness-checking
description: Use when auditing PoDP project JSON records (from Zenodo archives or
  the paired-data-form app) to verify that link records are structurally complete
  before tabulating or downstream linking analyses.
when_to_use_negative:
- When the input is raw metabolomics or genomics data (mzML, FASTA, GenBank accessions)
  rather than PoDP metadata link records — the PoDP 'does not store any metabolomics
  or genomics datasets, but captures metadata defining pairs of omics datasets'
- When validating free-text 'Other' box metadata for machine-readability — the article
  notes such free text is 'inherently not machine-readable' and out of scope for structured
  field checks
- When the goal is biological validation of BGC–metabolite links (e.g., scoring with
  NPLinker) rather than structural completeness of link records
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Paired Omics Data Platform
  role: Source of the project JSON records whose genome–metabolome and BGC–MS/MS link
    fields are checked; defines the link-record structure (genome label, metadata
    labels, spectra URL; MIBiG ID, MS/MS URL, scan number)
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Repository of the regularly archived JSON dumps of community-curated PoDP
    projects that serve as the input documents for the completeness check
- name: MIBiG
  role: Identifier namespace for BGC–MS/MS link records; the MIBiG identifier is a
    required field validated in each BGC–MS/MS link
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:764a6f0fc68c3e126b423be549c479b7d558518803832d96a9ebdd435c4144c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - MIBiG
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Spectra URL field completeness checking

## Summary

Validation procedure that confirms every genome–metabolome link record in Paired Omics Data Platform (PoDP) project JSONs carries the required fields — genome label, user-specified metadata labels, and the URL for the specific set of MS spectra — and that BGC–MS/MS links carry a MIBiG identifier, MS/MS URL, and scan number.

## When to use

Use when auditing PoDP project JSON records (from Zenodo archives or the paired-data-form app) to verify that link records are structurally complete before tabulating or downstream linking analyses.

## When NOT to use

- When the input is raw metabolomics or genomics data (mzML, FASTA, GenBank accessions) rather than PoDP metadata link records — the PoDP 'does not store any metabolomics or genomics datasets, but captures metadata defining pairs of omics datasets'
- When validating free-text 'Other' box metadata for machine-readability — the article notes such free text is 'inherently not machine-readable' and out of scope for structured field checks
- When the goal is biological validation of BGC–metabolite links (e.g., scoring with NPLinker) rather than structural completeness of link records

## Inputs

- Zenodo-archived PoDP project JSON documents (one per project, ~70 projects)
- Expected record counts from the published platform snapshot (e.g., 114 BGC–MS/MS links)
- JSON schema defining the project format (app/public/schema.json from iomega/paired-data-form)

## Outputs

- Tabulated genome–metabolome link records (genome label, metadata labels, MS spectra URL)
- Tabulated BGC–MS/MS link records (MIBiG identifier, MS/MS URL, scan number)
- Validation results with per-record completeness flags and BGC–MS/MS link count check (e.g., podp_link_records.json)

## How to apply

Load the Zenodo-archived PoDP project JSON documents for all registered projects (the reference article reports 70 paired datasets). For each project JSON, extract genome–metabolome link records and check that each has three required fields: the genome label, the user-specified metadata labels, and the URL for the specific set of MS spectra. Separately extract BGC–MS/MS link records and validate that each captures the MIBiG identifier, the MS/MS URL, and the scan number. Use the article's registered totals as invariants: 114 BGC–MS/MS links and over 4,800 genome–metabolome links across the platform. Write the two tabulated record sets plus validation results (per-record completeness flags and count checks) to a JSON output such as podp_link_records.json. The JSON schema shipped with the paired-data-form app (app/public/schema.json) defines the expected project format and can drive schema-level validation.

## Related tools

- **Paired Omics Data Platform** (Source of the project JSON records whose genome–metabolome and BGC–MS/MS link fields are checked; defines the link-record structure (genome label, metadata labels, spectra URL; MIBiG ID, MS/MS URL, scan number)) — https://github.com/iomega/paired-data-form
- **Zenodo** (Repository of the regularly archived JSON dumps of community-curated PoDP projects that serve as the input documents for the completeness check)
- **MIBiG** (Identifier namespace for BGC–MS/MS link records; the MIBiG identifier is a required field validated in each BGC–MS/MS link)

## Examples

```
python check_podp_links.py --projects-json zenodo_archive/*.json --expect-bgc-links 114 --output podp_link_records.json
```

## Evaluation signals

- Every genome–metabolome link record contains all three required fields: genome label, user-specified metadata labels, and MS spectra URL (no null/missing values)
- Every BGC–MS/MS link record contains a MIBiG identifier, an MS/MS URL, and a scan number
- Extracted BGC–MS/MS link count equals 114, matching the platform total reported in the article
- All project JSONs validate against the paired-data-form JSON schema (app/public/schema.json)
- Validation results and both tabulated record sets are written to podp_link_records.json with per-record completeness flags

## Limitations

- Free text entered in 'Other' boxes is inherently not machine-readable, so those fields cannot be exhaustively completeness-checked
- Record totals (70 projects, 114 BGC–MS/MS links) reflect the article's snapshot; the platform is regularly updated and counts change over time, so expected counts must be pinned to the Zenodo archive version used
- Field presence does not guarantee correctness — e.g., the MS/MS URL may point to a valid-but-wrong spectra set; completeness checking is structural, not semantic

## Evidence

- [abstract] User-specified metadata labels are also used for easy recall in the linking step, in which a URL for a specific set of MS spectra is linked with the genome label: "User-specified metadata labels are also used for easy recall in the linking step, in which a URL for a specific set of MS spectra is linked with the genome label"
- [abstract] To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [abstract] These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] The JSON schema (app/public/schema.json) describes the format of an project.: "The [JSON schema (app/public/schema.json)](app/public/schema.json) describes the format of an project."
