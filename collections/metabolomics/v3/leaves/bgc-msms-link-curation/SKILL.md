---
name: bgc-msms-link-curation
description: Use when parsing PoDP project JSON documents (Zenodo-archived, ~70 projects)
  to enumerate or verify BGC–MS/MS links, i.e., records connecting a MIBiG identifier
  to an MS/MS URL and scan number.
when_to_use_negative:
- You need the underlying MS/MS spectra or raw metabolomics/genomics data themselves
  — the PoDP stores only metadata and URLs, not the datasets
- You only need genome–metabolome pairing (genome label + spectra URL + metadata labels)
  without any BGC/MIBiG component
- Your input is raw mass spectrometry files (e.g., MassIVE or MetaboLights submissions)
  rather than PoDP JSON project metadata
tools:
- name: Paired Omics Data Platform
  role: Source of the project JSON records being curated; its web application stores
    each paired-omics project and defines the BGC–MS/MS link structure
  repo: https://github.com/iomega/paired-data-form
- name: Zenodo
  role: Hosts the regularly archived JSON download of all community-curated PoDP projects
    that is parsed for link records
  repo: https://doi.org/10.5281/zenodo.6091663
- name: MIBiG
  role: Identifier namespace whose accession numbers (for the same or similar BGC)
    form one field of every BGC–MS/MS link record
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41589-020-00724-z
    title: A community resource for paired genomic and metabolomic data mining
  source_content_hash: sha256:7436af3b9a7181ec7952aa8fb3254d218d6fc705432b40857c34df4de4517690
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/paired-data-form
  tools:
  - MIBiG
  - Paired Omics Data Platform
  - Zenodo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bgc-msms-link-curation

## Summary

Extracts and validates BGC–MS/MS link records — each defined by a MIBiG identifier, an MS/MS URL, and a scan number — from Paired Omics Data Platform project JSONs archived on Zenodo, and verifies the corpus-level count against the published statistic of 114 links.

## When to use

Use when parsing PoDP project JSON documents (Zenodo-archived, ~70 projects) to enumerate or verify BGC–MS/MS links, i.e., records connecting a MIBiG identifier to an MS/MS URL and scan number.

## When NOT to use

- You need the underlying MS/MS spectra or raw metabolomics/genomics data themselves — the PoDP stores only metadata and URLs, not the datasets
- You only need genome–metabolome pairing (genome label + spectra URL + metadata labels) without any BGC/MIBiG component
- Your input is raw mass spectrometry files (e.g., MassIVE or MetaboLights submissions) rather than PoDP JSON project metadata

## Inputs

- Zenodo-archived PoDP project JSON documents (all 70 projects, downloaded from the PoDP Zenodo deposit)
- JSON project schema (app/public/schema.json from iomega/paired-data-form) defining the project record format

## Outputs

- podp_link_records.json containing tabulated BGC–MS/MS link records (MIBiG identifier, MS/MS URL, scan number) and genome–metabolome link records (genome label, metadata labels, spectra URL)
- Validation results confirming the BGC–MS/MS link count equals 114

## How to apply

Download the Zenodo deposit of community-curated PoDP project metadata, which contains the JSON documents for all 70 projects. For each project JSON, extract BGC–MS/MS link records capturing the MIBiG identifier, the MS/MS URL, and the scan number; in parallel extract genome–metabolome link records capturing the genome label, user-specified metadata labels, and the URL for the specific set of MS spectra. Validate that every genome–metabolome link has a genome label, a spectra URL, and metadata labels, and that every BGC–MS/MS record has all three required fields (MIBiG ID, MS/MS URL, scan number). Count all registered BGC–MS/MS links and check the total equals 114, matching the paper's Box 1 Preliminary Dataset Statistics (alongside 70 projects, 4,853 MS samples, 1,268 genomes, 1,306 metagenomes, 42 MAGs). Rationale: these links are the platform's mechanism for tying biosynthetic gene clusters to spectra, and they supply the validated BGC–metabolite link datasets used to develop the new scoring system in NPLinker. Write the two tabulated record sets plus validation results to podp_link_records.json.

## Related tools

- **Paired Omics Data Platform** (Source of the project JSON records being curated; its web application stores each paired-omics project and defines the BGC–MS/MS link structure) — https://github.com/iomega/paired-data-form
- **Zenodo** (Hosts the regularly archived JSON download of all community-curated PoDP projects that is parsed for link records) — https://doi.org/10.5281/zenodo.6091663
- **MIBiG** (Identifier namespace whose accession numbers (for the same or similar BGC) form one field of every BGC–MS/MS link record)

## Examples

```
python -c "import json,glob; projects=[json.load(open(f)) for f in glob.glob('podp_projects/*.json')]"
```

## Evaluation signals

- Total number of extracted BGC–MS/MS links equals 114, matching the published Box 1 statistic
- Every BGC–MS/MS record contains a non-empty MIBiG identifier, an MS/MS URL, and a scan number
- Every extracted genome–metabolome link has a genome label, a spectra URL, and user-specified metadata labels
- Cross-check aggregated project/sample statistics against Box 1 (70 projects, 4,853 MS samples, 1,268 genomes, 1,306 metagenomes, 42 metagenome-assembled genomes)
- Link records can be traced back to specific project JSONs in the Zenodo deposit without schema parse errors

## Limitations

- Links are made to a MIBiG identifier for 'the same or similar BGC', so a link does not guarantee an exact cluster match
- Free text entered in the 'Other' boxes is inherently not machine-readable, limiting full automated validation of metadata fields
- The PoDP does not store metabolomics or genomics datasets — only metadata and URLs — so link correctness cannot be checked against spectral data directly
- Zenodo archives are snapshots uploaded monthly, so counts can drift from the published values in later deposits

## Evidence

- [abstract] To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number: "To create a BGC–MS/MS link, a MIBiG identifier for the same or similar BGC can be linked with a MS/MS URL and scan number"
- [abstract] These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format: "These community-curated data are regularly archived to a Zenodo dataset and made available for download in JSON format"
- [readme] The web service uses a redis queue (v5.0.5) to schedule jobs to fetch more information about the public identifiers and to upload the projects to Zenodo each month.: [evidence span withheld — anchor status: external_doc]
