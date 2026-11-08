---
name: cosine-score-hit-validation
description: Use when a candidate annotation for an MS1-only feature needs validation
  or propagation by matching its pseudo MS/MS spectrum against a pool of pseudo MS/MS
  spectra or neighbor nodes via modified cosine similarity and delta masses.
when_to_use_negative:
- The query has true acquired MS/MS spectra — use standard library spectral matching
  rather than pseudo MS/MS cross-matching
- Input is only a feature table or m/z list with no correlated in-source fragments
  available to build pseudo MS/MS spectra (pseudo spectra require >= 4 consecutive
  shared MS1 scans for peak-peak correlation)
- No reference or neighbor spectra exist to match against — cosine-based validation
  cannot produce a structural assignment de novo
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ms1-id
  role: Performs the flash-entropy-based cosine matching of pseudo MS/MS spectra against
    indexed MS/MS libraries and spectral pools for annotation and MASST-like searches
  repo: https://github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:5621d2e8cb8686cc63f50f3ae5aedb09184d1cf72edf9a6e1359ef3aebd02454
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - ms1-id
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cosine-score-hit-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates and propagates molecular annotations of pseudo MS/MS spectra (generated from MS1-only full-scan data) by cross-matching them via flash-entropy-based cosine / modified cosine similarity and delta masses against neighbor nodes or a spectral pool, as demonstrated in ms1-id's MASST-like proof-of-principle searches.

## When to use

Use when a candidate annotation for an MS1-only feature needs validation or propagation by matching its pseudo MS/MS spectrum against a pool of pseudo MS/MS spectra or neighbor nodes via modified cosine similarity and delta masses.

## When NOT to use

- The query has true acquired MS/MS spectra — use standard library spectral matching rather than pseudo MS/MS cross-matching
- Input is only a feature table or m/z list with no correlated in-source fragments available to build pseudo MS/MS spectra (pseudo spectra require >= 4 consecutive shared MS1 scans for peak-peak correlation)
- No reference or neighbor spectra exist to match against — cosine-based validation cannot produce a structural assignment de novo

## Inputs

- Query pseudo MS/MS spectrum (mgf format) deconvolved from MS1-only full-scan data (e.g., from NIST feces data MSV000095787)
- Pool of pre-generated pseudo MS/MS spectra from a target dataset (e.g., IBD dataset, Metabolomics Workbench PR000639)
- Indexed MS/MS spectral library (e.g., indexed GNPS library, gnps.pkl / gnps_k10.pkl built from mgf or msp)

## Outputs

- Ranked list of cosine-similarity matches with precursor masses and annotations (tsv per input file)
- 'masst_search_result.json' containing query spectrum metadata, top-hit precursor mass, cosine score, and annotation
- Mirror plots showing matched fragmentation patterns and delta-mass neutral losses (e.g., tyrosine pattern with fatty acyl chain loss)

## How to apply

Load the query pseudo MS/MS spectrum (e.g., phenylalanine-C3:0 generated from NIST feces full-scan data MSV000095787) and the target pool of pseudo MS/MS spectra (e.g., pre-generated from the IBD dataset, Metabolomics Workbench PR000639). Run a forward cosine similarity search of the query against every spectrum in the pool using the flash-entropy-based cosine matching in ms1-id, then rank matches and retain the top hit. Verify the hit against expected evidence: in the IBD proof-of-principle the top hit for phenylalanine-C3:0 attained a cosine score of 0.90, and propagated annotations (e.g., Tyr-C8:1) were confirmed by a mirror plot showing the tyrosine fragmentation pattern and neutral loss of the fatty acyl chain. When building molecular networks, clean each feature's pseudo MS/MS by removing ions larger than the feature m/z and require modified cosine >= 0.8 with >= 6 matched peaks for an edge. For general LC-MS annotation cutoffs, apply minimum score 0.7, minimum 3 matched peaks, and minimum spectral usage 0.20 (0.05 for MS imaging). Write the query metadata, top-hit precursor mass, cosine score, and annotation to 'masst_search_result.json'.

## Related tools

- **ms1-id** (Performs the flash-entropy-based cosine matching of pseudo MS/MS spectra against indexed MS/MS libraries and spectral pools for annotation and MASST-like searches) — https://github.com/Philipbear/ms1_id

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- The top IBD-pool hit must match phenylalanine-C3:0 with cosine score 0.90
- Molecular network edges require modified cosine >= 0.8 and >= 6 matched peaks
- General LC-MS annotations pass minimum score 0.7, minimum 3 matched peaks, and minimum spectral usage 0.20
- The mirror plot of a validated propagated annotation (e.g., Tyr-C8:1) shows the expected fragment pattern and the delta-mass neutral loss of the fatty acyl chain
- 'masst_search_result.json' contains the query metadata, top-hit precursor mass, cosine score, and annotation

## Limitations

- In-source fragments are low-energy while most public reference spectra are collected at medium to high collision energies, so peak intensity scaling is needed to align pseudo MS/MS with reference spectra
- Pseudo MS/MS quality depends on correlation support: at least 4 consecutive shared MS1 scans (LC-MS) or 50 shared non-zero pixels (imaging) are required, and isotope peaks determined by MassCube are excluded
- For MS imaging, fragments with m/z < 100 are typically absent and a looser spectral usage cutoff of 0.05 applies

## Evidence

- [discussion] A minimum of modified cosine of 0.8 and matched peaks of 6 are required to build an edge in the network construction: "A minimum of modified cosine of 0.8 and matched peaks of 6 are required to build an edge in the network construction"
- [discussion] LC-MS annotation cutoffs: "minimum score, 0.7; minimum matched peaks, 3; minimum spectral usage, 0.20"
- [discussion] Cleaning pseudo spectra for modified cosine: "we cleaned each metabolic feature's pseudo MS/MS by removing all ions larger than the feature m/z to enable modified cosine calculations"
- [readme] README on annotate command and output location: [evidence span withheld — anchor status: external_doc]
