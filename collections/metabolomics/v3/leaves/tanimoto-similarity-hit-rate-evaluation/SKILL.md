---
name: tanimoto-similarity-hit-rate-evaluation
description: Use when a ranked-candidate metabolite-identification run (e.g. ChemEmbed
  or similar embedding-based spectral matching) has produced candidate lists per spectrum
  and you must quantify ranking performance as top-1 (Tanimoto ≥ 0.95) and top-5 hit
  rates.
when_to_use_negative:
- When the query spectra have no annotated SMILES and only structural-class or spectral
  similarity (cosine) evaluation is intended — Tanimoto requires molecular structures
  for both query and candidate.
- 'When evaluating against a differently sized reference database: results are not
  directly comparable to the reported >42% top-1 / >76% top-5 rates because candidates
  outside the 520,083-compound ChemEmbed reference cannot be identified regardless
  of spectral quality.'
- When the analysis goal is de novo structure elucidation rather than ranking an existing
  candidate list — the metric assumes a ranked list of candidate molecules is available.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: RDKit
  role: Compute exact Tanimoto similarity between query and candidate SMILES strings
    per spectrum
  repo: https://github.com/rdkit/rdkit
- name: pandas
  role: Load prediction_results.csv, group per spectrum, aggregate boolean top-1/top-5
    flags into hit-rate fractions
- name: Python 3.6+
  role: Runtime for the evaluation script/notebook (ChemEmbed_Sample_Test.ipynb)
  repo: https://github.com/faizanurv/ChemEmbed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:37e3d29370759b546913fc9f80109e8c12d73831cb9d0eb08249ab3a6ca7d2c8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/massspecdl/ChemEmbed
  repo_url: https://github.com/rdkit/rdkit
  tools:
  - ChemEmbed
  - Python 3.6+
  - Python+
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tanimoto-similarity-hit-rate-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates metabolite-identification ranking output by computing RDKit Tanimoto similarity between query and candidate SMILES, then reporting the fraction of spectra whose top-1 candidate reaches Tanimoto ≥ 0.95 and whose correct compound appears in the top five.

## When to use

Use when a ranked-candidate metabolite-identification run (e.g. ChemEmbed or similar embedding-based spectral matching) has produced candidate lists per spectrum and you must quantify ranking performance as top-1 (Tanimoto ≥ 0.95) and top-5 hit rates.

## When NOT to use

- When the query spectra have no annotated SMILES and only structural-class or spectral similarity (cosine) evaluation is intended — Tanimoto requires molecular structures for both query and candidate.
- When evaluating against a differently sized reference database: results are not directly comparable to the reported >42% top-1 / >76% top-5 rates because candidates outside the 520,083-compound ChemEmbed reference cannot be identified regardless of spectral quality.
- When the analysis goal is de novo structure elucidation rather than ranking an existing candidate list — the metric assumes a ranked list of candidate molecules is available.

## Inputs

- prediction_results.csv or equivalent ranked candidate list per spectrum (query SMILES, candidate SMILES, rank)
- Test-split MSP spectra with annotated SMILES (e.g. ChemEmbed held-out test set)
- Reference database of candidate SMILES and precursor masses (e.g. chemembed_reference_mol2vec_300d_520k.parquet, 520,083 compounds)

## Outputs

- Per-spectrum boolean top-1 hit (Tanimoto ≥ 0.95) and top-5 hit flags
- Aggregate top-1 and top-5 hit-rate fractions across the test split
- Tanimoto similarity values between query SMILES and each candidate SMILES

## How to apply

For each spectrum in the results file (one row per input spectrum, e.g. prediction_results.csv), take the query SMILES and each ranked candidate SMILES and compute exact Tanimoto similarity with RDKit. Count a spectrum as a top-1 hit when the highest-ranked candidate has Tanimoto ≥ 0.95 (structural equivalence threshold used by ChemEmbed), and as a top-5 hit when any of the top-5 candidates (top_n_candidates=5 default) meets that threshold. Restrict the candidate set to compounds whose precursor mass falls within the precursor tolerance (e.g. --precursor_tolerance_ppm 5 for Orbitrap, 10–20 for QTOF) before ranking, since this filter drives recall. Aggregate the per-spectrum boolean outcomes into overall fractions and validate against the article's reported test-set values: >42% top-1 correct (Tanimoto ≥ 0.95) and >76% top-5 annotations. Spectra with no reference compound inside the mass window are scored as misses (NA candidates), so the hit rate denominators remain all input spectra.

## Related tools

- **RDKit** (Compute exact Tanimoto similarity between query and candidate SMILES strings per spectrum) — https://github.com/rdkit/rdkit
- **pandas** (Load prediction_results.csv, group per spectrum, aggregate boolean top-1/top-5 flags into hit-rate fractions)
- **Python 3.6+** (Runtime for the evaluation script/notebook (ChemEmbed_Sample_Test.ipynb)) — https://github.com/faizanurv/ChemEmbed

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive test_spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- Top-1 correct-candidate fraction (Tanimoto ≥ 0.95) should exceed 42% on the ChemEmbed test dataset
- Top-5 annotation fraction should exceed 76% on the ChemEmbed test dataset
- Results file contains exactly one row per input spectrum; unmatched spectra carry NA candidates and count as misses, so the denominator equals the number of input spectra
- All Tanimoto values lie in [0, 1]; a value ≥ 0.95 indicates structural equivalence by the article's criterion
- Per-spectrum candidate lists contain at most top_n_candidates (default 5) entries

## Limitations

- Candidate matching can only return molecules present in the reference database; a compound outside the 520,083-compound ChemEmbed reference cannot be identified regardless of spectral quality, so hit rates are reference-database-dependent.
- Spectra with no reference compound inside the precursor mass window are reported with NA candidates rather than omitted; they are treated as misses and lower the aggregate hit rates.
- A precursor tolerance of 0 (legacy three-decimal bucket) should not be used for new evaluation because reference masses were rounded while query masses were truncated, causing the filter to fail on roughly half of compounds; use 5 ppm (Orbitrap) or 10–20 ppm (QTOF) instead.
- Tanimoto ≥ 0.95 is a structural-equivalence proxy, not exact-identity matching; very similar isomers may both pass while only one is the true annotation.

## Evidence
