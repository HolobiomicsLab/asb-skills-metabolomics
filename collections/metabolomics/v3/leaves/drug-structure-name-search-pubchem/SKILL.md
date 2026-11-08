---
name: drug-structure-name-search-pubchem
description: Use when building a curated drug reference library and you need to enumerate
  drug candidates with canonical structures and cross-references before pulling MS/MS
  spectra.
when_to_use_negative:
- Input is already a curated library table with resolved drug accessions and spectra
- You need quantitative drug abundance measurements from biological samples rather
  than reference-spectrum library construction
- The task is annotating spectra in a single untargeted metabolomics dataset rather
  than building a reference resource
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: PubChem
  role: Structural and name search to enumerate drug identifiers and cross-database
    accessions
- name: DrugCentral
  role: Cross-referenced source of drug identifiers and pharmacologic metadata
- name: ChEMBL
  role: Source of drug structures and accessions for cross-referencing
- name: DrugBank
  role: Source of drug identifiers and pharmacologic information
- name: Broad Drug Repurposing Hub
  role: Supplementary source of drug structures and annotations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:aab7d0c9053f3e023288954f6be15efad9838b7731bb750ad04c87ce99d00ac7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Broad Drug Repurposing Hub
  - ChEMBL
  - DrugBank
  - DrugCentral
  - PubChem
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# drug-structure-name-search-pubchem

## Summary

Enumerate drug identifiers, structures, and cross-database accessions via structural and name search against PubChem, DrugCentral, Broad Drug Repurposing Hub, ChEMBL, and DrugBank as the first step of curating the GNPS Drug Library.

## When to use

Use when building a curated drug reference library and you need to enumerate drug candidates with canonical structures and cross-references before pulling MS/MS spectra.

## When NOT to use

- Input is already a curated library table with resolved drug accessions and spectra
- You need quantitative drug abundance measurements from biological samples rather than reference-spectrum library construction
- The task is annotating spectra in a single untargeted metabolomics dataset rather than building a reference resource

## Inputs

- Drug identifiers and compound names from public databases
- Compound structures for structural search
- MS/MS reference spectra from GNPS and MSnLib

## Outputs

- List of 4723 unique drugs with cross-database accessions and standardized structures
- Merged drug library table with 99,122 MS/MS spectra keyed by drug identity
- Pharmacologic metadata table (exposure source, pharmacologic class, therapeutic indication, mechanism of action) for 4560 drugs

## How to apply

Query PubChem, DrugCentral, the Broad Drug Repurposing Hub, ChEMBL, and DrugBank via both structural and name search to enumerate drug identifiers and structures and resolve cross-database accessions for each compound. Use these resolved identities as keys to retrieve MS/MS reference spectra from GNPS and MSnLib. Then map each drug to standardized pharmacologic metadata fields (exposure source, therapeutic area, pharmacologic class, therapeutic indication, mechanism of action). Validate the outcome by confirming the library contains 4723 unique drugs with 99,122 MS/MS spectra and metadata populated for 4560 drugs.

## Related tools

- **PubChem** (Structural and name search to enumerate drug identifiers and cross-database accessions)
- **DrugCentral** (Cross-referenced source of drug identifiers and pharmacologic metadata)
- **ChEMBL** (Source of drug structures and accessions for cross-referencing)
- **DrugBank** (Source of drug identifiers and pharmacologic information)
- **Broad Drug Repurposing Hub** (Supplementary source of drug structures and annotations)

## Evaluation signals

- Library contains exactly 4723 unique drugs
- Library contains 99,122 MS/MS spectra in total
- Pharmacologic metadata (exposure source, class, indication, mechanism of action) is populated for 4560 of 4723 drugs
- Every spectrum entry is keyed to a resolved drug identity with cross-database accessions

## Limitations

- Metadata cannot be populated for all drugs; 163 of 4723 drugs lack complete pharmacologic metadata
- Coverage is limited to publicly accessible drug MS/MS spectra; spectra absent from GNPS and MSnLib will be missed
- The resource is not intended for immediate clinical decision-making

## Evidence

- [methods] Query PubChem, DrugCentral, Broad Drug Repurposing Hub, ChEMBL, and DrugBank via structural and name search to enumerate drug identifiers, structures, and cross-database accessions.: [evidence span withheld — anchor status: not_found]
- [results] Library composition: [evidence span withheld — anchor status: not_found]
- [results] Metadata completeness: [evidence span withheld — anchor status: not_found]
- [intro] Public databases intro: "Public databases, such as DrugBank16,17, DrugCentral18, DailyMed19, and KEGG DRUG20, can assist in interpretation."
- [abstract] Clinical decision-making scope: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable."
