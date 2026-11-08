---
name: spectral-similarity-scoring
description: Use when reproducing or auditing GNPS molecular networking edges from
  a .mgf consensus spectra file plus an edge results table, to verify that computed
  modified cosine scores match GNPS's reported cosine_score per node pair.
when_to_use_negative:
- The goal is qualitative molecular networking or library annotation itself rather
  than reconstructing/verifying the pairwise edge scoring
- Input is already a feature table or quantified abundance matrix — classic molecular
  networking uses only MS2 scans and provides no quantitative information
- Calibrated quantitation is required — classic molecular networking is qualitative
  and would need follow-up targeted LC-MS
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Platform whose molecular networking workflow (METABOLOMICS-SNETS-V2) generates
    the .mgf consensus spectra, edge table, and reference cosine_score values being
    reconstructed
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw data files to open formats (.mzXML, .mzML,
    .mgf) before scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:8fe99de0a9e2332173e8c664fda584a0e82cc2e5a2407cc32a1596484f0a4b54
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MSConvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs and validates GNPS's modified cosine pairwise MS2 spectral alignment — the scoring function that assigns cosine-similarity edges between nodes in molecular networking.

## When to use

Use when reproducing or auditing GNPS molecular networking edges from a .mgf consensus spectra file plus an edge results table, to verify that computed modified cosine scores match GNPS's reported cosine_score per node pair.

## When NOT to use

- The goal is qualitative molecular networking or library annotation itself rather than reconstructing/verifying the pairwise edge scoring
- Input is already a feature table or quantified abundance matrix — classic molecular networking uses only MS2 scans and provides no quantitative information
- Calibrated quantitation is required — classic molecular networking is qualitative and would need follow-up targeted LC-MS

## Inputs

- Consensus spectra file (.mgf) from a GNPS molecular networking job (MSV000083437), with peak lists and precursor masses
- GNPS edge/network results table with node pair identifiers and cosine_score values
- GNPS job parameter settings (top-N peak filtering, precursor mass tolerance)

## Outputs

- cosine_score_comparison.tsv with columns: node pair, computed modified cosine score, GNPS cosine_score, and their difference

## How to apply

Parse the consensus spectra (.mgf) from the reproduced MSV000083437 GNPS molecular networking job to load peak lists and precursor masses, normalizing peaks to base peak intensity. Before scoring, apply GNPS peak filtering: remove peaks in a ±17 Da window around the precursor ion, keep only the top six most intense peaks within each ±50 Da window, and use the job's precursor mass tolerance. For each node pair in the edge table, compute a modified cosine score by greedily matching peaks at identical m/z and at m/z shifts equal to the precursor ion mass difference, weighting matched intensities by the square root of intensity. Join computed scores with the GNPS edge table on node pair identifiers and write cosine_score_comparison.tsv with node pair, computed score, GNPS cosine_score, and their difference.

## Related tools

- **GNPS** (Platform whose molecular networking workflow (METABOLOMICS-SNETS-V2) generates the .mgf consensus spectra, edge table, and reference cosine_score values being reconstructed) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw data files to open formats (.mzXML, .mzML, .mgf) before scoring)

## Examples

```
python compute_modified_cosine.py --mgf MSV000083437_consensus.mgf --edges clusterinfo_summary.tsv --topn 6 --window 50 --out cosine_score_comparison.tsv
```

## Evaluation signals

- Computed modified cosine scores for shared node pairs match the GNPS edge table cosine_score within a small tolerance (difference column near zero)
- Every node pair in the comparison table resolves via join on node pair identifiers with no unmatched edges
- Peak filtering reproduced: top 6 peaks per 50 Da window and ±17 Da precursor-window removal applied before scoring
- Matched-intensity weighting uses square root of intensity and scores are normalized by base peak intensity
- Shifted-peak matches occur at m/z offsets equal to the precursor ion mass difference, not at arbitrary shifts

## Limitations

- Classic molecular networking is qualitative (MS2 scans only); it cannot provide calibrated quantitation without follow-up targeted LC-MS
- MS2 spectra typically cannot differentiate regio- or stereoisomers, so high cosine edges do not imply unique structures
- Reproduction depends on matching the exact job filtering parameters (top 6 in 50 Da windows, precursor mass tolerance); deviation changes scores

## Evidence

- [intro] GNPS scores pairwise MS2 spectral similarity with a modified cosine score in which, in addition to matching fragment ions at identical m/z, the alignment also considers fragment ions shifted by the mass difference of the two precursor ions.: [evidence span withheld — anchor status: not_found]
- [intro] two structurally related molecules share fragment ion patterns when subjected to MS2 fragmentation methods: [evidence span withheld — anchor status: not_found]
