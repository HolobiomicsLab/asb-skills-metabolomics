---
name: cross-database-drug-metadata-harmonization
description: Reference for building a curated drug spectral library where MS/MS spectra
  from public repositories must be joined to harmonized pharmacologic metadata sourced
  from PubChem, DrugCentral, ChEMBL, DrugBank, or the Broad Drug Repurposing Hub.
when_to_use_negative:
- When the goal is immediate clinical decision-making — the article explicitly states
  the GNPS Drug Library 'is not intended for immediate clinical decision-making'
- When a targeted method with purchased reference standards is required and metabolite
  reference spectra must be experimentally acquired rather than aggregated from public
  libraries
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3520
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Source of public MS/MS reference spectra that are aggregated and merged with
    cross-database pharmacologic metadata to form the GNPS Drug Library
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:9cbb6a124de346ea86950a40e70bda4e0a90deb7564c10070793a694d7a22c9b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# cross-database-drug-metadata-harmonization

## Summary

Enumerate drugs from five public pharmacology databases, retrieve their MS/MS reference spectra from GNPS/MSnLib, and attach standardized machine-readable pharmacologic metadata (exposure source, therapeutic area, pharmacologic class, indication, mechanism of action) to build the GNPS Drug Library used for empirical drug-exposure screening in untargeted metabolomics.

## When to use

Reference for building a curated drug spectral library where MS/MS spectra from public repositories must be joined to harmonized pharmacologic metadata sourced from PubChem, DrugCentral, ChEMBL, DrugBank, or the Broad Drug Repurposing Hub.

## When NOT to use

- When the goal is immediate clinical decision-making — the article explicitly states the GNPS Drug Library 'is not intended for immediate clinical decision-making'
- When a targeted method with purchased reference standards is required and metabolite reference spectra must be experimentally acquired rather than aggregated from public libraries

## Inputs

- Drug identifier/structure lists from PubChem, DrugCentral, Broad Drug Repurposing Hub, ChEMBL, and DrugBank
- Public MS/MS reference spectra from the GNPS Spectral Library and MSnLib

## Outputs

- GNPS Drug Library: 4723 unique drugs × 99,122 MS/MS spectra with standardized pharmacologic metadata (exposure source, therapeutic area, pharmacologic class, therapeutic indication, mechanism of action) populated for 4560 drugs

## How to apply

Start by querying PubChem, DrugCentral, the Broad Drug Repurposing Hub, ChEMBL, and DrugBank via structural and name search to enumerate drug identifiers, structures, and cross-database accessions (4723 unique drugs in the published build). Retrieve all publicly available MS/MS reference spectra for those drugs from the GNPS Spectral Library and MSnLib (99,122 spectra collected). For each drug, map to the standardized vocabulary fields — exposure source, therapeutic area, pharmacologic class, therapeutic indication, and mechanism of action — completing metadata for 4560 of 4723 drugs. Merge spectra and metadata into a single GNPS Drug Library table keyed by drug identity, so that downstream workflows like drugMASST can screen untargeted metabolomics spectra and return drug annotations with machine-readable pharmacologic context. Validation is structural: confirm the final library contains 4723 unique drugs, 99,122 MS/MS spectra, and metadata populated for 4560 drugs.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Source of public MS/MS reference spectra that are aggregated and merged with cross-database pharmacologic metadata to form the GNPS Drug Library)

## Examples

```
python jobs.py --input spectra.mgf --output out_prefix
```

## Evaluation signals

- Count of unique drugs in the final library table equals 4723
- Count of collected MS/MS spectra equals 99,122
- Metadata completeness check: 4560 of 4723 drugs have all five standardized pharmacologic fields populated (exposure source, therapeutic area, pharmacologic class, therapeutic indication, mechanism of action)
- Every spectrum row is keyed to a drug identity and carries the merged metadata columns; no orphan spectra without a drug record

## Limitations

- Metadata is populated for 4560 of 4723 drugs (163 drugs lack full pharmacologic annotation)
- The library aggregates existing public MS/MS spectra, so drugs absent from GNPS/MSnLib will be missing
- Existing MS/MS reference libraries capture only the parent drug for most entries because drug-metabolite reference standards are difficult to source; metabolite spectra are largely unaccounted for in public libraries
- Not intended for immediate clinical decision-making; positioned as a data-science screening resource

## Evidence

- [full_text] compiles all publicly accessible drug MS/MS spectra, covering 4723 unique drugs represented by 99,122 MS/MS spectra collected from the GNPS Spectral Library and MSnLib: [evidence span withheld — anchor status: not_found]
- [full_text] metadata enrichment against PubChem, DrugCentral, the Broad Drug Repurposing Hub, ChEMBL, and DrugBank: [evidence span withheld — anchor status: not_found]
- [abstract] standardized vocabularies on their exposure sources, pharmacologic classes, therapeutic indications, and mechanisms: "This resource integrates MS/MS references of drugs and their metabolites/analogs with standardized vocabularies on their exposure sources, pharmacologic classes, therapeutic indications, and"
- [intro] existing MS/MS reference libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards: "existing MS/MS reference libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards25, leaving a gap in the current drug"
- [discussion] not intended for immediate clinical decision-making: "While the GNPS Drug Library is not intended for immediate clinical decision-making, this tool lays the groundwork for future integration of exposure data into precision medicine frameworks"
