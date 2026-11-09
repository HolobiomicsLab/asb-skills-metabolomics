---
name: spectral-library-deduplication
description: Use when an MGF-format MS/MS library must be audited for redundant spectra
  prior to model training or library release — i.e., you need to count spectra that
  are near-identical (cosine >= 0.99) to another spectrum of the same compound.
when_to_use_negative:
- When the goal is removing noisy or low-quality spectra (zero/uniform intensity fragments,
  low-resolution spectra) rather than redundancy — use the library_cleaning_1/2.yaml
  matchms filter pipeline instead.
- When similarity between different compounds is of interest — this skill only compares
  spectra sharing the first 14 InChIKey characters and ion polarity.
- When chemical/electronic noise removal is needed — spectral_denoising handles that,
  not near-duplicate detection.
edam_operation: http://edamontology.org/operation_3695
edam_topics: []
tools:
- name: matchms
  role: Parses and loads the MGF spectral library and provides the spectral comparison
    framework for the cosine score
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Provides the cosine similarity score implementation used for pairwise same-compound
    spectrum comparison
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: Python
  role: Language in which the grouping, pairwise scoring, and near-duplicate flagging
    workflow is implemented
- name: spectraverse-analysis
  role: Repository containing the preprocessing code, including near-duplicate removal,
    used to build Spectraverse
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:69ce7c32708090fefe382a594e9da4661c99d2b8abdb96c126b171a4940239ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - SpectralEntropy
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-library-deduplication

## Summary

Quantifies and flags near-duplicate MS/MS spectra within a spectral library (e.g., MassSpecGym) by grouping spectra from the same compound and flagging pairs with cosine similarity >= 0.99. The paper used this to identify 77,774 near-duplicate spectra in MassSpecGym, motivating their removal before machine-learning benchmarking.

## When to use

Use when an MGF-format MS/MS library must be audited for redundant spectra prior to model training or library release — i.e., you need to count spectra that are near-identical (cosine >= 0.99) to another spectrum of the same compound.

## When NOT to use

- When the goal is removing noisy or low-quality spectra (zero/uniform intensity fragments, low-resolution spectra) rather than redundancy — use the library_cleaning_1/2.yaml matchms filter pipeline instead.
- When similarity between different compounds is of interest — this skill only compares spectra sharing the first 14 InChIKey characters and ion polarity.
- When chemical/electronic noise removal is needed — spectral_denoising handles that, not near-duplicate detection.

## Inputs

- MassSpecGym.mgf (MGF-format MS/MS spectral library file placed in the data/ directory)
- Spectrum metadata: InChIKey (for connectivity-layer grouping) and ion polarity

## Outputs

- near_duplicate_clusters.csv — table of near-duplicate spectra with pair/group identifiers
- Total count of near-duplicate spectra (expected 77,774 for MassSpecGym)

## How to apply

Load the library MGF file (placed in data/, e.g. MassSpecGym.mgf) and parse spectra with matchms. Group spectra by the first 14 characters of their InChIKey (the connectivity layer) together with ion polarity, so that only same-compound, same-polarity spectra are compared. Within each group, compute pairwise cosine similarity between spectra using the SpectralEntropy cosine score implementation. Flag every spectrum whose cosine similarity to another same-compound spectrum is >= 0.99 as a near-duplicate, retaining pair/group identifiers. Write the near-duplicate cluster table to near_duplicate_clusters.csv and report the total near-duplicate count (77,774 for MassSpecGym). The rationale is that near-duplicates inflate apparent library size and can confound machine-learning model comparisons, which is why the Spectraverse authors removed them (yielding more than three times as many spectra beyond MassSpecGym).

## Related tools

- **matchms** (Parses and loads the MGF spectral library and provides the spectral comparison framework for the cosine score) — https://github.com/matchms/matchms
- **SpectralEntropy** (Provides the cosine similarity score implementation used for pairwise same-compound spectrum comparison) — https://github.com/YuanyueLi/SpectralEntropy
- **Python** (Language in which the grouping, pairwise scoring, and near-duplicate flagging workflow is implemented)
- **spectraverse-analysis** (Repository containing the preprocessing code, including near-duplicate removal, used to build Spectraverse) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
from matchms.importing import load_from_mgf; spectra = list(load_from_mgf('data/MassSpecGym.mgf')); # then group by InChIKey[:14] + polarity and score pairs with SpectralEntropy cosine
```

## Evaluation signals

- near_duplicate_clusters.csv exists in CSV format and contains pair/group identifiers for flagged spectra.
- The reported near-duplicate count equals the paper's figure of 77,774 spectra for MassSpecGym.
- Every flagged spectrum has a same-compound partner (matching first 14 InChIKey characters and ion polarity) with cosine similarity >= 0.99.
- Spot-check a sample of flagged pairs by recomputing SpectralEntropy cosine scores and confirming values >= 0.99.
- No cross-compound pairs appear in the near-duplicate table, confirming grouping was restricted to the connectivity InChIKey layer.

## Limitations

- Near-duplicate spectra removal was only reported for MassSpecGym comparisons; the completeness of de-duplication for Spectraverse itself is not stated.
- The broader curation pipeline relied on manual review of only several thousand spectra, so pitfalls may remain beyond what was manually reviewed.
- Only near-identical spectra (cosine >= 0.99) are captured; partially similar but non-redundant spectra of the same compound are not flagged.

## Evidence

- [other] 77,774 spectra are near-duplicates, defined as spectra with a cosine similarity of 0.99 or greater to another spectrum of the same compound in the data set.: [evidence span withheld — anchor status: not_found]
- [other] Group spectra by the first 14 characters of their InChIKey (connectivity layer) together with their ion polarity to define candidate same-compound sets.: [evidence span withheld — anchor status: not_found]
- [discussion] Removal of near-duplicate spectra from datasets: [evidence span withheld — anchor status: not_found]
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: [evidence span withheld — anchor status: external_doc]
- [methods] Prepare your data: Place raw data files (MGF format) in the `data/` directory.: "Place raw data files (MGF format) in the `data/` directory."
