---
name: summary-table-assembly
description: Use when a task requires pulling exact reported numbers (e.g. 518 total
  metabolites, 313 positive-mode, 205 negative-mode) out of a Results-section text
  or figure legend and packaging them into a machine-readable summary table.
when_to_use_negative:
- Input is already a machine-readable feature table or peak list from LC-MS processing
  — the skill targets extraction of author-reported counts from prose/figures, not
  re-tabulation of raw data
- The goal is to re-run metabolite identification or peak detection in PlantMetSuite
  to recompute counts — this skill transcribes and validates reported values rather
  than reproducing the upstream analysis
- Reported values lack an internal cross-check (e.g. no mode-specific breakdown that
  must sum to a total), so the consistency-verification step cannot be performed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: PlantMetSuite
  role: Web-based metabolomics analysis and visualisation tool whose published case-study
    counts (acd5 vs Col-0) are the values being extracted and tabulated
  repo: yaonanlab/plantmetsuite
- name: manual expert review
  role: Optional human cross-check that the transcribed counts match the source text
    and figure legend before finalising the CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:a210803f1c1a01a7a0f4c9d8cf553786f5013556632e81447e53ad6d3c6a3697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yaonanlab/plantmetsuite
  tools:
  - PlantMetSuite
  - manual expert review
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# summary-table-assembly

## Summary

Extracts the specific quantitative values reported in a publication section (here, LC-MS peak counts and metabolite identification counts from the acd5 vs Col-0 Arabidopsis thaliana case study) and assembles them into a tidy metric/ionization_mode/value table with an internal consistency check, exported as UTF-8 CSV.

## When to use

Use when a task requires pulling exact reported numbers (e.g. 518 total metabolites, 313 positive-mode, 205 negative-mode) out of a Results-section text or figure legend and packaging them into a machine-readable summary table.

## When NOT to use

- Input is already a machine-readable feature table or peak list from LC-MS processing — the skill targets extraction of author-reported counts from prose/figures, not re-tabulation of raw data
- The goal is to re-run metabolite identification or peak detection in PlantMetSuite to recompute counts — this skill transcribes and validates reported values rather than reproducing the upstream analysis
- Reported values lack an internal cross-check (e.g. no mode-specific breakdown that must sum to a total), so the consistency-verification step cannot be performed

## Inputs

- Results-section text (e.g. Section 2.3) describing the untargeted LC-MS metabolomic profiling of A. thaliana Col-0 vs acd5
- Supplementary figure asset (e.g. Figure S1) containing the reported peak/identification counts
- List of metrics to extract (positive-mode peaks, negative-mode peaks, total and per-mode metabolite identifications)

## Outputs

- results_summary_table.csv (UTF-8 CSV) with columns metric, ionization_mode, value — one row per reported quantity

## How to apply

Load the Results text (e.g. Section 2.3) and any supplementary figure asset (e.g. Figure S1) supplied as input files. Enumerate exactly the quantitative metrics the task names — for this case study: peak count in positive mode, peak count in negative mode, total identified metabolites, metabolites identified in positive mode, and metabolites identified in negative mode — taken from the untargeted LC-MS profiling of wild-type A. thaliana versus the ceramide kinase mutant acd5. Assemble them into a tidy pandas DataFrame with columns metric, ionization_mode, and value so each reported number is one observation. Verify internal consistency by checking that the mode-specific metabolite identifications (313 + 205) sum to the reported total of 518 before writing anything. Finally, write the table to results_summary_table.csv in UTF-8 CSV format, and optionally cross-check the transcribed values against the source text or by manual expert review.

## Related tools

- **PlantMetSuite** (Web-based metabolomics analysis and visualisation tool whose published case-study counts (acd5 vs Col-0) are the values being extracted and tabulated) — yaonanlab/plantmetsuite
- **manual expert review** (Optional human cross-check that the transcribed counts match the source text and figure legend before finalising the CSV)

## Examples

```
import pandas as pd; df = pd.DataFrame([{'metric':'identified_metabolites','ionization_mode':'positive','value':313},{'metric':'identified_metabolites','ionization_mode':'negative','value':205},{'metric':'identified_metabolites','ionization_mode':'total','value':518}]); assert df[df.metric=='identified_metabolites'].query("ionization_mode!='total'").value.sum()==518; df.to_csv('results_summary_table.csv', index=False, encoding='utf-8')
```

## Evaluation signals

- Sum invariant holds: metabolites identified in positive mode (313) plus negative mode (205) equals the reported total of 518
- Output CSV parses as UTF-8 and has exactly the tidy schema with columns metric, ionization_mode, and value
- All five required metrics (positive-mode peaks, negative-mode peaks, total identified metabolites, per-mode identifications) are present as distinct rows
- Transcribed values match the verbatim numbers in the Results 2.3 text / Figure S1 caption
- Ionization-mode labels are consistent across rows (e.g. 'positive' / 'negative' used uniformly, positive-mode counts higher than negative-mode in this dataset)

## Limitations

- Values are transcriptions of author-reported counts, not recomputed results — the skill inherits any errors in the source text and does not re-run peak detection or metabolite identification
- PlantMetSuite is browser-accessible without installation or programming skills, so reproducibility of the underlying analysis pipeline depends on the tool's current web implementation; no changelog was found, complicating version tracking
- The consistency check only works when the report provides both a total and an additive breakdown (mode-specific counts); tables built from non-additive metrics cannot be internally verified this way

## Evidence

- [abstract] Tool and domain grounding: "we developed PlantMetSuite, a web-based tool for comprehensive metabolomics analysis and visualisation"
- [abstract] Usability / reproducibility caveat: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills."
- [discussion] Version-tracking limitation: "Source: github:github.com__YaonanLab__PlantMetSuite"
