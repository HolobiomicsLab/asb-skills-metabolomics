---
name: disease-relevance-metabolite-tagging
description: Use when producing the final ROIAL-NMR Result Show tables for a disease-versus-control
  NMR study and metabolites must be highlighted as disease-relevant against a curated
  disease list (HMDB-derived).
when_to_use_negative:
- 'No disease under study: tagging is optional (''Highlight disease-relevant metabolites
  (optional)''), so skip it for purely descriptive or single-population profiling
  analyses'
- Metabolites already excluded by the >=50% match-ratio filter or below the 5 uM HMDB
  concentration limit must not be tagged or appear in the final table
- Automated ROI identification is out of scope — this skill does not determine which
  ROIs to define; ROIs must come from the user's own study design
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0091
tools:
- name: ROIAL-NMR
  role: GUI program whose Step 1 parameter setting accepts the disease-specific list
    to highlight disease-relevant metabolites in the final tables
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Loads and joins the match-ratio table, ROI trend/significance markers, and
    disease-specific metabolite list before tagging
- name: openpyxl
  role: Writes the disease-tagged categorized result tables to the dataResult Excel
    file
- name: XlsxWriter
  role: Excel writer used together with openpyxl for exporting the final tagged tables
- name: PyQt5
  role: GUI framework hosting the Result Show window where tagged results are displayed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:da9b2a21a931667e27348573a663d89475fba8232dedf86707951ac229f24ce1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - PyQt5
  - Python
  - ROIAL-NMR
  - XlsxWriter
  - openpyxl
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# disease-relevance-metabolite-tagging

## Summary

Tags ROIAL-NMR-identified metabolites as disease-relevant by matching them against a user-supplied (or bundled) disease-specific metabolite list, so that final categorized output tables flag which potential metabolites relate to the disease under study (e.g., lung cancer, Alzheimer's disease/ADRD, prostate cancer).

## When to use

Use when producing the final ROIAL-NMR Result Show tables for a disease-versus-control NMR study and metabolites must be highlighted as disease-relevant against a curated disease list (HMDB-derived).

## When NOT to use

- No disease under study: tagging is optional ('Highlight disease-relevant metabolites (optional)'), so skip it for purely descriptive or single-population profiling analyses
- Metabolites already excluded by the >=50% match-ratio filter or below the 5 uM HMDB concentration limit must not be tagged or appear in the final table
- Automated ROI identification is out of scope — this skill does not determine which ROIs to define; ROIs must come from the user's own study design

## Inputs

- Match-ratio table of candidate metabolites passing the >=50% SRRM match-ratio and concentration (>=5 uM) filters
- ROI table with trend markers ('+' increase / '-' decrease) and significance markers ('*' Level 1 / '!' Level 2) for the analyzed 0.5-4.5 ppm region
- Disease-specific metabolite list (user-prepared from HMDB, or ROIAL-NMR-shipped lung cancer / Alzheimer's disease / prostate cancer lists)

## Outputs

- Disease-tagged categorized metabolite tables (increase/decrease x not significant / significant / FDR-significant) shown in the Result Show window
- Final tagged tables written to the dataResult Excel file (XlsxWriter/openpyxl)

## How to apply

During Step 1 parameter setting in the ROIAL-NMR GUI, supply a disease-specific metabolite list — either one of the lists shipped with ROIAL-NMR (lung cancer, Alzheimer's disease, or prostate cancer) or a user-prepared list built from the HMDB or other reliable sources — as the 'Highlight disease-relevant metabolites' option. Tagging is applied after the candidate metabolites have passed the match-ratio filter (>=50% of SRRMs within the analyzed 0.5–4.5 ppm region, excluding peaks beyond the analyzed segment), the significance-ratio filter (>50% of SRRMs significant at p<0.05, with FDR-significance when at least one SRRM survives FDR correction), and the HMDB concentration filter (exclude concentration limits below 5 uM). Load the match-ratio table and ROI trend/significance markers ('+'/'-', '*'/'!') with pandas in Python >=3.9, join the disease-relevant labels to each metabolite, and render the tagged rows in the Result Show table. Finally write the categorized, disease-tagged tables to the dataResult Excel file using XlsxWriter/openpyxl; in the article's demonstration this yielded 88 identified metabolites with 66 disease-relevant for LC versus controls and 80 for LC-ADRD versus LC-non-ADRD.

## Related tools

- **ROIAL-NMR** (GUI program whose Step 1 parameter setting accepts the disease-specific list to highlight disease-relevant metabolites in the final tables) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Loads and joins the match-ratio table, ROI trend/significance markers, and disease-specific metabolite list before tagging)
- **openpyxl** (Writes the disease-tagged categorized result tables to the dataResult Excel file)
- **XlsxWriter** (Excel writer used together with openpyxl for exporting the final tagged tables)
- **PyQt5** (GUI framework hosting the Result Show window where tagged results are displayed)

## Examples

```
python main.py  # in ROIAL-NMR GUI: Step 1 -> input ROIs, trends, significance levels, select sample type, and 'Highlight disease-relevant metabolites'
```

## Evaluation signals

- dataResult Excel file exists and contains the disease-tagged categorized tables rendered in Result Show
- Tagged output reproduces the reported scale: 88 identified metabolites, with 66 differentiating LC from controls and 80 distinguishing LC-ADRD from LC-non-ADRD
- Every metabolite in the final table passes the >=50% match-ratio and >=5 uM concentration thresholds; sub-5-uM or low-match metabolites are absent
- Each tagged disease-relevant metabolite can be traced to an entry in the supplied disease-specific metabolite list (HMDB-derived or shipped list)
- Tagged metabolites fall into exactly one of the six categories (increase/decrease x not significant / significant / FDR-significant)

## Limitations

- ROIAL-NMR was not built to test the accuracy of user-defined ROIs, and user-defined ROIs can lead to variable results between users, which propagates to which metabolites get tagged
- Disease-specific metabolite lists depend on user preparation from the HMDB or other reliable sources; list quality directly determines tagging quality
- Absolute quantification is not the aim of the program, so disease relevance is based on HMDB concentration ranges (e.g., the 5 uM exclusion) rather than measured concentrations

## Evidence
