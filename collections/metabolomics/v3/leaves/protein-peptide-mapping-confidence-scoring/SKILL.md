---
name: protein-peptide-mapping-confidence-scoring
description: Use when raw MSFragger/FragPipe PSM tables (e.g., searched against a
  proteome like Mus musculus at 1% protein FDR) must be filtered into a confident
  peptide/protein set for validating MSI peptide identifications.
when_to_use_negative:
- The input is already a filtered, protein-level feature table rather than raw PSM-level
  output — the cascade assumes unfiltered MSFragger PSM identifications.
- The goal is MSI image processing, co-registration of H&E stains, or MSI clustering
  rather than LC-MS/MS peptide confirmation (those are separate MSIght modules).
- No monoisotopic mass information is available in the PSM table, making the mass-uniqueness
  filter inapplicable without re-running the search.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MSIght
  role: Platform whose LC-MS/MS confirmation module automates the unique-peptide,
    monoisotopic-mass, and >=5-peptides-per-protein filtering cascade used to confirm
    putative MSI identifications
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language for reproducing the filtering cascade and writing
    filtered_protein_results.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:1426063320ba1ffffe8c196524198537c911ca25783b445a3d4590713973c1d0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# protein-peptide-mapping-confidence-scoring

## Summary

Reproduces MSIght's LC-MS/MS peptide confirmation cascade, applying successive filters (unique peptides, unique monoisotopic mass at 0.02 Da tolerance, >=5 peptides per protein) to MSFragger PSM output at 1% protein FDR so that MALDI-MSI peptide images are annotated with high-confidence protein-level support.

## When to use

Use when raw MSFragger/FragPipe PSM tables (e.g., searched against a proteome like Mus musculus at 1% protein FDR) must be filtered into a confident peptide/protein set for validating MSI peptide identifications.

## When NOT to use

- The input is already a filtered, protein-level feature table rather than raw PSM-level output — the cascade assumes unfiltered MSFragger PSM identifications.
- The goal is MSI image processing, co-registration of H&E stains, or MSI clustering rather than LC-MS/MS peptide confirmation (those are separate MSIght modules).
- No monoisotopic mass information is available in the PSM table, making the mass-uniqueness filter inapplicable without re-running the search.

## Inputs

- MSFragger/FragPipe PSM output table from LC-MS/MS analysis (searched at 1% protein FDR, e.g., against the Mus musculus proteome)
- Peptide-to-protein mapping (to identify peptides mapping to multiple proteins)
- Peptide monoisotopic mass values for the uniqueness filter

## Outputs

- Stepwise filtered peptide/protein table (filtered_protein_results.csv)
- Count summary of peptides and proteins remaining after each successive filter, comparable to Figure 4 and Supplemental File 2

## How to apply

Load the MSFragger PSM output table from the deposited LC-MS/MS results file and first restrict to unique peptide sequences, dropping peptides that map to multiple proteins. Next apply the monoisotopic mass uniqueness filter, requiring each peptide's monoisotopic mass to be unique within the paper's 0.02 Da tolerance (a 0.1 Da margin is noted as an alternative). Aggregate the surviving peptides by protein and retain only proteins supported by at least 5 qualifying peptides. Tally peptides and proteins after each successive filter and compare to Figure 4 and Supplemental File 2, where the unique-peptide and monoisotopic-mass filters remove up to 70% of initially identified peptides while no more than 50% of proteins are lost. Write the stepwise filtered table and count summary to filtered_protein_results.csv as the validation artifact.

## Related tools

- **MSIght** (Platform whose LC-MS/MS confirmation module automates the unique-peptide, monoisotopic-mass, and >=5-peptides-per-protein filtering cascade used to confirm putative MSI identifications) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language for reproducing the filtering cascade and writing filtered_protein_results.csv)

## Examples

```
pip install msight
```

## Evaluation signals

- filtered_protein_results.csv exists and is well-formed tabular output
- Stepwise peptide counts after each filter match the reduction reported in Figure 4 (up to 70% of initially identified peptides removed by unique-peptide + monoisotopic-mass filters)
- Protein counts match Figure 4 / Supplemental File 2 (no more than 50% of initially identified proteins filtered out)
- Every retained protein is supported by at least 5 qualifying peptides after both uniqueness filters
- All retained peptides have a monoisotopic mass unique within the 0.02 Da tolerance

## Limitations

- The 0.1 Da alternative margin is noted as an alternative to the 0.02 Da tolerance; choice of tolerance changes which peptides pass the monoisotopic mass uniqueness filter.
- The monoisotopic mass filter can be aggressive: combined with unique-peptide filtering it removed up to 70% of initially identified peptides in the paper's evaluation.
- Protein-level inference from peptide filtering depends on the >=5 peptides-per-protein cutoff, which may discard low-abundance proteins supported by few peptides.

## Evidence
