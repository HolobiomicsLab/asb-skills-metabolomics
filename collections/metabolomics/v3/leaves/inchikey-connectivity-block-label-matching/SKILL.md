---
name: inchikey-connectivity-block-label-matching
description: Use when you need ground-truth same/different-compound labels to benchmark
  spectral similarity scores (cosine, entropy) on MS/MS library pairs, and both spectra
  carry InChIKey annotations plus precursor m/z metadata.
when_to_use_negative:
- The analysis goal requires distinguishing stereoisomers or distinguishing protonation/charge
  states — the 14-character connectivity block ignores the stereochemistry (characters
  15-18) and ionization layers of the InChIKey
- Spectra lack InChIKey annotations or reliable precursor m/z metadata, so candidate
  pairs cannot be filtered at 10 ppm or labeled by connectivity block
- Input is a single spectrum or a one-to-many library search where pairwise same/different
  labels and AUROC ranking are not the evaluation target
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: matchms
  role: Parse MGF spectra and access precursor m/z and InChIKey metadata used to build
    candidate pairs and labels
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Provides the cosine similarity scoring code whose outputs are ranked against
    the InChIKey-derived label matrix
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: MSEntropy
  role: MSEntropy-derived cosine scoring function used to compute pairwise similarity
    matrices for each spectrum variant
  repo: https://github.com/YuanyueLi/MSEntropy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:90515b6486861b13239adc72487d84a8e00bfa7c2fb27fdf5999164eb3be68b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MSEntropy
  - Python
  - SpectralEntropy
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# inchikey-connectivity-block-label-matching

## Summary

Constructs binary same-compound ground-truth labels for MS/MS spectrum pairs by comparing the first 14 characters (connectivity/skeleton block) of their InChIKeys, restricted to candidate pairs whose precursor m/z values match within 10 ppm. These labels support AUROC-style evaluation of spectral similarity scores (e.g., cosine) for same-vs-different compound discrimination.

## When to use

Use when you need ground-truth same/different-compound labels to benchmark spectral similarity scores (cosine, entropy) on MS/MS library pairs, and both spectra carry InChIKey annotations plus precursor m/z metadata.

## When NOT to use

- The analysis goal requires distinguishing stereoisomers or distinguishing protonation/charge states — the 14-character connectivity block ignores the stereochemistry (characters 15-18) and ionization layers of the InChIKey
- Spectra lack InChIKey annotations or reliable precursor m/z metadata, so candidate pairs cannot be filtered at 10 ppm or labeled by connectivity block
- Input is a single spectrum or a one-to-many library search where pairwise same/different labels and AUROC ranking are not the evaluation target

## Inputs

- MGF-format MS/MS spectra (raw and preprocessed variants) with InChIKey and precursor m/z metadata, e.g. from the Spectraverse Zenodo deposit (10.5281/zenodo.17870921)
- Parsed matchms Spectrum objects or an equivalent table of InChIKeys and precursor m/z values per spectrum

## Outputs

- Binary same-compound label matrix (numpy array) indexed over candidate spectrum pairs
- Candidate-pair index restricted to precursor m/z matches within 10 ppm
- Per-variant AUROC values comparing cosine similarity scores against the label matrix (e.g., Figure S3-style comparison table)

## How to apply

First identify candidate spectrum pairs whose precursor m/z values match within a 10 ppm tolerance, so that only chemically plausible pairs are labeled. For each candidate pair, compare the first 14 characters of the two InChIKeys — the connectivity block, which encodes the molecular skeleton but not stereochemistry or protonation/charge layers. Assign label 1 (same compound) when the 14-character blocks are identical, and 0 (different compound) otherwise, yielding a binary label matrix. Pair this label matrix with per-variant cosine similarity matrices (computed e.g. with matchms and SpectralEntropy/MSEntropy-derived scoring on raw, centroided, denoised, and top-n ion-retained MGF variants from the Spectraverse Zenodo deposit 10.5281/zenodo.17870921) and rank cosine scores against the labels to compute AUROC per variant. The 14-character cutoff is deliberate: using the full 25-character InChIKey would split stereoisomers and protonation states into 'different', which is not the intended spectral-search task.

## Related tools

- **matchms** (Parse MGF spectra and access precursor m/z and InChIKey metadata used to build candidate pairs and labels) — https://github.com/matchms/matchms
- **SpectralEntropy** (Provides the cosine similarity scoring code whose outputs are ranked against the InChIKey-derived label matrix) — https://github.com/YuanyueLi/SpectralEntropy
- **MSEntropy** (MSEntropy-derived cosine scoring function used to compute pairwise similarity matrices for each spectrum variant) — https://github.com/YuanyueLi/MSEntropy

## Examples

```
labels = numpy.array([[1 if ik_i[:14] == ik_j[:14] else 0 for ik_j in iks] for ik_i in iks])  # after matchms parses MGF and pairs are prefiltered at 10 ppm precursor m/z
```

## Evaluation signals

- Label matrix is binary and symmetric; pairs with identical full 14-character InChIKey prefixes are labeled 1 and all others 0
- No label is assigned outside the candidate set, i.e. every labeled pair has precursor m/z values matching within the 10 ppm tolerance
- AUROC computed per spectrum variant (raw, centroided, denoised, top-n retention) is finite and reproducible when scores are re-ranked against the saved label matrix
- Known same-compound duplicate spectra (identical InChIKey connectivity block) consistently receive cosine scores near 1.0 and label 1
- Saved similarity matrices and label matrix share consistent dimensions and spectrum ordering

## Limitations

- Labels treat any two compounds sharing a molecular skeleton as 'the same compound', so stereoisomers and distinct protonation states sharing a connectivity block are conflated
- The 10 ppm precursor-m/z prefilter assumes accurate precursor masses; low-resolution or misannotated precursor metadata will produce false candidate pairs
- AUROC outcomes depend on the upstream preprocessing variants; both centroiding and electronic denoising modestly but consistently decreased the AUROC in the source study

## Evidence

- [supplementary] Construct a binary same-compound label matrix by comparing the first 14 characters (connectivity block) of the InChIKeys of paired spectra.: [evidence span withheld — anchor status: not_found]
- [supplementary] Precursor tolerance for candidate pairs: [evidence span withheld — anchor status: not_found]
- [readme] matchms parsing and comparison role: [evidence span withheld — anchor status: external_doc]
- [supplementary] AUROC evaluation of spectral similarity variants: [evidence span withheld — anchor status: not_found]
