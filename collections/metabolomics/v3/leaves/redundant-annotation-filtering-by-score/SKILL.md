---
name: redundant-annotation-filtering-by-score
description: Use when a feature table has been annotated against multiple metabolite
  databases in tidyMass/metID and duplicate or conflicting compound assignments must
  be reconciled before downstream statistical analysis.
when_to_use_negative:
- When features have not yet been annotated by metID against at least one database
  — there are no redundant annotations to reconcile
- When the analysis requires MSI level 1 (authentic standard) identifications — this
  workflow explicitly keeps only level 2 annotations
- When working outside the tidyMass/mass_dataset ecosystem and no annotation score
  is available to rank duplicate assignments
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metID
  role: Performs the database annotation (accurate m/z and RT matching) and supplies
    the annotation score used to remove redundant annotations
  repo: https://www.tidymass.org/
- name: massdataset
  role: Provides the mass_dataset object holding the cleaned feature table that annotations
    are attached to
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidyMass
  role: Overall R framework in which annotation merging, score-based de-duplication
    and MSI level filtering are executed
  repo: https://github.com/tidymass/tidymass
- name: R
  role: Implementation language for the annotation reconciliation and filtering steps
  repo: https://gitlab.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:1ebb0924801aa1f3dcc9748e777150c127dfedc7efd7e10951f6acfb8f10a0bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - massdataset
  - metID
  - tidyMass
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# redundant-annotation-filtering-by-score

## Summary

Merges metabolite annotations obtained from multiple databases (in-house HILIC/RPLC and public) within the tidyMass/metID workflow and removes redundant annotations based on the annotation score, keeping only MSI level 2 identifications (74 metabolites in the CRC case study).

## When to use

Use when a feature table has been annotated against multiple metabolite databases in tidyMass/metID and duplicate or conflicting compound assignments must be reconciled before downstream statistical analysis.

## When NOT to use

- When features have not yet been annotated by metID against at least one database — there are no redundant annotations to reconcile
- When the analysis requires MSI level 1 (authentic standard) identifications — this workflow explicitly keeps only level 2 annotations
- When working outside the tidyMass/mass_dataset ecosystem and no annotation score is available to rank duplicate assignments

## Inputs

- Cleaned feature table as a mass_dataset object (R, massdataset package)
- metID annotation result objects from the in-house HILIC database (71 metabolites)
- metID annotation result objects from the in-house RPLC database (55 metabolites)
- metID annotation results from a public metabolite database
- Annotation scores attached to each candidate metabolite assignment

## Outputs

- De-duplicated annotation table restricted to MSI confidence level 2 (74 metabolites in the CRC case study)
- annotations_level2.csv (level 2 annotation table exported from R)

## How to apply

After running metID annotation of the cleaned mass_dataset feature table against each database separately (in-house HILIC database with 71 metabolites and in-house RPLC database with 55 metabolites, matched on accurate m/z and retention time, plus a public database to extend coverage), merge the per-database annotation results into a single annotation table. Resolve redundancy by ranking candidate assignments per feature using the metID annotation score and discarding lower-scored duplicates. Then apply the confidence filter: keep only annotations at MSI level 2, since in this workflow only level 2 annotations were used for subsequent analysis. In the tidyMass CRC case study this reconciliation left 74 annotated metabolites, which can be exported as a level 2 annotation table (e.g. annotations_level2.csv).

## Related tools

- **metID** (Performs the database annotation (accurate m/z and RT matching) and supplies the annotation score used to remove redundant annotations) — https://www.tidymass.org/
- **massdataset** (Provides the mass_dataset object holding the cleaned feature table that annotations are attached to) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidyMass** (Overall R framework in which annotation merging, score-based de-duplication and MSI level filtering are executed) — https://github.com/tidymass/tidymass
- **R** (Implementation language for the annotation reconciliation and filtering steps) — https://gitlab.com/tidymass/tidymass

## Examples

```
docker pull jaspershen/tidymass-case-study  # then run the metID annotation merge/score-filter R script writing annotations_level2.csv
```

## Evaluation signals

- After score-based de-duplication each feature retains at most one compound assignment
- All annotations remaining in the output table are MSI confidence level 2 (no level 3+ assignments survive the filter)
- For the CRC case study the reconciled level 2 set contains exactly 74 metabolites, consistent with the published result
- Annotation counts trace back to the source databases: 71 metabolites from the HILIC in-house database and 55 from the RPLC in-house database before merging
- Exported annotations_level2.csv parses cleanly and its row count matches the filtered annotation set

## Limitations

- Only level 2 annotations are retained, so identifications are putative (structurally informed) rather than confirmed with authentic standards
- Redundancy resolution depends entirely on the metID annotation score; features with tied or uninformative scores may not be resolved reliably
- Coverage is bounded by the reference databases used (71 HILIC and 55 in-house metabolites plus public database); metabolites absent from these sources remain unannotated

## Evidence

- [intro] Redundant annotations were removed based on the annotation score: "the redundant annotations were removed based on the annotation score24"
- [intro] Only the annotations with level 2 were used for subsequent analysis: "Only the annotations with level 2 were used for subsequent analysis"
- [intro] In-house database sizes for HILIC and RPLC modes: "two in-house databases were constructed using metID, that contain 71 and 55 metabolites in HILIC and RPLC modes, respectively"
- [abstract] tidyMass supports annotation within a reproducible workflow: "a comprehensive computational framework that can achieve the shareable and reproducible workflow needs of data processing and analysis for LC-MS-based untargeted metabolomics"
- [intro] Docker image with data, code and environment for the case study: [evidence span withheld — anchor status: not_found]
