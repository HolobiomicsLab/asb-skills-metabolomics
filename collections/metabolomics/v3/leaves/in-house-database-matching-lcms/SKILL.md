---
name: in-house-database-matching-lcms
description: Use when a cleaned feature table (mass_dataset object) from LC–MS untargeted
  metabolomics needs metabolite annotations, and in-house HILIC/RPLC standard databases
  with RT information are available for metID matching.
when_to_use_negative:
- The input is still raw LC–MS data or an uncleaned feature table — data cleaning
  and QC filtering (e.g. removing features with >20% MVs in QC samples or >50% MVs
  in study groups) must be done first
- MSI level 1 identifications requiring authentic-standard confirmation are the goal
  — this workflow only retains and reports annotations up to MSI level 2
- No retention-time-matched reference database exists for the chromatographic mode
  being used, since matching depends on both accurate m/z and retention time
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metID
  role: Performs annotation of the cleaned features against the in-house HILIC, RPLC,
    and public databases by matching m/z and retention time
- name: massdataset
  role: Loads and manages the cleaned feature table as the mass_dataset object that
    metID annotates
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidyMass
  role: Provides the object-oriented workflow and uniform data structure in which
    the annotation step is executed and made traceable/reproducible
  repo: https://www.tidymass.org/
- name: R
  role: Language in which the annotation workflow is implemented and run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:9b4ff69e90582ee2af82814b8d6aad86b157e092e5e0a57bb1500fa28b5a31c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - massdataset
  - metID
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# in-house-database-matching-lcms

## Summary

Annotates LC–MS features by matching accurate m/z and retention time against curated in-house HILIC (71 metabolites) and RPLC (55 metabolites) databases plus a public database using metID within tidyMass, then resolves redundancy by annotation score and retains MSI level 2 identifications.

## When to use

Use when a cleaned feature table (mass_dataset object) from LC–MS untargeted metabolomics needs metabolite annotations, and in-house HILIC/RPLC standard databases with RT information are available for metID matching.

## When NOT to use

- The input is still raw LC–MS data or an uncleaned feature table — data cleaning and QC filtering (e.g. removing features with >20% MVs in QC samples or >50% MVs in study groups) must be done first
- MSI level 1 identifications requiring authentic-standard confirmation are the goal — this workflow only retains and reports annotations up to MSI level 2
- No retention-time-matched reference database exists for the chromatographic mode being used, since matching depends on both accurate m/z and retention time

## Inputs

- Cleaned feature table as a mass_dataset object (features with accurate m/z and retention time, post QC/noisy-feature filtering)
- In-house HILIC database constructed with metID (71 metabolites)
- In-house RPLC database constructed with metID (55 metabolites)
- Public metabolite database for extended annotation coverage

## Outputs

- Merged annotation table with redundant annotations removed by annotation score
- Level 2 annotation table written to annotations_level2.csv (74 metabolites at MSI level 2 in the CRC case study)

## How to apply

Load the cleaned feature table as a mass_dataset object in R with massdataset, then run metID annotation against the in-house HILIC database (71 metabolites) matching on accurate m/z and retention time, and repeat against the in-house RPLC database (55 metabolites) under the same matching rules. Annotate against a public metabolite database as well to extend coverage beyond the in-house libraries. Merge all annotation results and remove redundant annotations based on the annotation score, which is how conflicting identifications of the same feature are resolved. Finally filter to keep only MSI confidence level 2 annotations for downstream analysis; in the CRC case study this retained 74 metabolites. The matching relies on the RT alignment between study data and the in-house standards, so both modes (HILIC and RPLC) must be annotated with their own respective database.

## Related tools

- **metID** (Performs annotation of the cleaned features against the in-house HILIC, RPLC, and public databases by matching m/z and retention time)
- **massdataset** (Loads and manages the cleaned feature table as the mass_dataset object that metID annotates) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidyMass** (Provides the object-oriented workflow and uniform data structure in which the annotation step is executed and made traceable/reproducible) — https://www.tidymass.org/
- **R** (Language in which the annotation workflow is implemented and run)

## Examples

```
remotes::install_gitlab("tidymass/tidymass")  # then run metID annotation of the mass_dataset object against the HILIC (71) and RPLC (55) in-house databases and export annotations_level2.csv
```

## Evaluation signals

- After filtering to MSI level 2, exactly 74 metabolites remain annotated in the CRC case study (71 metabolites from HILIC, 55 from RPLC in the source databases)
- No redundant annotations remain for the same feature — all conflicts were resolved using the annotation score
- Every retained annotation carries MSI confidence level 2 (not level 0/1/3)
- Matching used both accurate m/z and retention time against the mode-specific database (HILIC features matched to HILIC DB, RPLC features to RPLC DB)
- Output table annotations_level2.csv is produced and is consumable as a downstream mass_dataset annotation result

## Limitations

- In-house databases are small (71 HILIC and 55 metabolites), so coverage is limited and a public database is needed to extend annotation coverage
- Annotations are only trustworthy up to MSI level 2; only level 2 annotations are used for subsequent analysis
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, so interoperability with external annotation tools requires converting the mass_dataset class to other data structures

## Evidence

- [intro] In-house databases constructed with metID: "two in-house databases were constructed using metID, that contain 71 and 55 metabolites in HILIC and RPLC modes, respectively"
- [intro] Redundant annotation removal: "the redundant annotations were removed based on the annotation score24"
- [intro] MSI level 2 filter: "Only the annotations with level 2 were used for subsequent analysis"
- [readme] tidyMass installation: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [discussion] Interoperability caveat: "some widely used metabolomics tools are not wrapped up or supported in tidyMass, it is capable to convert the “mass_dataset” class to more eligible data structures"
