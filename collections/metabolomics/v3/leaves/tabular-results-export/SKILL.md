---
name: tabular-results-export
description: Use when an Inventa prioritization run has finished computing the four
  rank components and the combined score needs to be persisted as a flat TSV in the
  results folder for downstream review or reporting.
when_to_use_negative:
- When the goal is to export an intermediate artifact such as the MEMO dissimilarity
  matrix or the cleaned GNPS annotation table rather than the final ranked extract
  results
- When the consumer requires a structured interchange format (e.g. mzML, JSON) rather
  than Inventa's flat tab-separated output
tools:
- name: Inventa
  role: Computes the four rank components and writes the consolidated .tsv results
    file with sample identifiers and final rank
  repo: https://github.com/luigiquiros/inventa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:023790be25b4855afce39ca4fe06ad4a9ed9c5cfd637e8e331de7328650f8cc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - Inventa
  - scikit-bio
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tabular-results-export

## Summary

Writes Inventa's computed components (FC, LC, CC, SC) and the final priority rank for each extract to a tab-separated (.tsv) results file, alongside the sample identifiers.

## When to use

Use when an Inventa prioritization run has finished computing the four rank components and the combined score needs to be persisted as a flat TSV in the results folder for downstream review or reporting.

## When NOT to use

- When the goal is to export an intermediate artifact such as the MEMO dissimilarity matrix or the cleaned GNPS annotation table rather than the final ranked extract results
- When the consumer requires a structured interchange format (e.g. mzML, JSON) rather than Inventa's flat tab-separated output

## Inputs

- Computed component scores (FC, LC, CC, SC) and combined priority rank per extract
- Metadata table (GNPS format with ATTRIBUTE_Species and ATTRIBUTE_Organe headers) providing sample identifiers
- Optional: MEMO vectorized dissimilarity matrix (e.g. Celastraceae_memomatrix.csv) from which SC was derived

## Outputs

- Tab-separated (.tsv) results file in the `results` folder containing all generated information per extract plus the final rank of the samples
- SC column values (0 or 1 per extract) with matching sample identifiers

## How to apply

After Inventa computes the Feature Component (FC), Literature Component (LC), Class Component (CC) and Similarity Component (SC) — the last derived from the MEMO vectorized dissimilarity matrix (e.g. Celastraceae_memomatrix.csv) via LOF, One-Class SVM and Isolation Forest outlier detection — collect all generated values together with the final rank of the samples. Write the table to the results folder as a TAB separated file, keeping the sample identifiers (species and organe from the GNPS-format metadata) joined to each component score and the combined rank. Component weights (wn) modulating the final rank must be reflected in the exported values. Validate the output by checking the file exists in `results`, is tab-separated, and contains the expected per-extract scores (e.g. SC = 1 for the Pristimera indica roots extract in the Celastraceae proof-of-concept).

## Related tools

- **Inventa** (Computes the four rank components and writes the consolidated .tsv results file with sample identifiers and final rank) — https://github.com/luigiquiros/inventa

## Evaluation signals

- Output file exists in the `results` folder and is tab-separated
- File contains an SC value of 1 for the Pristimera indica roots extract in the Celastraceae case study
- Every row carries a sample identifier consistent with the GNPS-format metadata table
- All four components (FC, LC, CC, SC) plus the combined rank are present per extract
- Modulated component weights (wn) are reflected in the exported combined rank values

## Limitations

- Binder instances shut down after 15 min of inactivity, so the exported results file must be saved locally before the session ends
- The output format is fixed to tab-separated text; only 'Peak area' or 'Peak height' quantification can be considered at a time upstream, which propagates to what the exported table summarizes

## Evidence

- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [methods] methods: "The rank comes from the addition of four individual components"
- [supplementary] task_006 workflow: [evidence span withheld — anchor status: not_found]
