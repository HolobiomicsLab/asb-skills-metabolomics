---
name: identification-technique-frequency-analysis
description: Use when you need descriptive statistics on how a curated compound-annotation
  database (e.g. DNA adducts) is distributed across identification techniques or confidence
  levels.
when_to_use_negative:
- The goal is to identify new adducts from raw LC-HRMS/MS data rather than to summarize
  how existing database entries were identified — use the spectral-library workflow
  (CFM-ID in silico fragments, experimental MS/MS of reference standards) instead.
- The database has been locally modified (e.g. entries added beyond the 582 literature-reported
  plus 303 potential adducts), so published Figure 2 percentages no longer serve as
  validation anchors.
- Only a single identification technique or a single entry is of interest — a frequency
  table adds no information over direct inspection.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3315
tools:
- name: liquid chromatography (LC)
  role: Technique category whose share of database entries is tallied; LC-HRMS identifications
    dominate the 'MS' class
- name: high-resolution mass spectrometry (HRMS)
  role: Identifies adducts by accurate precursor mass in the 'MS' category that the
    skill counts
- name: tandem HRMS/MS
  role: Technique category (tentative identification by fragmentation) whose 16% share
    is verified by the skill
- name: nuclear magnetic resonance (NMR)
  role: Reference-standard confirmation technique counted as its own category in the
    frequency table
- name: ChemDraw
  role: Used upstream to generate structural identifiers (.cdxml, .mol, SMILES) stored
    in the database that this skill summarizes
- name: rstudio/DT
  role: R DataTables package used to render the database (and its technique annotations)
    in the interactive web datatables where the shares are inspected
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:7150d5a6eaa4d8d5dced67adfac9b07dbec53fab390b43d09f20192aa02c5bf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - ChemDraw
  - high-resolution mass spectrometry (HRMS)
  - liquid chromatography (LC)
  - nuclear magnetic resonance (NMR)
  - rstudio/DT
  - tandem HRMS/MS
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# identification-technique-frequency-analysis

## Summary

Tabulates how many DNA adduct entries in the 582-entry adductomics database were identified by each technique (MS, HRMS/MS, NMR, 32P-postlabeling, etc.) so that the evidential strength of the annotations can be summarized and reported.

## When to use

Use when you need descriptive statistics on how a curated compound-annotation database (e.g. DNA adducts) is distributed across identification techniques or confidence levels.

## When NOT to use

- The goal is to identify new adducts from raw LC-HRMS/MS data rather than to summarize how existing database entries were identified — use the spectral-library workflow (CFM-ID in silico fragments, experimental MS/MS of reference standards) instead.
- The database has been locally modified (e.g. entries added beyond the 582 literature-reported plus 303 potential adducts), so published Figure 2 percentages no longer serve as validation anchors.
- Only a single identification technique or a single entry is of interest — a frequency table adds no information over direct inspection.

## Inputs

- 582-entry DNA adduct database (.xlsx) from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- accompanying metadata documentation (.docx)
- per-entry identification-technique annotations (MS, HRMS/MS, NMR, UV/fluorescence/immunoanalysis, 32P-postlabeling)

## Outputs

- frequency table of identification-technique shares across the 582 entries
- cross-tabulations of technique vs. study design, genotoxicant class, or source class
- comparison report against Figure 2 reference values (78% MS, 16% HRMS/MS)

## How to apply

Load the 582-entry database (.xlsx with metadata .docx) from the GitLab dna_adductomics_database repository into a pandas DataFrame via openpyxl. For each entry, assign the identification technique recorded in the metadata — the paper distinguishes MS, tandem HRMS/MS, NMR, UV/fluorescence/immunoanalysis, and 32P-postlabeling, with 'MS' denoting identification by exact mass or LC-HRMS retention behavior and 'HRMS/MS' requiring fragment spectra matched to reference standards. Cross-tabulate technique counts against other metadata axes (study design, genotoxicant class, source) if a richer breakdown is needed. Validate the resulting frequency table against the reported shares in Figure 2 of the paper: 78% of entries identified by MS, 16% by HRMS/MS, and the remainder by NMR, UV/fluorescence/immunoanalysis, or 32P-postlabeling; flag any recomputed value that deviates from these published percentages.

## Related tools

- **liquid chromatography (LC)** (Technique category whose share of database entries is tallied; LC-HRMS identifications dominate the 'MS' class)
- **high-resolution mass spectrometry (HRMS)** (Identifies adducts by accurate precursor mass in the 'MS' category that the skill counts)
- **tandem HRMS/MS** (Technique category (tentative identification by fragmentation) whose 16% share is verified by the skill)
- **nuclear magnetic resonance (NMR)** (Reference-standard confirmation technique counted as its own category in the frequency table)
- **ChemDraw** (Used upstream to generate structural identifiers (.cdxml, .mol, SMILES) stored in the database that this skill summarizes)
- **rstudio/DT** (R DataTables package used to render the database (and its technique annotations) in the interactive web datatables where the shares are inspected) — https://github.com/rstudio/DT

## Examples

```
python -c "import pandas as pd; df = pd.read_excel('dna_adduct_database.xlsx'); print(df['identification_technique'].value_counts(normalize=True) * 100)"
```

## Evaluation signals

- Recomputed 'MS' share equals the published 78% of the 582 entries (≈454 entries).
- Recomputed 'HRMS/MS' share equals the published 16% (≈93 entries).
- Technique counts sum to 582 with no unassigned rows (after resolving UV/fluorescence/immunoanalysis and 32P-postlabeling into their own buckets).
- Cross-tabulated genotoxicant classes total 16 and source classes total 9, matching Figure 2.
- Mass range computed from the same table (min–max of [M+H]+/[M]+ precursor mass) falls within the reported 220–640 Da.

## Limitations

- Frequencies reflect how entries were originally reported in the literature; 'MS' annotations rest on accurate-mass matching only and are weaker evidence than HRMS/MS or NMR standard comparisons.
- The frequency split is fixed for the 582-entry release; adding the 303 in-silico potential adducts (which have no experimental identification technique) changes the denominators and invalidates direct comparison to Figure 2.
- The paper does not document the exact technique-vocabulary parsing of the metadata .docx, so category assignment may need manual curation for borderline entries.

## Evidence

- [full_text] 78% MS / 16% HRMS/MS reference values: [evidence span withheld — anchor status: not_found]
- [abstract] technique vocabulary from the abstract: "Liquid chromatography (LC) coupled with high-resolution mass spectrometry (HRMS) is a powerful tool for DNA adductomics"
- [abstract] tentative-identification by HRMS/MS category: "and tentatively identified by tandem HRMS/MS"
- [abstract] NMR reference-standard confirmation category: "either comparison with reference standards or with nuclear magnetic resonance (NMR)"
- [abstract] repository where the database lives: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [readme] DT datatable rendering context: [evidence span withheld — anchor status: external_doc]
