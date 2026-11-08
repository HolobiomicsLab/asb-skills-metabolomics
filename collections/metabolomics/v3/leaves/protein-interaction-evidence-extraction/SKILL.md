---
name: protein-interaction-evidence-extraction
description: Use when the research question asks whether two specific proteins physically
  interact and the source text describes named assays (e.g. yeast two-hybrid, co-IP)
  with figure/supplement locators that must be tabulated per assay.
when_to_use_negative:
- The question concerns genetic cooperation or mutant phenotypes (e.g. rmh-2(jf94);
  top-3-ZnF::ha double-mutant phenotypes) rather than physical interaction assays
- The source text describes a negative interaction result without assay locators,
  e.g. RMH-1 pull-down experiments that did not detect interaction with RMH-2
- A structured interaction table already exists and only needs reformatting rather
  than extraction from prose
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Python
  role: 'Runs the extraction workflow: parsing the Results text, locating the Figure
    7C and Supplementary Figure S5F passages, and assembling the per-assay rows'
- name: pandas
  role: Builds the DataFrame with columns assay, construct_pair, outcome, evidence_location
    and writes interaction_evidence_table.csv
- name: Microsoft Excel
  role: Opens/inspects the generated interaction_evidence_table.csv for manual validation
    of row count and evidence_location fields
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:b1d2be88bd3895ce52a49f859635861ead074b6dd294af99ed03434a55428e8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Microsoft Excel
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# protein-interaction-evidence-extraction

## Summary

Extracts physical protein-interaction evidence (assay type, construct pair, outcome, and exact figure/supplement locator) from article Results text into a structured CSV. For the Dello Stritto et al. 2022 NAR TOP-3/RMH-2 study, this yields a two-row table covering the yeast two-hybrid assay (Figure 7C) and reciprocal co-immunoprecipitation (Supplementary Figure S5F).

## When to use

Use when the research question asks whether two specific proteins physically interact and the source text describes named assays (e.g. yeast two-hybrid, co-IP) with figure/supplement locators that must be tabulated per assay.

## When NOT to use

- The question concerns genetic cooperation or mutant phenotypes (e.g. rmh-2(jf94); top-3-ZnF::ha double-mutant phenotypes) rather than physical interaction assays
- The source text describes a negative interaction result without assay locators, e.g. RMH-1 pull-down experiments that did not detect interaction with RMH-2
- A structured interaction table already exists and only needs reformatting rather than extraction from prose

## Inputs

- Results section text of the article (e.g. the 'Top-3-Znf And Rmh-2 Physically Interact And Form Protein Complexes' subsection)
- Research question specifying the protein pair (here TOP-3 and RMH-2)
- Passages referencing the rmh-2::mCherry / top-3::HA co-IP strain and Supplementary Figure S5F

## Outputs

- interaction_evidence_table.csv: a two-row pandas DataFrame with columns assay, construct_pair, outcome, evidence_location

## How to apply

Load the Results section (e.g. heading 'Top-3-Znf And Rmh-2 Physically Interact And Form Protein Complexes') and locate each assay passage: first the yeast two-hybrid where RMH-2 is prey and TOP-3 is bait (C-terminally or N-terminally tagged) reporting a positive interaction (Figure 7C), then the co-immunoprecipitation passages describing the rmh-2::mCherry / top-3::HA strain and reciprocal co-IPs in which TOP-3 with or without the ZnF domain co-purified with RMH-2 (Supplementary Figure S5F). Populate one pandas DataFrame row per assay with columns assay, construct_pair, outcome, evidence_location, recording positive interaction outcomes and exact locators. Write to interaction_evidence_table.csv via pandas and validate row count, column completeness, and that each evidence_location cites the correct source.

## Related tools

- **Python** (Runs the extraction workflow: parsing the Results text, locating the Figure 7C and Supplementary Figure S5F passages, and assembling the per-assay rows)
- **pandas** (Builds the DataFrame with columns assay, construct_pair, outcome, evidence_location and writes interaction_evidence_table.csv)
- **Microsoft Excel** (Opens/inspects the generated interaction_evidence_table.csv for manual validation of row count and evidence_location fields)

## Examples

```
import pandas as pd; df = pd.DataFrame([{'assay':'yeast two-hybrid','construct_pair':'TOP-3 bait / RMH-2 prey','outcome':'positive interaction','evidence_location':'Figure 7C'},{'assay':'reciprocal co-IP','construct_pair':'rmh-2::mCherry / top-3::HA','outcome':'TOP-3 co-purified with RMH-2','evidence_location':'Supplementary Figure S5F'}]); df.to_csv('interaction_evidence_table.csv', index=False)
```

## Evaluation signals

- interaction_evidence_table.csv exists on disk after the run
- CSV contains exactly 2 data rows (one per assay: yeast two-hybrid and reciprocal co-IP)
- All four required columns (assay, construct_pair, outcome, evidence_location) are populated in every row
- The yeast two-hybrid row's evidence_location cites Figure 7C and the co-IP row cites Supplementary Figure S5F
- outcome fields record positive interaction results for both assays, consistent with the text stating RMH-2 and TOP-3 physically interact and form protein complexes

## Limitations

- Only assays explicitly reported with figure locators should be tabulated; RMH-1 pull-down experiments did not detect an interaction with RMH-2 and would need a separate negative-result treatment
- The co-IP evidence depends on the specific rmh-2::mCherry / top-3::HA strain; construct choices (TOP-3 with or without the ZnF domain) must be preserved in construct_pair to avoid over-claiming
- Evidence spans from supplementary material may not be present in the main text (e.g. Supplementary Figure S5F is cited but its content is in the supplement)

## Evidence

- [results] yeast two-hybrid passage reporting RMH-2 as prey and TOP-3 as bait: [evidence span withheld — anchor status: not_found]
- [methods] reciprocal co-IP evidence locator: [evidence span withheld — anchor status: not_found]
- [other] validation criteria for the extracted table: [evidence span withheld — anchor status: not_found]
- [discussion] biological conclusion supporting the positive interaction outcome: "Our biochemical and genetic analyses indicate that RMH-2 interacts with TOP-3 to perform some joint activities"
