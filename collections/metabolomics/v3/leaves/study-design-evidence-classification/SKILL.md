---
name: study-design-evidence-classification
description: Use when auditing or rebuilding a DNA adduct database table where each
  literature-derived entry must carry a study-design label (in vitro/in vivo/human)
  plus source/genotoxicant and detection-technique fields.
when_to_use_negative:
- The 303 'suspected' adduct rows have no reference link, so no study design can be
  classified from a publication — do not force labels on them
- The task is mass validation (recomputing monoisotopic masses from .mol/SDF structures
  with RDKit) rather than metadata/evidence-level curation
- The task is annotating unknowns in raw LC-HRMS data rather than curating the database
  table itself
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3174
tools:
- name: ChemDraw
  role: Source of the .cdxml structural identifiers accompanying each classified entry
    in the database
- name: rstudio/DT
  role: R DataTables interface used to display the database entries (including study-design
    and metadata fields) in interactive web datatables
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:cc2a08a1054070d183614314c86f92617014988f89f8ecbc9fceb0a48876ff2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - ChemDraw
  - rstudio/DT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# study-design-evidence-classification

## Summary

Assigns each literature-derived DNA adduct entry in the DNA adductomics database a study-design class (in vitro, in vivo, or human) based on the experimental conditions or biological sample reported in the source publication, as part of curating the 279-entry literature core.

## When to use

Use when auditing or rebuilding a DNA adduct database table where each literature-derived entry must carry a study-design label (in vitro/in vivo/human) plus source/genotoxicant and detection-technique fields.

## When NOT to use

- The 303 'suspected' adduct rows have no reference link, so no study design can be classified from a publication — do not force labels on them
- The task is mass validation (recomputing monoisotopic masses from .mol/SDF structures with RDKit) rather than metadata/evidence-level curation
- The task is annotating unknowns in raw LC-HRMS data rather than curating the database table itself

## Inputs

- dna_adductomics_database .xlsx file (582 entries: 279 literature-derived + 303 suspected)
- .docx database file and linked .cdxml/.mol structural files from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- Reference/publication links attached to literature-derived rows

## Outputs

- curated_core_adducts.csv with the 279 literature-derived entries including study-design labels (in vitro/in vivo/human), source/genotoxicant, and identification technique
- Per-entry QA flag list marking missing or inconsistent metadata fields

## How to apply

Parse the database .xlsx file from the dna_adductomics_database GitLab repository with pandas (openpyxl engine) and isolate the 279 literature-derived entries by selecting rows that carry a reference link, separating them from the 303 'suspected' adducts that lack references. For each such row, verify the metadata fields reported in the curated core — name, abbreviation, alternative name, source/genotoxicant, study design, and identification technique — against the deposited table, marking missing or inconsistent values. Classify the study design into in vitro, in vivo, or human according to the experimental conditions or biological sample described in the cited publication. Cross-check the classified subset against the paper's reported totals (279 literature adducts spanning 16 genotoxicant classes and 9 sources) and keep positional-isomer entries distinct. Write the verified core subset with all fields to curated_core_adducts.csv. Optionally inspect classified entries in the interactive web datatables that also hold the experimental MS/MS spectra of 15 reference standards and in silico fragments.

## Related tools

- **ChemDraw** (Source of the .cdxml structural identifiers accompanying each classified entry in the database)
- **rstudio/DT** (R DataTables interface used to display the database entries (including study-design and metadata fields) in interactive web datatables) — https://github.com/rstudio/DT

## Examples

```
import pandas as pd; df = pd.read_excel('dna_adducts.xlsx', engine='openpyxl'); core = df[df['reference'].notna()]; core['study_design'].apply(lambda s: s in {'in vitro','in vivo','human'})
```

## Evaluation signals

- The rebuilt core contains exactly 279 literature-derived rows, matching the paper's reported total
- Every row has non-null name, formula, mass, source, study-design (in vitro/in vivo/human) and technique fields
- Entries span the reported 16 genotoxicant classes and 9 sources
- Positional-isomer entries remain distinct rather than being merged
- Recomputed masses from .mol/SDF structures match deposited [M+H]+ or [M]+ values within 0.01 Da

## Limitations

- Only the 279 literature-derived entries can be study-design classified; the 303 potential adducts from genotoxicant–nucleobase combinations lack references
- Study-design assignment depends on how the source publication describes experimental conditions or biological samples; ambiguous descriptions require manual adjudication
- No specific retention-time matching or chromatographic separation details are available to further stratify in vivo evidence

## Evidence

- [abstract] Literature vs suspected distinction: "The first step included a literature search, selecting only DNA adducts that were unequivocally identified"
- [abstract] Database scope: "for building a systematic database of 279 DNA adducts"
- [readme] Interactive display of classified entries: [evidence span withheld — anchor status: external_doc]
