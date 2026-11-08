---
name: msms-spectral-library-search
description: Use when you have experimental metabolomics MS/MS spectra (e.g. MGF-format
  human blood spectra) and want to count how many can be annotated by reference-library
  search, comparing coverage across multiple candidate libraries.
when_to_use_negative:
- When the goal is building or curating a library (adduct correction, duplicate removal,
  structure standardization) rather than annotating experimental spectra with an existing
  one
- When similarity scoring must use spectral entropy or MS2DeepScore instead of cosine
  dot-product, since the article's benchmark workflow uses matchms cosine similarity
  specifically
- When precursor m/z or fragmentation parameters differ from the 10 ppm / ≥3 matched-fragment
  setup used for the human blood benchmark
edam_operation: http://edamontology.org/operation_3790
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: matchms
  role: Parses MGF/library files into spectrum objects, applies precursor m/z tolerance
    and matched-fragment-ion thresholds, and computes cosine similarity scores for
    library search
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Provides spectral similarity machinery (cosine score computation) used alongside
    matchms in the article's analysis environment
  repo: https://github.com/YuanyueLi/SpectralEntropy.git
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:69a1d5a71eafc9141f7c89daacf68e169b82264f5024f2fd968e5bd9d1d67eba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - SpectralEntropy
  - matchms
  edam_operation: http://edamontology.org/operation_3790
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# msms-spectral-library-search

## Summary

Annotates experimental MS/MS spectra by spectral library search against curated reference libraries (Spectraverse, MassSpecGym, NPLIB1) using cosine similarity scoring with matchms, comparing how many spectra each library annotates across cosine-similarity thresholds.

## When to use

Use when you have experimental metabolomics MS/MS spectra (e.g. MGF-format human blood spectra) and want to count how many can be annotated by reference-library search, comparing coverage across multiple candidate libraries.

## When NOT to use

- When the goal is building or curating a library (adduct correction, duplicate removal, structure standardization) rather than annotating experimental spectra with an existing one
- When similarity scoring must use spectral entropy or MS2DeepScore instead of cosine dot-product, since the article's benchmark workflow uses matchms cosine similarity specifically
- When precursor m/z or fragmentation parameters differ from the 10 ppm / ≥3 matched-fragment setup used for the human blood benchmark

## Inputs

- Experimental MS/MS spectra in MGF format (e.g. human blood metabolomics spectra)
- Reference spectral library files in MGF/JSON format: Spectraverse v1.0.1 (Zenodo 10.5281/zenodo.17870921), MassSpecGym, NPLIB1

## Outputs

- Annotation-count table with number of experimental spectra annotated per library at each cosine similarity threshold
- Per-library ranking confirming which library (e.g. Spectraverse) annotates the most spectra

## How to apply

Load the experimental spectra and each reference library (Spectraverse v1.0.1 from Zenodo doi:10.5281/zenodo.17870921, MassSpecGym, NPLIB1) into matchms spectrum objects in Python 3.10+. For each library, run an independent matchms spectral library search using cosine similarity scoring with a 10 ppm precursor m/z tolerance and a minimum of 3 matched fragment ions. Vary the cosine-similarity threshold and count the number of experimental spectra annotated at each threshold per library. Assemble per-library, per-threshold counts into a single comparison table; accept the result only when Spectraverse annotates the most spectra at every threshold, consistent with its ~488,630 spectra over 44,237 unique small molecules.

## Related tools

- **matchms** (Parses MGF/library files into spectrum objects, applies precursor m/z tolerance and matched-fragment-ion thresholds, and computes cosine similarity scores for library search) — https://github.com/matchms/matchms
- **SpectralEntropy** (Provides spectral similarity machinery (cosine score computation) used alongside matchms in the article's analysis environment) — https://github.com/YuanyueLi/SpectralEntropy.git

## Examples

```
from matchms import calculate_scores; from matchms.similarity import CosineGreedy; scores = calculate_scores(references, queries, CosineGreedy(tolerance=0.01), 'precursor_mz')
```

## Evaluation signals

- Annotation counts are produced for all three libraries (Spectraverse, MassSpecGym, NPLIB1) at every cosine similarity threshold evaluated
- Spectraverse shows the highest annotation count at every threshold, matching the paper's reported ranking
- Matched fragment ions per annotation ≥ 3 and precursor m/z deviation ≤ 10 ppm
- Counts are reproducible from the same experimental MGF and library files without manual curation steps
- Table covers all thresholds with consistent per-library comparisons (same scoring function and tolerance throughout)

## Limitations

- Only a fraction of MS/MS spectra can be annotated because libraries are limited in size and biased toward well-studied, commercially available compounds
- Comparison outcome depends on the cosine-similarity scoring and 10 ppm tolerance; other scoring schemes (e.g. spectral entropy) could change the ranking
- Commercial spectral libraries cannot readily be used to train reproducible machine-learning models, so this workflow is limited to publicly redistributable libraries

## Evidence

- [results] Library searching against Spectraverse annotated the greatest number of experimental human blood MS/MS spectra among the three libraries, at every cosine similarity threshold evaluated.: [evidence span withheld — anchor status: not_found]
- [methods] For each reference library, run a matchms spectral library search using cosine similarity scoring with a 10 ppm precursor m/z tolerance and a minimum of 3 matched fragment ions.: [evidence span withheld — anchor status: not_found]
- [intro] Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small molecules and is freely available from Zenodo at https://doi.org/10.5281/zenodo.17870921.: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small
molecules and is freely available from Zenodo at https://doi.org/10.5281/zenodo.17870921."
- [intro] Typically, however, only a fraction of MS/MS spectra can be annotated through spectral library search, in part because these libraries are limited in size and are generally biased toward: "Typically, however, only a fraction of MS/MS spectra can be annotated
through spectral library search, in part
because these libraries are limited in size and are generally biased
toward"
- [readme] Matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: [evidence span withheld — anchor status: external_doc]
