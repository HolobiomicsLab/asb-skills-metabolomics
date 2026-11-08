---
name: near-duplicate-spectra-detection
description: Use when curating or benchmarking an MS/MS spectral library (e.g. an
  MGF file) and you need to quantify or remove spectra that are near-duplicates of
  another spectrum of the same compound, as a data-quality or train/test-leakage check.
when_to_use_negative:
- Input spectra lack InChIKey annotations or ion-polarity metadata, so same-compound
  groups cannot be formed.
- You need de-duplication for Spectraverse itself; the article only reports near-duplicate
  removal counts in comparison to MassSpecGym and does not state completeness of de-duplication
  for Spectraverse.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Parsing/loading MGF spectra and preprocessing for pairwise comparison
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Computing cosine similarity scores between same-compound spectra
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:5861d0b61adea86fb16346ec4c5e951037ee14b77e481cb92e92bb1c1b77ec47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - SpectralEntropy
  - matchms
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# near-duplicate-spectra-detection

## Summary

Detects near-duplicate MS/MS spectra by grouping spectra from the same compound (first 14 InChIKey characters plus ion polarity) and flagging pairs with cosine similarity ≥ 0.99 using the SpectralEntropy cosine implementation, as used to quantify the 77,774 near-duplicate spectra in MassSpecGym.

## When to use

Use when curating or benchmarking an MS/MS spectral library (e.g. an MGF file) and you need to quantify or remove spectra that are near-duplicates of another spectrum of the same compound, as a data-quality or train/test-leakage check.

## When NOT to use

- Input spectra lack InChIKey annotations or ion-polarity metadata, so same-compound groups cannot be formed.
- You need de-duplication for Spectraverse itself; the article only reports near-duplicate removal counts in comparison to MassSpecGym and does not state completeness of de-duplication for Spectraverse.

## Inputs

- MGF-format MS/MS library file placed in the data/ directory (e.g. MassSpecGym.mgf)
- Per-spectrum metadata: InChIKey and ion polarity

## Outputs

- near_duplicate_clusters.csv — table of flagged near-duplicate spectra with pair/group identifiers
- Total count of near-duplicate spectra (77,774 for MassSpecGym)

## How to apply

Load the MGF library (e.g. MassSpecGym.mgf) and parse spectra with matchms. Group spectra by the first 14 characters of their InChIKey (the connectivity layer) together with their ion polarity to define candidate same-compound sets. Within each group compute pairwise cosine similarity between spectra using the SpectralEntropy cosine score implementation. Flag every spectrum whose cosine similarity to another spectrum of the same compound is ≥ 0.99 as a near-duplicate, then write the near-duplicate cluster table (near_duplicate_clusters.csv) and report the total count — for MassSpecGym this equals 77,774 spectra. The rationale is that near-duplicates can confound machine-learning model comparisons or cause valid spectra to be discarded, and removing them yields 'more than three times as many' spectra relative to Spectraverse comparisons.

## Related tools

- **matchms** (Parsing/loading MGF spectra and preprocessing for pairwise comparison) — https://github.com/matchms/matchms
- **SpectralEntropy** (Computing cosine similarity scores between same-compound spectra) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
python run.py --input data/MassSpecGym.mgf --output near_duplicate_clusters.csv
```

## Evaluation signals

- near_duplicate_clusters.csv exists in CSV format in the output directory.
- Reported near-duplicate count equals the paper's figure of 77,774 spectra for MassSpecGym.
- Every flagged spectrum has cosine similarity ≥ 0.99 to at least one other spectrum in the same compound group (first-14-InChIKey + polarity).
- No spectrum is flagged as a near-duplicate without a same-compound partner spectrum.

## Limitations

- Curation relied on manual review of only several thousand spectra, so pitfalls may remain beyond what was manually reviewed.
- Near-duplicate spectra removal was only reported for MassSpecGym comparisons; completeness of de-duplication for Spectraverse itself is not stated.
- InChIKey connectivity-layer grouping assumes correctly annotated structures; missing or wrong InChIKeys would break group formation.

## Evidence

- [supplementary] near-duplicate workflow trace: [evidence span withheld — anchor status: not_found]
- [discussion] Removal of near-duplicate spectra: "(and more than three times as many after removing near-duplicate spectra from this resource)"
- [methods] pipeline for near-duplicate clusters: "Place raw data files (MGF format) in the `data/` directory."
- [readme] SpectralEntropy tool description: [evidence span withheld — anchor status: not_found]
