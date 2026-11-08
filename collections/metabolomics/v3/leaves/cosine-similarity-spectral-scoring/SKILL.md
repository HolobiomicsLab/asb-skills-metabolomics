---
name: cosine-similarity-spectral-scoring
description: Use when experimental MGF-format MS/MS spectra must be annotated by reference-library
  search against libraries such as Spectraverse, MassSpecGym, or NPLIB1, or when pairwise
  spectral similarity matrices are needed to evaluate preprocessing variants (centroiding,
  denoising, top-n ion retention).
when_to_use_negative:
- Spectra are already normalized feature tables or fingerprint embeddings rather than
  peak-list MS/MS spectra — use a fingerprint- or ML-based similarity instead
- The goal is de novo structural prediction or ML model training rather than library-search
  annotation or similarity-based discrimination
- Library comparison requires metrics other than cosine (e.g., spectral entropy) —
  SpectralEntropy's entropy-based similarity was developed as the alternative the
  Spectraverse authors also use
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: matchms
  role: Parses MGF spectra into spectrum objects, applies precursor m/z tolerance
    (10 ppm) and matched-peak minimum (3), and computes cosine similarity scores for
    library search and pairwise matrices
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Source of the cosine score calculation used in the spectraverse-analysis pipeline;
    cloned and moved into the spectraverse package
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: MSEntropy
  role: Provides the MSEntropy-derived cosine function used for custom centroided-variant
    scoring in the AUROC comparison
- name: spectral_denoising
  role: Generates the electronically denoised spectrum variant whose cosine-scored
    AUROC is compared against raw spectra
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:2a86091b5adf903f97ca3d662449bcb9b698a4b95828619c10fea653f30629aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MSEntropy
  - Python
  - SpectralEntropy
  - matchms
  - spectral_denoising
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cosine-similarity-spectral-scoring

## Summary

Compute cosine similarity scores between experimental MS/MS spectra and reference library spectra to annotate metabolites, using matchms with a 10 ppm precursor m/z tolerance and a minimum of 3 matched fragment ions, and reuse the same scoring to build pairwise similarity matrices for AUROC benchmarking of preprocessing variants.

## When to use

Use when experimental MGF-format MS/MS spectra must be annotated by reference-library search against libraries such as Spectraverse, MassSpecGym, or NPLIB1, or when pairwise spectral similarity matrices are needed to evaluate preprocessing variants (centroiding, denoising, top-n ion retention).

## When NOT to use

- Spectra are already normalized feature tables or fingerprint embeddings rather than peak-list MS/MS spectra — use a fingerprint- or ML-based similarity instead
- The goal is de novo structural prediction or ML model training rather than library-search annotation or similarity-based discrimination
- Library comparison requires metrics other than cosine (e.g., spectral entropy) — SpectralEntropy's entropy-based similarity was developed as the alternative the Spectraverse authors also use

## Inputs

- Experimental MS/MS spectra in MGF format (e.g., human blood metabolomics dataset)
- Reference spectral libraries in MGF format: Spectraverse v1.0.1 (Zenodo 10.5281/zenodo.17870921), MassSpecGym, NPLIB1
- InChIKey identifiers for paired spectra (first 14 connectivity-block characters) for same-compound labeling

## Outputs

- Per-library annotation counts at each cosine similarity threshold (Spectraverse vs MassSpecGym vs NPLIB1)
- Pairwise cosine similarity matrices saved in numpy format per spectrum variant (raw, centroided, denoised, top-n)
- AUROC values per preprocessing variant and the Figure S3 comparison plot

## How to apply

Load the experimental spectra (e.g., human blood metabolomics MGF files) and each reference library into matchms spectrum objects, then run the library search independently against Spectraverse, MassSpecGym, and NPLIB1 using cosine similarity scoring with a 10 ppm precursor m/z tolerance and a minimum of 3 matched fragment ions. Count annotated spectra at each cosine similarity threshold and assemble per-library, per-threshold counts into a single table. For the AUROC variant, identify candidate pairs whose precursor m/z values match within 10 ppm, build a binary same-compound label matrix by comparing the first 14 characters (connectivity block) of InChIKeys, and compute pairwise cosine similarity matrices with the SpectralEntropy/MSEntropy-derived scoring code, saving them as numpy arrays. Rank cosine scores against the label matrix to obtain AUROC per spectrum variant (raw, centroided, electronically denoised, top-n retention). The rationale is that identical scoring parameters across libraries or variants make annotation counts and AUROC values directly comparable.

## Related tools

- **matchms** (Parses MGF spectra into spectrum objects, applies precursor m/z tolerance (10 ppm) and matched-peak minimum (3), and computes cosine similarity scores for library search and pairwise matrices) — https://github.com/matchms/matchms
- **SpectralEntropy** (Source of the cosine score calculation used in the spectraverse-analysis pipeline; cloned and moved into the spectraverse package) — https://github.com/YuanyueLi/SpectralEntropy
- **MSEntropy** (Provides the MSEntropy-derived cosine function used for custom centroided-variant scoring in the AUROC comparison)
- **spectral_denoising** (Generates the electronically denoised spectrum variant whose cosine-scored AUROC is compared against raw spectra)

## Examples

```
from matchms import calculate_scores; from matchms.similarity import CosineGreedyScore; results = calculate_scores(library_spectra, query_spectra, CosineGreedyScore(tolerance=0.01))
```

## Evaluation signals

- Annotation-count table covers all three libraries (Spectraverse, MassSpecGym, NPLIB1) at every cosine similarity threshold evaluated
- Spectraverse shows the highest annotation count at every threshold, consistent with the paper's reported comparison
- Precursor m/z matching is restricted to a 10 ppm tolerance and a minimum of 3 matched fragment ions is enforced for every library search
- Same-compound label matrix entries are determined by identical first-14-character (connectivity block) InChIKeys of paired spectra
- AUROC decreases modestly but consistently for centroided and denoised variants relative to raw spectra, with the top-n effect largely saturated at top-4096 ions

## Limitations

- Both centroiding and electronic denoising decreased AUROC, so these functions were deliberately excluded from the preprocessing pipeline
- The matchms library must be patched: a zip file replaces the original matchms libraries so that _get_neutral_mass ignores [M]+ or [M]− adducts with a reported charge of zero
- The spectral_denoising package requires Python 3.8–3.12 (RDkit has no compatible distribution for Python 3.13), while the analysis repo requires Python 3.10 or higher
- Cosine similarity discrimination depends on pair construction — only candidate pairs with precursor m/z matching within 10 ppm are evaluated

## Evidence

- [methods] Code used to calculate cosine score: [evidence span withheld — anchor status: not_found]
- [methods] SpectralEntropy setup in pipeline: [evidence span withheld — anchor status: not_found]
- [readme] matchms purpose statement: [evidence span withheld — anchor status: external_doc]
- [readme] SpectralEntropy benchmark rationale: [evidence span withheld — anchor status: external_doc]
