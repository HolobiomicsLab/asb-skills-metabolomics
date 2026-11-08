---
name: modified-cosine-fragment-ion-matching
description: 'Use when validating or re-implementing GNPS molecular networking edge
  scores: you have consensus .mgf spectra, precursor masses, and a GNPS edge table
  and must reproduce cosine_score per node pair.'
when_to_use_negative:
- The goal is library spectral matching / dereplication of a single spectrum against
  reference libraries, not pairwise consensus-vs-consensus edge scoring
- The input is already a feature abundance table (quantitative data) — classic molecular
  networking with modified cosine provides only qualitative information from MS2 scans,
  not calibrated quantitation
- Spectra were not pre-clustered into consensus spectra / nodes (e.g., raw per-scan
  MS2 without GNPS consensus clustering), since the score is defined on GNPS-style
  consensus spectra
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Source of the molecular networking workflow whose modified cosine edge scores
    are being reconstructed and compared against
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSConvert (ProteoWizard)
  role: Used upstream to convert instrument raw data to open formats (.mzXML, .mzML,
    .mgf) feeding the consensus spectra used in the alignment
  repo: http://proteowizard.sourceforge.net/downloads.shtml
- name: GNPS_Documentation
  role: Reference for the GNPS modified cosine scoring and networking parameter definitions
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:7a62e7b2dd045684002adf66e6c8bd4f16e8d496321df95d5e7a6495f06a1308
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - GNPS_Documentation
  - MSConvert (ProteoWizard)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# modified-cosine-fragment-ion-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs GNPS's modified cosine pairwise MS2 spectral alignment, in which fragment ions are matched both at identical m/z and at m/z shifts equal to the precursor ion mass difference, to independently verify the cosine_score edges of a GNPS molecular networking job.

## When to use

Use when validating or re-implementing GNPS molecular networking edge scores: you have consensus .mgf spectra, precursor masses, and a GNPS edge table and must reproduce cosine_score per node pair.

## When NOT to use

- The goal is library spectral matching / dereplication of a single spectrum against reference libraries, not pairwise consensus-vs-consensus edge scoring
- The input is already a feature abundance table (quantitative data) — classic molecular networking with modified cosine provides only qualitative information from MS2 scans, not calibrated quantitation
- Spectra were not pre-clustered into consensus spectra / nodes (e.g., raw per-scan MS2 without GNPS consensus clustering), since the score is defined on GNPS-style consensus spectra

## Inputs

- Consensus spectra file (.mgf) with peak lists and precursor masses from a GNPS networking job
- GNPS edge/network results table with node pair identifiers and cosine_score column
- GNPS job parameter settings (top-N peak window filtering, precursor mass tolerance, e.g. top 6 peaks in 50 Da windows)

## Outputs

- cosine_score_comparison.tsv with columns: node pair, computed modified cosine score, GNPS cosine_score, and their difference

## How to apply

Download the consensus spectra (.mgf) and edge results table from the GNPS molecular networking job (e.g., the MSV000083437 GF/SPF mouse networking job). Parse the .mgf to load peak lists and precursor masses, normalizing peaks to base peak intensity. Before scoring, apply GNPS peak filtering: delete peaks in a ±17 Da window around the precursor, and keep only the top six most intense peaks within each ±50 Da window, plus the job's precursor mass tolerance. For each node pair in the edge table, compute the modified cosine by matching peaks at identical m/z AND at m/z shifts equal to the difference between the two precursor masses, weighting matched intensity products by the square root of intensity, and normalizing by the base peak; this shifted matching reflects that structurally related molecules share fragment ion patterns offset by their mass difference. Join computed scores with the GNPS edge table on node pair identifiers, writing cosine_score_comparison.tsv with computed score, GNPS cosine_score, and their difference per edge.

## Related tools

- **GNPS** (Source of the molecular networking workflow whose modified cosine edge scores are being reconstructed and compared against) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MSConvert (ProteoWizard)** (Used upstream to convert instrument raw data to open formats (.mzXML, .mzML, .mgf) feeding the consensus spectra used in the alignment) — http://proteowizard.sourceforge.net/downloads.shtml
- **GNPS_Documentation** (Reference for the GNPS modified cosine scoring and networking parameter definitions) — https://github.com/CCMS-UCSD/GNPSDocumentation

## Examples

```
python recompute_modified_cosine.py --mgf consensus.mgf --edges GNPS_edges.tsv --top-n 6 --window 50 --out cosine_score_comparison.tsv
```

## Evaluation signals

- Computed modified cosine scores joined per node pair should closely match the GNPS edge table cosine_score column (small differences recorded in cosine_score_comparison.tsv)
- Peak filtering must reproduce GNPS settings: only the top six most intense peaks per ±50 Da window retained, and ±17 Da precursor region removed
- Matched peak sets must include both identical-m/z matches and shifted matches at the precursor mass difference; removing shifted matching should lower scores on edges between analogs with different precursor masses
- All scores should fall in [0, 1] after intensity square-root weighting and base-peak normalization
- The comparison table must contain every shared node pair from the GNPS edge table (row count parity after the join)

## Limitations

- Classic molecular networking with this score is qualitative only (MS2 scans); it cannot provide calibrated quantitation without follow-up targeted LC-MS
- MS2 spectra typically cannot differentiate regio- or stereoisomers, so high modified cosine does not imply identical structure
- Results depend on exact replication of the job's filter settings (top-N window, precursor tolerance); misconfigured filtering will shift computed scores away from GNPS values
- Only MS2 spectra reaching the ~5–6% annotation rate in untargeted metabolomics can be anchored to known molecules; most edges remain among unannotated 'dark matter' nodes

## Evidence

- [intro] modified cosine score considers fragment ions shifted by the precursor mass difference: [evidence span withheld — anchor status: not_found]
- [intro] cosine 0.7 with 6 matching MS2 ions default: [evidence span withheld — anchor status: not_found]
- [intro] structural relatedness underlies fragment matching: [evidence span withheld — anchor status: not_found]
- [intro] MS2-only qualitative limitation: [evidence span withheld — anchor status: not_found]
