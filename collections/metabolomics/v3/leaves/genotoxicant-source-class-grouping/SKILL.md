---
name: genotoxicant-source-class-grouping
description: Decision support for tabulating a curated DNA adduct database by genotoxicant
  and source categories to confirm class coverage or database scope.
when_to_use_negative:
- The goal is spectral annotation of unknown adduct peaks in raw LC-HRMS/MS runs rather
  than validating database metadata structure — use spectral matching or in silico
  fragmentation (e.g. CFM-ID) instead.
- The input is a raw MS/MS peak list without per-adduct genotoxicant or source annotations,
  so class assignment cannot be performed without first resolving adduct identities.
- Only the experimental MS/MS spectra of the 15 reference standards are available,
  not the full 582-entry database, and the task is fragment prediction rather than
  class tabulation.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: liquid chromatography (LC)
  role: Contextual separation technique behind the adduct identifications whose metadata
    fields are being grouped; not used directly in the grouping step itself.
- name: ChemDraw
  role: Used upstream to harmonize the structures and generate chemical identifiers
    (.cdxml, .mol, SMILES, InChI, InChIKey) so each adduct entry can be linked to
    its genotoxicant and source class.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:aa4ddd7b72a4f14ae5e6553731ca66530829d74625481639fdaa526569422b01
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDraw
  - liquid chromatography (LC)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# genotoxicant-source-class-grouping

## Summary

Groups DNA adduct database entries by causative genotoxicant class (16 classes reported) and source class (nine classes reported), enabling the metadata structure of the 582-entry adductome database to be validated against the paper's reported classification counts.

## When to use

Decision support for tabulating a curated DNA adduct database by genotoxicant and source categories to confirm class coverage or database scope.

## When NOT to use

- The goal is spectral annotation of unknown adduct peaks in raw LC-HRMS/MS runs rather than validating database metadata structure — use spectral matching or in silico fragmentation (e.g. CFM-ID) instead.
- The input is a raw MS/MS peak list without per-adduct genotoxicant or source annotations, so class assignment cannot be performed without first resolving adduct identities.
- Only the experimental MS/MS spectra of the 15 reference standards are available, not the full 582-entry database, and the task is fragment prediction rather than class tabulation.

## Inputs

- 582-entry DNA adduct database (.xlsx) from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- Accompanying metadata documentation (.docx) defining genotoxicant and source fields
- Per-entry annotations: causative genotoxicant, source, study design (in vitro / in vivo / human), and identification technique (MS, HRMS/MS, NMR, UV/fluorescence/immunoanalysis, 32P-postlabeling)

## Outputs

- Table of adduct counts per genotoxicant class (expected 16 classes)
- Table of adduct counts per source class (expected nine classes)
- Cross-tabulation of study-design distribution (in vitro 38%, in vivo 35%, human 27%) and identification-technique shares (MS 78%, HRMS/MS 16%)
- Validated monoisotopic mass range of the [M+H]+/[M]+ precursors (220–640 Da)

## How to apply

Load the 582-entry DNA adduct database (.xlsx plus its metadata .docx) from the GitLab repository and use the metadata fields that record the causative genotoxicant and the source (e.g. exogenous exposure vs endogenous product) for each adduct. Assign every entry to one genotoxicant class and one source class; the paper reports the resulting structure as 16 genotoxicant classes and nine source classes, alongside the in vitro 38% / in vivo 35% / human 27% study-design split and the 78% MS / 16% HRMS/MS identification-technique shares from Figure 2. When new adducts (e.g. the 303 combinatorially generated potential adducts) are appended, verify that the class counts and distribution remain consistent with the reported figure, and flag any entry whose class assignment is ambiguous. The rationale is that class-level grouping makes the database navigable and its descriptive statistics reproducible, so the grouping is only valid if each recomputed count matches the published value.

## Related tools

- **liquid chromatography (LC)** (Contextual separation technique behind the adduct identifications whose metadata fields are being grouped; not used directly in the grouping step itself.)
- **ChemDraw** (Used upstream to harmonize the structures and generate chemical identifiers (.cdxml, .mol, SMILES, InChI, InChIKey) so each adduct entry can be linked to its genotoxicant and source class.)

## Examples

```
import pandas as pd; db = pd.read_excel('dna_adductomics_database.xlsx', engine='openpyxl'); print(db['genotoxicant'].nunique(), db['source'].nunique())
```

## Evaluation signals

- Recomputed number of distinct genotoxicant classes equals 16.
- Recomputed number of distinct source classes equals nine.
- Study-design shares match Figure 2: in vitro 38%, in vivo 35%, human 27%.
- Identification-technique shares match Figure 2: MS 78%, HRMS/MS 16%.
- Minimum and maximum monoisotopic precursor mass span the reported 220–640 Da range, and this range is unchanged after adding the 303 suspected adducts.

## Limitations

- Class assignments depend on the quality of the literature-derived metadata; entries that were only tentatively identified (as opposed to unequivocally identified) may carry less reliable genotoxicant or source labels.
- The reported class counts (16 genotoxicant classes, nine source classes) reflect the 582-entry curated database; adding combinatorial potential adducts may require re-verifying these counts.
- The grouping step alone does not validate spectral annotations — fragment verification relies on the experimental MS/MS spectra of 15 reference standards and in silico fragments from CFM-ID.

## Evidence

- [results] Figure 2 descriptive statistics: 16 genotoxicant classes and nine source classes: "16 genotoxicant classes, and nine source classes"
- [results] Study-design split of the database: [evidence span withheld — anchor status: not_found]
- [abstract] Database source location for the .xlsx input: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [abstract] The 303 added potential adducts that should not change the class structure: "implementing the database with 303 new potential DNA adducts coming from different combinations of genotoxicants with nucleobases"
