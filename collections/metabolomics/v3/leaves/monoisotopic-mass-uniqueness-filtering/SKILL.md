---
name: monoisotopic-mass-uniqueness-filtering
description: 'Use when converting MSFragger/FragPipe PSM results into a confident
  MALDI-MSI peptide imaging list: peptides with non-unique monoisotopic masses cannot
  be distinguished in an untargeted MS image and must be dropped before mapping peptides
  to proteins.'
when_to_use_negative:
- Input is raw LC-MS/MS spectra rather than an already-searched PSM table — the filter
  operates on identified PSMs, not on spectral preprocessing
- The analysis is targeted LC-MS/MS validation where isobaric discrimination is available
  and mass uniqueness is not required
- Only protein-level abundance or pathway analysis (e.g., Gene Ontology, STRING) is
  needed with no downstream MALDI-MSI imaging step
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0080
tools:
- name: MSIght
  role: Platform whose LC-MS/MS filtering cascade implements the unique-peptide, unique
    monoisotopic mass, and >=5-peptide-per-protein filters
  repo: github.com/lingjunli-research/MSIght
- name: Python
  role: Language in which the filtering cascade is written and distributed (pip-installable
    package with Jupyter notebook)
- name: MSFragger
  role: Producer of the PSM output table that serves as the filter's input at 1% protein
    FDR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:e1ae030e054c246e90a96423a45a2e94e2c3e4c78e8d63c24d089d0b4f0ea9ee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MSFragger
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# monoisotopic-mass-uniqueness-filtering

## Summary

A filtering step from the MSIght platform that removes LC-MS/MS peptide identifications whose monoisotopic masses are shared (within tolerance) with other peptides, so that only mass-discriminable peptides are imaged by MALDI-MSI. It is applied after unique-peptide filtering as part of MSIght's confirmation cascade.

## When to use

Use when converting MSFragger/FragPipe PSM results into a confident MALDI-MSI peptide imaging list: peptides with non-unique monoisotopic masses cannot be distinguished in an untargeted MS image and must be dropped before mapping peptides to proteins.

## When NOT to use

- Input is raw LC-MS/MS spectra rather than an already-searched PSM table — the filter operates on identified PSMs, not on spectral preprocessing
- The analysis is targeted LC-MS/MS validation where isobaric discrimination is available and mass uniqueness is not required
- Only protein-level abundance or pathway analysis (e.g., Gene Ontology, STRING) is needed with no downstream MALDI-MSI imaging step

## Inputs

- MSFragger/FragPipe PSM output table from LC-MS/MS, searched at 1% protein FDR (e.g., against the Mus musculus proteome)
- Peptide monoisotopic mass values with assigned protein mappings
- Tolerance setting (0.02 Da default; 0.1 Da margin alternative)

## Outputs

- Stepwise filtered peptide/protein table (filtered_protein_results.csv)
- Count summary of peptides and proteins remaining after each successive filter (unique peptides, unique monoisotopic mass, >=5 peptides per protein)
- Peptide list suitable for confident MALDI-MS image generation

## How to apply

Load the MSFragger/FragPipe PSM output table (searched against the Mus musculus proteome at 1% protein FDR) and first retain only unique peptide sequences, removing peptides mapping to multiple proteins. Then apply the monoisotopic mass uniqueness filter: within the 0.02 Da tolerance cited in the paper (with a 0.1 Da margin noted as an alternative), remove any peptide whose monoisotopic mass collides with another retained peptide. Rationale is that MALDI-MSI lacks high-throughput discrimination of isobaric and isomeric species, so only mass-unique peptides yield confident images. After filtering, aggregate qualifying peptides by protein and keep proteins supported by at least 5 qualifying peptides, tallying counts after each successive filter. In the MSIght implementation this cascade is scripted in Python and delivered as a distributable pip-installable package with a Jupyter notebook.

## Related tools

- **MSIght** (Platform whose LC-MS/MS filtering cascade implements the unique-peptide, unique monoisotopic mass, and >=5-peptide-per-protein filters) — github.com/lingjunli-research/MSIght
- **Python** (Language in which the filtering cascade is written and distributed (pip-installable package with Jupyter notebook))
- **MSFragger** (Producer of the PSM output table that serves as the filter's input at 1% protein FDR)

## Examples

```
pip install msight
```

## Evaluation signals

- Stepwise peptide and protein counts after each filter match the values reported in Figure 4 and Supplemental File 2 of the paper
- Removed peptides constitute up to ~70% of initially identified peptides, while no more than ~50% of initially identified proteins are filtered out
- Every surviving peptide's monoisotopic mass is unique within the chosen tolerance (0.02 Da, or 0.1 Da margin) across the retained set
- Every retained protein is supported by at least 5 qualifying unique peptides
- The output CSV (filtered_protein_results.csv) exists and is well-formed tabular data

## Limitations

- The mass uniqueness filter is aggressive: up to 70% of initially identified peptides can be removed, substantially shrinking the imageable peptide set
- Filter stringency depends on the tolerance choice — 0.02 Da is the cited default and 0.1 Da is an alternative margin, so results vary with the setting
- Confidence is bounded by the upstream search: identifications are only as good as the 1% protein FDR MSFragger search against the Mus musculus proteome

## Evidence
