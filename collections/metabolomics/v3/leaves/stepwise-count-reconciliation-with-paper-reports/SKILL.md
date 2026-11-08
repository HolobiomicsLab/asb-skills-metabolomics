---
name: stepwise-count-reconciliation-with-paper-reports
description: Use when reproducing a paper's filtering cascade on its deposited data
  and you need to verify each intermediate count (peptides/proteins remaining per
  filter) matches the reported figure or supplemental table.
when_to_use_negative:
- Input is already a filtered protein list with no PSM-level identifications, so the
  stepwise filter trajectory cannot be recomputed
- The analysis target is MSI image generation or histology co-registration (MSIght's
  cluster_msi / registration pipelines), not LC-MS/MS PSM filtering
- The source paper reports no intermediate per-filter counts, leaving nothing to reconcile
  against
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MSIght
  role: Defines the filtering cascade being reproduced (unique peptides, unique monoisotopic
    mass, >=5 peptides per protein) that generates confident MALDI-MS peptide images
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Language used to implement the stepwise filtering and count reconciliation
    against the paper-reported values
  repo: https://github.com/lingjunli-research/MSIght
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:6b2636dbbc523a44e141460a51781c3bf728c5f8372c755f55586dff26020b9b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# stepwise-count-reconciliation-with-paper-reports

## Summary

A validation skill that reproduces a published multi-step filtering cascade (MSIght's unique-peptide, unique monoisotopic mass, and >=5 peptides/protein filters) and reconciles the stepwise peptide/protein counts against the counts reported in the paper's figures (Figure 4) and supplemental files.

## When to use

Use when reproducing a paper's filtering cascade on its deposited data and you need to verify each intermediate count (peptides/proteins remaining per filter) matches the reported figure or supplemental table.

## When NOT to use

- Input is already a filtered protein list with no PSM-level identifications, so the stepwise filter trajectory cannot be recomputed
- The analysis target is MSI image generation or histology co-registration (MSIght's cluster_msi / registration pipelines), not LC-MS/MS PSM filtering
- The source paper reports no intermediate per-filter counts, leaving nothing to reconcile against

## Inputs

- MSFragger/FragPipe PSM output table from deposited LC-MS/MS results (searched against the Mus musculus proteome at 1% protein FDR)
- Paper's Figure 4 reported peptide/protein counts (stepwise filter counts)
- Supplemental File 2 (filtered peptide/protein results for comparison)

## Outputs

- Stepwise filtered peptide/protein table (peptides and proteins remaining after each successive filter)
- Count summary CSV (e.g. filtered_protein_results.csv) with per-filter peptide and protein tallies for reconciliation against Figure 4

## How to apply

Load the MSFragger/FragPipe PSM output table (Mus musculus proteome search at 1% protein FDR) and apply MSIght's successive filters in the published order: first retain only PSMs with unique peptide sequences (removing peptides mapping to multiple proteins), then apply the monoisotopic mass uniqueness filter with the 0.02 Da tolerance cited in the paper (0.1 Da margin noted as an alternative), then aggregate by protein and keep proteins supported by at least 5 qualifying peptides. After each filter, tally the remaining peptides and proteins so the count trajectory is recoverable step by step. Write both the stepwise-filtered table and a count summary (e.g. filtered_protein_results.csv), then diff the counts against the values in Figure 4 and Supplemental File 2. The published expectation is that unique-peptide plus unique-monoisotopic-mass filtering removes up to 70% of initially identified peptides, while no more than 50% of initially identified proteins are filtered out; a reproduction whose counts diverge materially from these bounds or from Figure 4 indicates a misapplied tolerance, wrong FDR level, or a non-unique-peptide definition.

## Related tools

- **MSIght** (Defines the filtering cascade being reproduced (unique peptides, unique monoisotopic mass, >=5 peptides per protein) that generates confident MALDI-MS peptide images) — https://github.com/lingjunli-research/MSIght
- **Python** (Language used to implement the stepwise filtering and count reconciliation against the paper-reported values) — https://github.com/lingjunli-research/MSIght

## Examples

```
python reproduce_filter_cascade.py --psm msfragger_psm.tsv --fdr 0.01 --mass-tol 0.02 --min-peptides 5 --out filtered_protein_results.csv
```

## Evaluation signals

- filtered_protein_results.csv exists and is well-formed tabular output
- Stepwise peptide counts after each filter match the values reported in Figure 4 of the paper
- Stepwise protein counts match the values in Supplemental File 2
- Fraction of peptides removed by the unique-peptide + unique-monoisotopic-mass filters does not exceed ~70% of initially identified peptides
- Fraction of proteins filtered out does not exceed ~50% of initially identified proteins

## Limitations

- The monoisotopic mass uniqueness tolerance is cited as 0.02 Da with an additional 0.1 Da margin noted as an alternative; the choice of tolerance changes which peptides survive the filter
- Reconciliation depends on the deposited MSFragger PSM output and the exact protein FDR level (1%); any deviation in search database (Mus musculus) or FDR changes the count trajectory
- Counts are specific to this paper's Figure 4 and Supplemental File 2; other datasets require recomputing reference counts rather than reusing these thresholds as pass/fail bounds

## Evidence

- [results] filter cascade removes up to 70% of peptides but <=50% of proteins: [evidence span withheld — anchor status: not_found]
- [results] unique-peptide filter rationale: [evidence span withheld — anchor status: not_found]
- [results] >=5 peptides per protein and mass tolerance: [evidence span withheld — anchor status: not_found]
