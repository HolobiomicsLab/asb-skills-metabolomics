---
name: hilic-rplc-complementary-annotation
description: Use when cleaned features from complementary HILIC and RPLC LC-MS modes
  must be annotated against in-house metabolite databases and reduced to non-redundant
  MSI level 2 identifications.
when_to_use_negative:
- The features are raw/unprocessed — annotation requires a cleaned feature table,
  not raw LC-MS data.
- Only level 1 (standard-based, MS2-confirmed) annotations are acceptable; this workflow
  caps at MSI level 2.
- The acquisition uses neither HILIC nor RPLC separation, so neither in-house database's
  m/z/RT matching rules apply.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: metID
  role: performs metabolite annotation of cleaned features against the in-house HILIC
    and RPLC databases by accurate m/z and retention time matching
- name: massdataset
  role: provides the mass_dataset object format that holds the cleaned feature table
    being annotated
- name: tidyMass
  role: umbrella framework organizing the annotation workflow; installed to obtain
    metID and massdataset
  repo: https://github.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:c1fe4a37f35718af81d35d571436bd6f20ef2c94405ef9f49a276ef6a12fb2c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - massdataset
  - metID
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# HILIC/RPLC complementary metabolite annotation

## Summary

Annotates cleaned LC-MS features using metID against complementary in-house HILIC and RPLC databases (71 and 55 metabolites respectively), merging results and resolving redundant annotations by annotation score to reach MSI level 2 identifications.

## When to use

Use when cleaned features from complementary HILIC and RPLC LC-MS modes must be annotated against in-house metabolite databases and reduced to non-redundant MSI level 2 identifications.

## When NOT to use

- The features are raw/unprocessed — annotation requires a cleaned feature table, not raw LC-MS data.
- Only level 1 (standard-based, MS2-confirmed) annotations are acceptable; this workflow caps at MSI level 2.
- The acquisition uses neither HILIC nor RPLC separation, so neither in-house database's m/z/RT matching rules apply.

## Inputs

- cleaned feature table as a mass_dataset object (post peak-picking and data cleaning, e.g. noisy features with >20% MVs in QC or >50% MVs in study groups already removed)
- in-house HILIC metabolite database (71 metabolites, metID format)
- in-house RPLC metabolite database (55 metabolites, metID format)
- public metabolite database (optional, for extended coverage)

## Outputs

- merged, non-redundant annotation table filtered to MSI level 2 (74 metabolites in the CRC case study)
- annotations_level2.csv

## How to apply

Load the cleaned feature table as a mass_dataset object in R with massdataset, then run metID annotation against the in-house HILIC database (71 metabolites) and the in-house RPLC database (55 metabolites), matching on accurate m/z and retention time in each mode. Optionally extend coverage with a public metabolite database using the same matching rules. Merge the HILIC and RPLC annotation results and remove redundant annotations based on the annotation score, which collapses multiple mode-specific matches to the same metabolite. Filter to keep only annotations at MSI confidence level 2; in the tidyMass CRC case study this retains 74 metabolites. Write the filtered table to annotations_level2.csv. The rationale for dual-mode annotation is that HILIC and RPLC resolve complementary metabolite polarity/retention spaces, so the two in-house databases (71 + 55 metabolites) jointly maximize coverage.

## Related tools

- **metID** (performs metabolite annotation of cleaned features against the in-house HILIC and RPLC databases by accurate m/z and retention time matching)
- **massdataset** (provides the mass_dataset object format that holds the cleaned feature table being annotated)
- **tidyMass** (umbrella framework organizing the annotation workflow; installed to obtain metID and massdataset) — https://github.com/tidymass/tidymass

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- All annotations in the final table are at MSI confidence level 2 or better; no level 3+ entries remain.
- No feature carries redundant annotations — deduplication by annotation score must leave one annotation per metabolite assignment.
- Annotation counts are consistent with database sizes: the HILIC database holds 71 metabolites and the RPLC database 55 metabolites; in the CRC case study the merged, deduplicated level 2 set is 74 metabolites.
- Every retained annotation traces to a match on accurate m/z and retention time in its respective mode (HILIC or RPLC).
- The output file annotations_level2.csv is produced and parseable with massdataset.

## Limitations

- Annotations reach only MSI level 2; no standard-confirmed level 1 identifications are produced by this skill.
- In-house database coverage is small (71 HILIC + 55 RPLC metabolites), so many features will remain unannotated unless a public database extends coverage.
- If redundant annotations are not removed based on the annotation score, merged HILIC/RPLC results will double-count metabolites.

## Evidence

- [intro] In-house database sizes for the two complementary LC modes: "two in-house databases were constructed using metID, that contain 71 and 55 metabolites in HILIC and RPLC modes, respectively"
- [intro] Deduplication rule for merged annotations: "the redundant annotations were removed based on the annotation score24"
- [intro] Confidence-level filter applied to annotations: "Only the annotations with level 2 were used for subsequent analysis"
- [readme] Citation anchoring the tidyMass tooling used for this annotation workflow: [evidence span withheld — anchor status: external_doc]
