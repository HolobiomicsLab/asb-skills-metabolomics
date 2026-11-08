---
name: lc-msms-workflow-result-integration
description: Use when raw PSM tables from LC-MS/MS peptide searches must be filtered
  (unique peptides, unique monoisotopic mass, >=5 peptides/protein) to confirm putative
  MSI identifications or replicate Figure 4-style filtering cascades.
when_to_use_negative:
- Input is already a filtered protein/peptide table (e.g., only proteins with >=5
  unique peptides) — reapplying the cascade would double-filter
- Data are unfiltered MSI spectra (mzML/imzML) requiring pre-processing or registration
  rather than LC-MS/MS PSM filtering — use the MSI pre-processing or H&E registration
  workflows instead
- Identifications come from targeted LC-MS/MS rather than the untargeted global approach
  MSIght is designed for
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSIght
  role: Platform that automates integration of LC-MS/MS peptide identifications with
    MSI data; supplies the filtering cascade confirmed against Figure 4
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language for the filtering script (loading PSM tables, applying
    unique-peptide/monoisotopic-mass/5-peptide filters, writing filtered_protein_results.csv)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:220766ff8cabebff467736ca1b8729de00814a8b56cb87f8f71e58a804e81ce6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lc-msms-workflow-result-integration

## Summary

A method for turning untargeted LC-MS/MS PSM results (e.g., MSFragger/FragPipe output searched against the Mus musculus proteome at 1% protein FDR) into a confident set of peptide identifications used to confirm and annotate MALDI-MS images in the MSIght pipeline.

## When to use

Use when raw PSM tables from LC-MS/MS peptide searches must be filtered (unique peptides, unique monoisotopic mass, >=5 peptides/protein) to confirm putative MSI identifications or replicate Figure 4-style filtering cascades.

## When NOT to use

- Input is already a filtered protein/peptide table (e.g., only proteins with >=5 unique peptides) — reapplying the cascade would double-filter
- Data are unfiltered MSI spectra (mzML/imzML) requiring pre-processing or registration rather than LC-MS/MS PSM filtering — use the MSI pre-processing or H&E registration workflows instead
- Identifications come from targeted LC-MS/MS rather than the untargeted global approach MSIght is designed for

## Inputs

- MSFragger/FragPipe PSM output table from LC-MS/MS searched against the Mus musculus proteome at 1% protein FDR
- Peptide-to-protein mapping (to determine peptide uniqueness)
- Monoisotopic mass values per identified peptide

## Outputs

- filtered_protein_results.csv — stepwise filtered peptide/protein table
- Stepwise count summary of peptides and proteins remaining after each filter (comparable to Figure 4 and Supplemental File 2)

## How to apply

Load the MSFragger/FragPipe PSM output table (searched against the Mus musculus proteome at 1% protein FDR) from the deposited LC-MS/MS results file. Filter to retain only PSMs with unique peptide sequences, removing peptides mapping to multiple proteins. Apply the monoisotopic mass uniqueness filter, requiring each peptide's monoisotopic mass to be unique within the 0.02 Da tolerance cited in the paper (with a 0.1 Da margin noted as an alternative). Aggregate qualifying peptides by protein and retain only proteins supported by at least 5 qualifying peptides. Tally peptides and proteins remaining after each successive filter; in the paper, the unique-peptide and unique-monoisotopic-mass filters removed up to 70% of initially identified peptides while no more than 50% of initially identified proteins were filtered. Write the stepwise filtered peptide/protein table and count summary (filtered_protein_results.csv) and compare against Figure 4 and Supplemental File 2.

## Related tools

- **MSIght** (Platform that automates integration of LC-MS/MS peptide identifications with MSI data; supplies the filtering cascade confirmed against Figure 4) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language for the filtering script (loading PSM tables, applying unique-peptide/monoisotopic-mass/5-peptide filters, writing filtered_protein_results.csv))

## Examples

```
pip install msight
```

## Evaluation signals

- filtered_protein_results.csv exists and is well-formed tabular output
- Stepwise peptide counts after the unique-peptide and unique-monoisotopic-mass (0.02 Da) filters match values in Figure 4 and Supplemental File 2
- Stepwise protein counts show no more than 50% of initially identified proteins removed by the filters
- Every retained protein is supported by at least 5 unique peptides with unique monoisotopic mass
- All input PSMs derive from a search at 1% protein FDR against the Mus musculus proteome

## Limitations

- The unique-peptide plus unique-monoisotopic-mass filters can remove up to 70% of initially identified peptides, substantially shrinking the confirmable peptide set
- The monoisotopic mass uniqueness tolerance is sensitive to the chosen window (0.02 Da cited; 0.1 Da noted as an alternative), affecting which peptides pass
- Counts should be verified against Figure 4 and Supplemental File 2; the index notes publication metadata (DOI/authors/journal) is not stated in the abstract itself

## Evidence
