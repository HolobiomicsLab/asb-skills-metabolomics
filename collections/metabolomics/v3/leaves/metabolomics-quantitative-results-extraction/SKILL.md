---
name: metabolomics-quantitative-results-extraction
description: Use when a source card asks to reproduce reported metabolomics counts
  (peaks/identifications per ionisation mode) from a paper's Results text and supplementary
  figures, and the deliverable is a validated summary table rather than re-processed
  spectra.
when_to_use_negative:
- Input is raw LC–MS spectral data (mzML/peak tables) requiring feature detection
  and metabolite identification rather than extraction of already-reported counts
- The task requires de novo statistical analysis or visualisation of metabolomics
  data (use PlantMetSuite's analysis modules directly instead)
- The paper does not report per-mode peak/metabolite counts, so the five-value extraction
  schema cannot be populated
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PlantMetSuite
  role: Web-based plant metabolomics analysis and visualisation tool whose reported
    outputs (e.g. the acd5 vs Col-0 case study counts) are the quantities extracted
    and cross-checked by this skill
  repo: github.com/YaonanLab/PlantMetSuite
- name: manual expert review
  role: Adjudication of ambiguous or figure-derived count values before writing the
    summary table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:5155ccd5bfd15a41a3ff772c40f1c941f3724a0ff74a3d4fa7e769aecd0f7c3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PlantMetSuite
  - manual expert review
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolomics-quantitative-results-extraction

## Summary

Extracts reported quantitative outputs from an untargeted LC–MS metabolomics publication — peak counts per ionisation mode and metabolite identification counts per mode plus total — into a verified, tidy CSV summary. It reproduces the paper's numbers (e.g. the acd5 vs Col-0 case study: 518 metabolites total, 313 positive mode, 205 negative mode) rather than re-running the analysis.

## When to use

Use when a source card asks to reproduce reported metabolomics counts (peaks/identifications per ionisation mode) from a paper's Results text and supplementary figures, and the deliverable is a validated summary table rather than re-processed spectra.

## When NOT to use

- Input is raw LC–MS spectral data (mzML/peak tables) requiring feature detection and metabolite identification rather than extraction of already-reported counts
- The task requires de novo statistical analysis or visualisation of metabolomics data (use PlantMetSuite's analysis modules directly instead)
- The paper does not report per-mode peak/metabolite counts, so the five-value extraction schema cannot be populated

## Inputs

- Results section text (e.g. Results 2.3) as a text/input file
- Supplementary figure asset (e.g. Figure S1) supplied as an input file
- Source card with research_question, finding, and workflow fields

## Outputs

- results_summary_table.csv — UTF-8 CSV with columns metric, ionization_mode, value
- Consistency check result (313 + 205 == 518 for the acd5 vs Col-0 case)

## How to apply

Load the Results section text (e.g. Results 2.3 for the acd5 vs Col-0 Arabidopsis thaliana case study) and the supplementary figure asset (e.g. Figure S1) supplied as input files. Extract the five reported quantitative values: peak count in positive mode, peak count in negative mode, total identified metabolites, metabolites identified in positive mode, and metabolites identified in negative mode. Assemble them into a tidy pandas DataFrame with columns metric, ionization_mode, and value. Verify internal consistency by checking that the mode-specific metabolite identifications (313 + 205) sum to the reported total of 518; raise a flag if they do not. Write the table to results_summary_table.csv in UTF-8 CSV format. Where the source analysis used PlantMetSuite — a browser-accessible, no-installation web tool for plant metabolomics analysis and visualisation — cross-check figure-derived values against the tool's own outputs, and rely on manual expert review to adjudicate any ambiguity in the reported counts.

## Related tools

- **PlantMetSuite** (Web-based plant metabolomics analysis and visualisation tool whose reported outputs (e.g. the acd5 vs Col-0 case study counts) are the quantities extracted and cross-checked by this skill) — github.com/YaonanLab/PlantMetSuite
- **manual expert review** (Adjudication of ambiguous or figure-derived count values before writing the summary table)

## Examples

```
import pandas as pd; df = pd.DataFrame({'metric':['peaks','peaks','metabolites','metabolites','metabolites'],'ionization_mode':['positive','negative','total','positive','negative'],'value':[None,None,518,313,205]}); df.to_csv('results_summary_table.csv', encoding='utf-8')
```

## Evaluation signals

- Mode-specific metabolite identifications sum to the reported total: 313 (positive) + 205 (negative) == 518
- All five metrics present: peak counts (positive and negative mode), total identified metabolites, and per-mode metabolite identifications
- Output CSV parses with exactly three columns — metric, ionization_mode, value — in UTF-8 encoding
- Extracted values match the paper's reported numbers when diffed against the finding field of the source card
- Values cross-checked against the supplementary figure (Figure S1) and, where available, PlantMetSuite's own reported outputs

## Limitations

- Reproduces only counts as reported in text/figures; it does not validate the upstream peak detection or metabolite identification pipeline itself
- Figure-derived values (e.g. from Figure S1) may be ambiguous and require manual expert review
- Depends on the source paper reporting both mode-specific and total identification numbers; partial reporting breaks the consistency check
- PlantMetSuite is browser-based with no changelog available in the extracted discussion, so version drift between reported results and the current tool release cannot be ruled out

## Evidence

- [abstract] PlantMetSuite abstract: "we developed PlantMetSuite, a web-based tool for comprehensive metabolomics analysis and visualisation"
- [discussion] PlantMetSuite discussion: "Source: github:github.com__YaonanLab__PlantMetSuite"
