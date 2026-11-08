---
name: static-formula-library-construction
description: Use when building or benchmarking a C2MS model that predicts spectra
  from a fixed formula vocabulary, or when quantifying how the breadth of a model's
  formula support (e.g. GrAFF-MS ~10,000 formulae vs FraGNNet's learned fragment distribution)
  affects formula annotation on held-out NIST20 spectra.
when_to_use_negative:
- The model already parameterizes its formula distribution internally (e.g. FraGNNet
  learns a distribution over molecule fragments, from which the formula distribution
  is derived) — a fixed vocabulary is unnecessary and would misrepresent the model.
- The spectra of interest are glycans or peptides, which the article explicitly excluded
  from its formula annotation analysis focused on small molecules.
- You need candidate formulae per individual unknown spectrum at retrieval time rather
  than one dataset-wide library.
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: GrAFF-MS
  role: Formula-predictor baseline whose support is defined by the static top-10,000
    formula library this skill constructs
- name: FraGNNet
  role: Contrast model whose formula support is derived from the approximate fragmentation
    DAG rather than the static library
  repo: github.com/FraGNNet/fragnnet
- name: RDKit
  role: Chemical-validity filtering of candidate formulae during library construction
- name: Python
  role: Implementation language for counting support sizes and computing summary statistics
    over the test set
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:9514ba88a1eb987e870f920e7b386e60ba4573904f7903661f81d93c2ac93d88
  source_papers:
  - doi: 10.48550/arxiv.2404.02360
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FraGNNet
  - GrAFF-MS
  - Python
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# static-formula-library-construction

## Summary

Constructs a fixed, dataset-wide formula candidate vocabulary for compound-to-mass-spectrum (C2MS) models by collecting product formulae and neutral-loss complements from NIST20 expert-curated annotations, keeping the top 10,000 formulae by total intensity, as used by GrAFF-MS as a contrast to FraGNNet's learned fragment-induced formula distribution.

## When to use

Use when building or benchmarking a C2MS model that predicts spectra from a fixed formula vocabulary, or when quantifying how the breadth of a model's formula support (e.g. GrAFF-MS ~10,000 formulae vs FraGNNet's learned fragment distribution) affects formula annotation on held-out NIST20 spectra.

## When NOT to use

- The model already parameterizes its formula distribution internally (e.g. FraGNNet learns a distribution over molecule fragments, from which the formula distribution is derived) — a fixed vocabulary is unnecessary and would misrepresent the model.
- The spectra of interest are glycans or peptides, which the article explicitly excluded from its formula annotation analysis focused on small molecules.
- You need candidate formulae per individual unknown spectrum at retrieval time rather than one dataset-wide library.

## Inputs

- NIST20 MS/MS dataset with expert-curated formula annotations (spectra exported from the NIST20 MS/MS spectral library)
- Precursor formulae per compound (for computing neutral-loss complements)
- Peak-to-formula annotation mass offsets (ppm, for the 10 ppm exclusion filter)

## Outputs

- Static formula vocabulary of the top 10,000 product and neutral loss formulae ranked by total intensity
- Per-molecule formula support sets with nonzero-probability counts (support sizes)
- Summary statistics table (median, quartiles, support-size distribution) e.g. formula_support_stats.csv

## How to apply

Start from the NIST20 expert-curated formula annotations, excluding any annotation whose formula mass is more than 10 ppm away from its associated peak mass and removing glycan and peptide spectra so the library covers small molecules only. Enumerate product formulae from the static library and, for each neutral loss formula, compute its complement against the precursor formula; pool both sets. Rank all pooled formulae by total intensity across spectra in the dataset and keep the top 10,000 product and neutral loss formulae as the static support, applying chemical-validity filtering (e.g. RDKit). For contrast with a learned-support model like FraGNNet, derive P_theta(f) per molecule from the approximate fragmentation DAG (depth d in {3,4}, hydrogen tolerance j=4) instead, then compare per-molecule nonzero-probability support sizes (median, quartiles) across the held-out NIST20 test set.

## Related tools

- **GrAFF-MS** (Formula-predictor baseline whose support is defined by the static top-10,000 formula library this skill constructs)
- **FraGNNet** (Contrast model whose formula support is derived from the approximate fragmentation DAG rather than the static library) — github.com/FraGNNet/fragnnet
- **RDKit** (Chemical-validity filtering of candidate formulae during library construction)
- **Python** (Implementation language for counting support sizes and computing summary statistics over the test set)

## Examples

```
stats = pd.DataFrame({'fragnnet': fragnnet_support_sizes, 'graffms': graffms_support_sizes}).describe(); stats.to_csv('formula_support_stats.csv')
```

## Evaluation signals

- Vocabulary size equals the top 10,000 product and neutral loss formulae by total intensity across spectra in the dataset
- No retained annotation deviates more than 10 ppm from its associated peak mass
- GrAFF-MS median support size is approximately 10,000 formulae per compound, as reported in the article (Section 5.3 / Table 4)
- FraGNNet contrast median support size is approximately 679 formulae per compound on the same held-out NIST20 test set
- Glycan and peptide spectra are absent from the constructed library

## Limitations

- A static vocabulary caps the candidate formula space per compound at ~10,000 regardless of molecular diversity, unlike fragment-induced supports (FraGNNet median ~679) that scale with the molecule.
- The article's fragmentation algorithm fails to account for some fragments (unexplained Outside-of-Support peaks), which may arise from complex chemistry such as cyclizations that expand the fragment search space.
- Performance was only evaluated on merged spectra under a limited range of experimental conditions (Orbitrap HCD, [M+H]+ adduct); other instrument platforms and precursor adducts are untested.

## Evidence

- [methods] Fragmentation DAG parameters for learned-support comparison: [evidence span withheld — anchor status: not_found]
