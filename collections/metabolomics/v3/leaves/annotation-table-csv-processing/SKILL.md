---
name: annotation-table-csv-processing
description: Use when MZmine/GNPS2 FBMN output (annotated feature tables with row
  ID, m/z, retention time) needs consolidation into a per-strain .csv summary, especially
  when KEGG pathway assignments must be attached and compared across strains.
when_to_use_negative:
- Input is raw .mzXML or unprocessed MS/MS spectra that still require MZmine preprocessing
  and RegFilter feature selection before annotation consolidation.
- Spectral library matching or in silico annotation itself is the goal — use the GNPS2
  FBMN/ChemWalker/SIRIUS annotation workflow first; this skill handles the post-annotation
  table processing only.
- A single strain or non-metabolomics dataset is under analysis, where cross-strain
  KEGG pathway comparison is not meaningful.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MZmine
  role: Produces the preprocessed full-scan and MS/MS feature tables whose exported
    .csv rows are the input to annotation consolidation
- name: ChemWalker
  role: Propagates annotations across the molecular network topology via its GNPS2
    interface, enriching the feature table rows that this skill consolidates
- name: RegFilter
  role: Generates the scheduled precursor lists and filtered feature tables feeding
    the FBMN annotation that ends up in the processed .csv
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:08b08283b2d31ca96c82c99cbda0a843058ece946f2c500e82c1163fac6961a0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - ChemWalker
  - MZmine
  - RegFilter
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# annotation-table-csv-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-annotation processing of feature tables exported from MZmine into annotated .csv files that record strain-level occurrence and metabolite counts, with downstream KEGG-based pathway reporting for the actinomycete strains BRA006, BRA010, and BRA177.

## When to use

Use when MZmine/GNPS2 FBMN output (annotated feature tables with row ID, m/z, retention time) needs consolidation into a per-strain .csv summary, especially when KEGG pathway assignments must be attached and compared across strains.

## When NOT to use

- Input is raw .mzXML or unprocessed MS/MS spectra that still require MZmine preprocessing and RegFilter feature selection before annotation consolidation.
- Spectral library matching or in silico annotation itself is the goal — use the GNPS2 FBMN/ChemWalker/SIRIUS annotation workflow first; this skill handles the post-annotation table processing only.
- A single strain or non-metabolomics dataset is under analysis, where cross-strain KEGG pathway comparison is not meaningful.

## Inputs

- MZmine-exported annotated feature table (.csv) for each actinomycete strain (BRA006, BRA010, BRA177)
- KEGG database compound and pathway entries

## Outputs

- .csv table listing the 39 KEGG-annotated compounds with their pathway assignments and strain-level occurrence
- Cross-strain pathway comparison flagging shared pathways (e.g. purine metabolism) and noting differing intermediates between strains

## How to apply

Process the MZmine-exported feature tables (row ID, row m/z, row retention time, exported from full-scan and MS/MS preprocessing) that carry annotations propagated by ChemWalker on GNPS2 and layered with SIRIUS 5.8.5 in silico annotations. For each strain (BRA006, BRA010, BRA177), subset the annotated rows that map to a KEGG compound entry (Kanehisa and Goto, 2000), retrieve the KEGG pathway for each mapped compound, and attach it to the corresponding feature row. Consolidate results across the three strains into a single .csv listing all annotated compounds with pathway assignments and strain-level occurrence; the article reports 39 such KEGG-annotated compounds (9 BRA006, 10 BRA010, 20 BRA177). Finally, compare pathway assignments across strains to identify common pathways, verifying that purine metabolism appears in all three with differing intermediates.

## Related tools

- **MZmine** (Produces the preprocessed full-scan and MS/MS feature tables whose exported .csv rows are the input to annotation consolidation)
- **ChemWalker** (Propagates annotations across the molecular network topology via its GNPS2 interface, enriching the feature table rows that this skill consolidates)
- **RegFilter** (Generates the scheduled precursor lists and filtered feature tables feeding the FBMN annotation that ends up in the processed .csv) — https://github.com/computational-chemical-biology/regression_filter

## Examples

```
conda activate regfilter; python notebooks/building_spl_and_assessing_XIC_for_all_ions.ipynb --input annotated_features.csv --npts 4
```

## Evaluation signals

- Output table contains exactly 39 annotated compound rows, each carrying a KEGG pathway assignment
- Strain-level counts match the reported distribution: 9 compounds for BRA006, 10 for BRA010, 20 for BRA177
- Purine metabolism is flagged as present in all three strains with differing intermediates
- Every feature row in the output retains MZmine row ID, m/z, and retention time as exported columns

## Limitations

- Annotated compounds without a KEGG cross-reference are excluded from the 39-compound total, so the output under-represents the full annotation set
- Strain differences in intermediate identity within shared pathways are reported qualitatively rather than quantified
- The workflow depends on prior successful ChemWalker and SIRIUS annotation steps; failure upstream propagates into the processed table

## Evidence

- [methods] Article methods — KEGG cross-referencing: "we cross-referenced the annotation data of each of the molecules contained in the three bacteria with the KEGG database (Kanehisa and Goto, 2000)"
- [results] Article results — counts per strain: [evidence span withheld — anchor status: not_found]
- [results] Article results — shared pathway: "there are common metabolic pathways between the three strains but with different intermediates, such as purine metabolic pathways"
- [readme] README — filtered table export: "a new .csv file with the features that meet the desired criteria will be exported to the notebooks directory of the package installed on your machine"
- [results] Article results — precursors per strain: "135 features for strain BRA006, 219 features for strain BRA010, and 150 features for strain BRA177 met the filtering requirements"
