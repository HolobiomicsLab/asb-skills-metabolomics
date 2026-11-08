---
name: yeast-two-hybrid-result-interpretation
description: Use when a Results passage describes a yeast two-hybrid assay with a
  bait/prey protein pair and a positive interaction that must be captured as one row
  of a physical-interaction evidence table.
when_to_use_negative:
- The assay described is a co-immunoprecipitation rather than a yeast two-hybrid;
  that result belongs in the co-IP row citing Supplementary Figure S5F, not Figure
  7C.
- The passage reports a negative or undetected interaction (e.g. the RMH-1 pull-down
  experiments that did not detect an interaction with RMH-2) and no positive yeast
  two-hybrid outcome is available to tabulate.
- You need the functional consequence of the interaction (e.g. stabilization of BTR
  complex at recombination intermediates) rather than the binary evidence that the
  proteins physically interact.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: Python
  role: Scripting language used to assemble the interaction-evidence rows into a pandas
    DataFrame and export the CSV
- name: pandas
  role: DataFrame library used to hold the two-row table with assay/construct_pair/outcome/evidence_location
    columns and write it to interaction_evidence_table.csv
- name: Microsoft Excel
  role: Spreadsheet application for inspecting the exported interaction_evidence_table.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:71425e970dbee6b1247a8ad961cbd915f3c3bf7fa8415668a81d93267c8a572a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Microsoft Excel
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# yeast-two-hybrid-result-interpretation

## Summary

Interprets a yeast two-hybrid assay reported as bait-prey protein pairs (e.g. TOP-3 as bait, RMH-2 as prey) as a positive physical-interaction result, recording the construct tagging (C-terminal or N-terminal) and the exact figure citation for downstream evidence tabulation.

## When to use

Use when a Results passage describes a yeast two-hybrid assay with a bait/prey protein pair and a positive interaction that must be captured as one row of a physical-interaction evidence table.

## When NOT to use

- The assay described is a co-immunoprecipitation rather than a yeast two-hybrid; that result belongs in the co-IP row citing Supplementary Figure S5F, not Figure 7C.
- The passage reports a negative or undetected interaction (e.g. the RMH-1 pull-down experiments that did not detect an interaction with RMH-2) and no positive yeast two-hybrid outcome is available to tabulate.
- You need the functional consequence of the interaction (e.g. stabilization of BTR complex at recombination intermediates) rather than the binary evidence that the proteins physically interact.

## Inputs

- Results-section text describing a yeast two-hybrid assay with bait/prey constructs (e.g. TOP-3 as bait, RMH-2 as prey)
- Figure/supplement citation for the assay result (e.g. Figure 7C)

## Outputs

- One-row interaction-evidence entry with fields assay, construct_pair, outcome, evidence_location
- interaction_evidence_table.csv containing both the yeast two-hybrid row and the reciprocal co-IP row

## How to apply

Locate the Results subsection reporting the yeast two-hybrid experiment (here 'Top-3-Znf And Rmh-2 Physically Interact And Form Protein Complexes') and identify the bait and prey constructs, including whether the bait is C-terminally or N-terminally tagged. Record the assay outcome (positive interaction) and the exact figure location (Figure 7C) so the row can be distinguished from complementary assays such as reciprocal co-immunoprecipitation of rmh-2::mCherry / top-3::HA, which cite Supplementary Figure S5F instead. Then write the row, with columns assay, construct_pair, outcome, evidence_location, into a pandas DataFrame and export to CSV alongside the co-IP rows; validation requires exactly two data rows, all four columns populated, and each evidence_location pointing to the correct assay's figure.

## Related tools

- **Python** (Scripting language used to assemble the interaction-evidence rows into a pandas DataFrame and export the CSV)
- **pandas** (DataFrame library used to hold the two-row table with assay/construct_pair/outcome/evidence_location columns and write it to interaction_evidence_table.csv)
- **Microsoft Excel** (Spreadsheet application for inspecting the exported interaction_evidence_table.csv)

## Examples

```
import pandas as pd; df = pd.DataFrame([{'assay':'yeast_two_hybrid','construct_pair':'TOP-3(bait)/RMH-2(prey)','outcome':'positive_interaction','evidence_location':'Figure 7C'}]); df.to_csv('interaction_evidence_table.csv')
```

## Evaluation signals

- interaction_evidence_table.csv contains exactly 2 data rows
- All four required columns (assay, construct_pair, outcome, evidence_location) are populated in every row
- The yeast two-hybrid row's evidence_location cites Figure 7C
- The reciprocal co-IP row's evidence_location cites Supplementary Figure S5F
- The yeast two-hybrid row records TOP-3 as bait and RMH-2 as prey with a positive interaction outcome

## Limitations

- A positive yeast two-hybrid interaction is reported per construct pair; a different bait tagging (C-terminal vs N-terminal) can define a separate construct pair, so partial-tagging information changes how rows should be split.
- Yeast two-hybrid is a binary presence/absence assay at this reporting granularity; it does not quantify interaction strength and must be corroborated by the reciprocal co-IP to support a claim of a physical complex in vivo.
- The interpretation covers only the protein-interaction evidence itself; mechanistic claims (e.g. RMH-2 cooperating with TOP-3 as part of a BTR subcomplex) remain speculative in the source article and are outside this skill's scope.

## Evidence

- [results] Yeast two-hybrid bait-prey outcome: [evidence span withheld — anchor status: not_found]
- [results] Complementary co-IP evidence location: [evidence span withheld — anchor status: not_found]
- [full_text] Positive-interaction outcome recorded in workflow: [evidence span withheld — anchor status: not_found]
