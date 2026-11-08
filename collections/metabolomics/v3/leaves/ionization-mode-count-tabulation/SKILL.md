---
name: ionization-mode-count-tabulation
description: Use when a results-section text or figure reports metabolite/peak counts
  per ionisation mode (e.g. positive and negative ESI) and the counts must be extracted,
  consistency-checked against a stated total, and tabulated.
when_to_use_negative:
- Input is already a processed feature/peak table or an abundance matrix rather than
  reported count values in prose or a figure
- The task requires re-running untargeted LC–MS peak detection or metabolite identification
  from raw spectra instead of reproducing reported counts
- No per-ionisation-mode breakdown (positive vs negative ESI) is reported or derivable
  from the supplied text/figure
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PlantMetSuite
  role: Source tool whose untargeted LC–MS metabolomics workflow produced the reported
    per-mode metabolite identification counts being tabulated
  repo: github.com/YaonanLab/PlantMetSuite
- name: pandas
  role: Used to assemble the extracted counts into a tidy table (columns metric, ionization_mode,
    value) and write results_summary_table.csv
- name: manual expert review
  role: 'Verification step: expert check that extracted values match the source text
    and that mode-specific counts sum to the total'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:d44caf3c848b76413866d85628f14e1ac05c59efbfbae97af5136899775c702e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PlantMetSuite
  - manual expert review
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# ionization-mode-count-tabulation

## Summary

Tabulates reported peak and metabolite-identification counts split by ESI ionisation mode (positive/negative) from an LC–MS metabolomics results section, and verifies that mode-specific metabolite counts sum to the reported total. Reproduces the acd5 vs Col-0 reporting (518 total; 313 positive; 205 negative) as an auditable tidy table.

## When to use

Use when a results-section text or figure reports metabolite/peak counts per ionisation mode (e.g. positive and negative ESI) and the counts must be extracted, consistency-checked against a stated total, and tabulated.

## When NOT to use

- Input is already a processed feature/peak table or an abundance matrix rather than reported count values in prose or a figure
- The task requires re-running untargeted LC–MS peak detection or metabolite identification from raw spectra instead of reproducing reported counts
- No per-ionisation-mode breakdown (positive vs negative ESI) is reported or derivable from the supplied text/figure

## Inputs

- Results section text reporting per-mode peak and metabolite counts (e.g. Results 2.3 for the acd5 vs Col-0 case study)
- Supplementary figure asset containing the ionisation-mode breakdown (e.g. Figure S1)

## Outputs

- results_summary_table.csv (UTF-8 CSV with columns metric, ionization_mode, value)

## How to apply

Load the Results section text (e.g. Results 2.3) and the supplementary figure asset (e.g. Figure S1) supplied as inputs. Extract the five reported quantitative values: peak count in positive mode, peak count in negative mode, total identified metabolites, metabolites identified in positive mode, and metabolites identified in negative mode. Assemble them into a tidy pandas DataFrame with columns metric, ionization_mode, and value so each count is one row. Verify internal consistency by checking that the mode-specific metabolite identifications (313 + 205) sum to the reported total (518); flag any mismatch before writing output. Write the table to results_summary_table.csv in UTF-8 CSV format.

## Related tools

- **PlantMetSuite** (Source tool whose untargeted LC–MS metabolomics workflow produced the reported per-mode metabolite identification counts being tabulated) — github.com/YaonanLab/PlantMetSuite
- **pandas** (Used to assemble the extracted counts into a tidy table (columns metric, ionization_mode, value) and write results_summary_table.csv)
- **manual expert review** (Verification step: expert check that extracted values match the source text and that mode-specific counts sum to the total)

## Examples

```
import pandas as pd; df = pd.DataFrame({'metric':['peaks','peaks','metabolites','metabolites','metabolites'],'ionization_mode':['positive','negative','total','positive','negative'],'value':[None,None,518,313,205]}); df.to_csv('results_summary_table.csv', index=False, encoding='utf-8')
```

## Evaluation signals

- Mode-specific metabolite identifications sum to the reported total: 313 (positive) + 205 (negative) = 518
- Output CSV contains exactly the five expected metrics (peaks positive, peaks negative, total metabolites, metabolites positive, metabolites negative)
- CSV schema matches columns metric, ionization_mode, value and the file is UTF-8 encoded
- Values in the table match verbatim the numbers stated in the Results 2.3 text / Figure S1
- Manual expert review confirms no transcription errors in extracted counts

## Limitations

- Reproduces only the counts reported in the source text/figure; it does not re-run peak detection or metabolite identification on raw LC–MS data
- Depends on the source clearly attributing counts to positive vs negative ionisation mode; ambiguous reporting breaks the tabulation
- Consistency check only validates that mode counts sum to the total; it cannot detect errors present in the source itself

## Evidence

- [results] Untargeted metabolomic profiling of wild-type A. thaliana and acd5 identified 518 metabolites in total, comprising 313 metabolites in the positive mode and 205 metabolites in the negative mode.: [evidence span withheld — anchor status: not_found]
- [abstract] PlantMetSuite provides interactive bioinformatics tools and databases tailored to plant metabolomics data: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics"
- [discussion] PlantMetSuite is browser-accessible without installation or programming skills: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills. The tool is freely available and will undergo regular updates and expansions"
