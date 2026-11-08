---
name: spectral-peak-threshold-filtering
description: Use when preparing raw EI-MS peak lists (e.g., NIST 20 msp records) for
  a transformer model that expects bounded m/z range, bounded peak count, and bounded
  SMILES label length prior to intensity binning.
when_to_use_negative:
- Input spectra are MS/MS tandem spectra or need precursor ion mass/molecular formula
  as input cues — the SpecTUS thresholds were tuned for GC-EI-MS peak lists only
- The downstream model accepts unbounded m/z or peak counts — these cutoffs are SpecTUS-specific
  architecture constraints, not general mass-spectrometry QC rules
- Spectra are already binned/encoded intensity vectors rather than raw peak lists
  — filtering thresholds apply to raw m/z and peak-count fields
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: SpecTUS
  role: Reference implementation whose model-specific thresholds (m/z ≤ 500, ≤ 300
    peaks, SMILES ≤ 100 chars) define the filter; preprocessing notebooks in the repo
    implement this step
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Data preprocessing is documented and run step by step in the repository's
    notebooks/ directory
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:215a93a2aec011b1bb098e81af7a3967ea4a702d55f7433f12baab5905ee3270
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  - conda
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spectral-peak-threshold-filtering

## Summary

Applies SpecTUS's model-specific filtering thresholds to raw EI-MS spectra before preprocessing: m/z values ≤ 500, ≤ 300 peaks per spectrum, and SMILES labels ≤ 100 characters, which together exclude ~3% of the NIST dataset records.

## When to use

Use when preparing raw EI-MS peak lists (e.g., NIST 20 msp records) for a transformer model that expects bounded m/z range, bounded peak count, and bounded SMILES label length prior to intensity binning.

## When NOT to use

- Input spectra are MS/MS tandem spectra or need precursor ion mass/molecular formula as input cues — the SpecTUS thresholds were tuned for GC-EI-MS peak lists only
- The downstream model accepts unbounded m/z or peak counts — these cutoffs are SpecTUS-specific architecture constraints, not general mass-spectrometry QC rules
- Spectra are already binned/encoded intensity vectors rather than raw peak lists — filtering thresholds apply to raw m/z and peak-count fields

## Inputs

- Raw EI-MS spectra records in msp format from NIST 20 (m/z / relative-intensity peak lists)
- Associated SMILES structure labels for each spectrum record

## Outputs

- Filtered spectrum records satisfying max m/z 500, max 300 peaks, max SMILES length 100 characters
- Preprocessed spectra file (e.g., preprocessed_spectra.csv) ready for logarithmic intensity binning

## How to apply

Load the raw EI-MS spectra (m/z / relative-intensity peak lists with associated SMILES labels) from the NIST 20 records in msp format, then discard any record violating the three model-specific thresholds: any m/z value above 500, more than 300 peaks per spectrum, or a SMILES string longer than 100 characters. Apply the filters before intensity encoding, since downstream logarithmic binning (log base 1.28, shift 29, 30 bins) assumes spectra within these bounds. The rationale is that the transformer's input vocabulary and sequence length are sized to these maxima, so out-of-range spectra cannot be encoded. The paper reports that applying these thresholds excluded approximately 3% of the NIST dataset, so a drastically higher removal fraction signals a bug in the parsing or threshold logic. Write the surviving records forward as the input to the preprocessing/encoding step (e.g., a preprocessed spectra CSV).

## Related tools

- **SpecTUS** (Reference implementation whose model-specific thresholds (m/z ≤ 500, ≤ 300 peaks, SMILES ≤ 100 chars) define the filter; preprocessing notebooks in the repo implement this step) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Data preprocessing is documented and run step by step in the repository's notebooks/ directory) — https://github.com/hejjack/SpecTUS

## Evaluation signals

- Fraction of NIST records removed by the thresholds should be approximately 3%, matching the paper's reported exclusion rate
- Every retained spectrum must contain no m/z value above 500
- Every retained spectrum must contain at most 300 peaks
- Every retained SMILES label must be at most 100 characters long
- Each retained spectrum must be encodable into exactly 30 possible intensity bins in the subsequent binning step

## Limitations

- The thresholds are model-specific: applying them to other models' test sets (e.g., MassGenie's CASMI 2017 set filtered from 243 to 93 spectra) can remove a far larger fraction of records
- Filtered-out compounds (~3% of NIST) are permanently excluded from training and evaluation, which may bias coverage toward compounds within the architecture's input bounds
- The NIST 20 dataset is proprietary; reproducing the filtering requires a license for NIST20

## Evidence
