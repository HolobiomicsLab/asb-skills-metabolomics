---
name: coimmunoprecipitation-result-curation
description: Use when a paper's Results section reports targeted physical-interaction
  assays — e.g. a yeast two-hybrid with named bait/prey constructs or co-IPs in a
  tagged strain — and the task is to extract each assay into a row-level evidence
  table with figure/supplement provenance.
when_to_use_negative:
- The article reports only high-throughput interaction datasets (e.g. AP-MS networks)
  rather than targeted low-throughput assays with named constructs and figure citations
- The task is to re-analyze raw co-IP or Y2H assay data rather than curate already-reported
  results from the text
- The source text does not name construct pairs or figure/supplement locations, so
  the evidence_location and construct_pair columns cannot be populated verifiably
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Executes the extraction and validation logic for building the interaction
    evidence table from the Results text
- name: pandas
  role: Constructs the two-row DataFrame with assay/construct_pair/outcome/evidence_location
    columns and writes interaction_evidence_table.csv
- name: Microsoft Excel
  role: Human-readable inspection of the curated interaction_evidence_table.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:8191e59b36b46bc2a0ae57adda7dce534c9eb3d6a425ff2fa1d6b37a31f5581c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Microsoft Excel
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# coimmunoprecipitation-result-curation

## Summary

Curates reported physical protein–protein interaction evidence (yeast two-hybrid and reciprocal co-immunoprecipitation assays) from a paper's Results text into a structured, provenance-cited table. In the TOP-3/RMH-2 case, this produces a two-row CSV recording the Y2H bait–prey result (Figure 7C) and the reciprocal co-IPs in the rmh-2::mCherry / top-3::HA strain (Supplementary Figure S5F).

## When to use

Use when a paper's Results section reports targeted physical-interaction assays — e.g. a yeast two-hybrid with named bait/prey constructs or co-IPs in a tagged strain — and the task is to extract each assay into a row-level evidence table with figure/supplement provenance.

## When NOT to use

- The article reports only high-throughput interaction datasets (e.g. AP-MS networks) rather than targeted low-throughput assays with named constructs and figure citations
- The task is to re-analyze raw co-IP or Y2H assay data rather than curate already-reported results from the text
- The source text does not name construct pairs or figure/supplement locations, so the evidence_location and construct_pair columns cannot be populated verifiably

## Inputs

- Article Results section text containing the Y2H and co-IP passages (e.g. 'Top-3-Znf And Rmh-2 Physically Interact And Form Protein Complexes')
- Named figure/supplement citations embedded in the text (Figure 7C; Supplementary Figure S5F)
- Construct/strain descriptions: TOP-3 bait (C- or N-terminally tagged), RMH-2 prey, rmh-2::mCherry / top-3::HA strain

## Outputs

- interaction_evidence_table.csv with columns assay, construct_pair, outcome, evidence_location and one row per reported assay (2 data rows for this paper)
- Per-row provenance strings tying each outcome to its figure/supplement citation

## How to apply

1. Load the relevant Results section (here, 'Top-3-Znf And Rmh-2 Physically Interact And Form Protein Complexes') and locate the yeast two-hybrid passage specifying the bait–prey orientation (TOP-3 as bait, C-terminally or N-terminally tagged; RMH-2 as prey) and its positive outcome with the exact figure citation (Figure 7C). 2. Locate the co-immunoprecipitation passages describing the epitagged strain (rmh-2::mCherry / top-3::HA) and the reciprocal co-IPs in which TOP-3 — with or without the ZnF domain — co-purified with RMH-2 (Supplementary Figure S5F). 3. In pandas, build a DataFrame with columns assay, construct_pair, outcome, evidence_location, adding one row per assay; record the outcome verbatim as reported (positive interaction), not an interpretation. 4. Write to interaction_evidence_table.csv with df.to_csv, then validate: the file exists, contains exactly the expected number of data rows (2 here), all four columns are populated, and each evidence_location cites the correct source (Figure 7C for Y2H; Supplementary Figure S5F for the reciprocal co-IP row). Rationale: separating assay type, construct pair, outcome, and provenance keeps the curated claim traceable to the exact figure, which matters when ZnF-domain presence/absence distinguishes the co-IP rows.

## Related tools

- **Python** (Executes the extraction and validation logic for building the interaction evidence table from the Results text)
- **pandas** (Constructs the two-row DataFrame with assay/construct_pair/outcome/evidence_location columns and writes interaction_evidence_table.csv)
- **Microsoft Excel** (Human-readable inspection of the curated interaction_evidence_table.csv)

## Examples

```
df = pd.DataFrame(rows, columns=['assay','construct_pair','outcome','evidence_location']); df.to_csv('interaction_evidence_table.csv', index=False)
```

## Evaluation signals

- interaction_evidence_table.csv exists on disk after the run
- The CSV contains exactly 2 data rows (one per assay: yeast two-hybrid; reciprocal co-IP)
- All four required columns (assay, construct_pair, outcome, evidence_location) are non-empty in every row
- The Y2H row's evidence_location cites Figure 7C and the co-IP row cites Supplementary Figure S5F — no cross-citation errors
- Each outcome field reports the interaction as a positive result, matching the article's claim that TOP-3 and RMH-2 physically interact

## Limitations

- Downstream biological interpretation is uncertain: the article states RMH-2 and TOP-3 'perform some joint activities, possibly as part of a BTR subcomplex' — the table records the interaction evidence but not the mechanism
- Reproducibility caveats carry over: RMH-2::FLAG expression in the top-3-ZnF background showed highly variable levels with large standard deviations, and no biological replicates or sample sizes are stated for the described experiments
- Some referenced supplements (e.g. Supplemental Figure S7) may be absent from the provided text, so only citations actually located in the text should be entered in evidence_location

## Evidence

- [results] A yeast two-hybrid assay using RMH-2 as prey and TOP-3 as bait (C-terminally or N-terminally tagged) revealed a physical interaction between the two proteins (Figure 7C).: [evidence span withheld — anchor status: not_found]
- [methods] the rmh-2::mCherry / top-3::HA strain and the reciprocal co-IPs in which TOP-3 (with or without the ZnF domain) co-purified with RMH-2 (Supplementary Figure S5F): [evidence span withheld — anchor status: not_found]
- [discussion] Our biochemical and genetic analyses indicate that RMH-2 interacts with TOP-3 to perform some joint activities: "Our biochemical and genetic analyses indicate that RMH-2 interacts with TOP-3 to perform some joint activities"
