---
name: mass-spectral-intensity-binning
description: Use when raw GC-EI-MS peak lists (m/z / relative-intensity pairs, e.g.
  NIST 20 msp records) must be converted into a discrete intensity vocabulary before
  training or running a spectra-to-SMILES transformer, or when choosing among binning
  schemes.
when_to_use_negative:
- The data are MS/MS spectra rather than GC-EI-MS spectra — the binning scheme was
  tuned for single-precursor EI-MS peak lists
- A downstream model already consumes continuous intensity values (e.g. 4-decimal-place
  linear encoding) and you do not need the discrete-token trade-off
- Spectra have not yet passed the model-specific filters (max m/z 500, max 300 peaks,
  max SMILES length 100), so binned output would include records the model cannot
  process
tools:
- name: SpecTUS
  role: Repository whose notebooks/ directory contains the data-preprocessing notebooks
    implementing the logarithmic intensity binning (30 bins, log base 1.28)
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Notebook environment in which the preprocessing (intensity binning) step is
    run and documented
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:0a0d41b7f699d74a94671ad3dbcddb4ac7c65a90f6fef48cd7be62edef24a081
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - Python
  - SpecTUS
  - conda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-spectral-intensity-binning

## Summary

Discretizes continuous relative-intensity values of EI-MS peak lists into a fixed vocabulary of 30 logarithmically spaced bins (log base 1.28, shift 29) so spectra can be fed to a transformer (SpecTUS) as a discrete sequence alongside integer-rounded m/z tokens.

## When to use

Use when raw GC-EI-MS peak lists (m/z / relative-intensity pairs, e.g. NIST 20 msp records) must be converted into a discrete intensity vocabulary before training or running a spectra-to-SMILES transformer, or when choosing among binning schemes.

## When NOT to use

- The data are MS/MS spectra rather than GC-EI-MS spectra — the binning scheme was tuned for single-precursor EI-MS peak lists
- A downstream model already consumes continuous intensity values (e.g. 4-decimal-place linear encoding) and you do not need the discrete-token trade-off
- Spectra have not yet passed the model-specific filters (max m/z 500, max 300 peaks, max SMILES length 100), so binned output would include records the model cannot process

## Inputs

- Raw EI-MS spectra as m/z / relative-intensity peak lists with associated SMILES labels, loaded from NIST 20 records in msp format
- Filtered spectra records surviving the m/z ≤ 500, ≤300 peaks, and SMILES ≤100 character thresholds

## Outputs

- Spectra records with each relative intensity mapped to one of 30 discrete logarithmic bins (log base 1.28, shift 29)
- preprocessed_spectra.csv containing the filtered, binned spectra for model training

## How to apply

Load the raw EI-MS peak lists from NIST 20 records in msp format and first apply the model-specific filters: discard spectra with any m/z above 500, more than 300 peaks, or SMILES longer than 100 characters (about 3% of NIST records are excluded). Then encode relative intensities by logarithmic binning with log base 1.28 and a shift of 29, yielding 30 discrete intensity bins, and write the filtered, binned records to preprocessed_spectra.csv. This scheme was selected in the paper's intensity-binning experiment: logarithmic binning with 30 bins gave a 0.9% improvement in validation Acc 1 over 4-decimal-place linear binning while reducing trainable parameters by 10 million, whereas over-simplified options (linear binning with 2 decimal places, logarithmic binning with 10 bins) cost 3–4% Acc 1. The preprocessing notebooks in the SpecTUS repository document this step; after encoding, m/z values are rounded to the nearest integer to form the spectral token sequence.

## Related tools

- **SpecTUS** (Repository whose notebooks/ directory contains the data-preprocessing notebooks implementing the logarithmic intensity binning (30 bins, log base 1.28)) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Notebook environment in which the preprocessing (intensity binning) step is run and documented) — https://github.com/hejjack/SpecTUS

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git; cd SpecTUS; conda env create -f trainSpectus_build.yaml
```

## Evaluation signals

- Every retained spectrum is represented with exactly 30 possible intensity bins (no intensities fall outside the bin vocabulary)
- The fraction of NIST 20 records removed by the preceding thresholds (m/z > 500, >300 peaks, SMILES >100 chars) is approximately 3%
- A model trained with the 30-bin logarithmic encoding should beat a 4-decimal-place linear-binning baseline by about 0.9% validation Acc 1 with roughly 10 million fewer trainable parameters
- Avoid over-simplified schemes: linear binning with 2 decimal places or logarithmic binning with 10 bins causes a 3–4% drop in validation Acc 1 relative to the chosen encoding
- Binned output written to preprocessed_spectra.csv pairs each spectrum with a valid SMILES label for finetuning

## Limitations

- Binning granularity is a trade-off: coarser schemes (10 log bins, 2-decimal linear) measurably reduce validation Acc 1, and the 30-bin log scheme was chosen empirically for this model
- The final SpecTUS model and this preprocessing scheme target EI-MS spectra; comparison with de novo approaches trained on MS/MS spectra is not straightforward
- Model performance is sensitive to input spectra quality — binning noisy spectra (e.g. from Cayman or MONA-style sources) degrades results relative to curated NIST/SWGDRUG data
- Training with the resulting encodings on NIST 20 requires a license for that proprietary dataset

## Evidence
