---
name: msms-fragment-lfp-similarity-scoring
description: Use when you have chained MS/MS fragment ion-intensity profiles of two
  (or more) isomeric compounds across multiple collision energies and need per-fragment
  similarity scores to distinguish them.
when_to_use_negative:
- Only a single MS/MS spectrum per compound is available (no multi-collision-energy
  profiles) — cross-sectional algorithms like entropy similarity or dot product are
  the appropriate choice instead
- The goal is general metabolite identification against a spectral library rather
  than pairwise isomer differentiation — the full identification pipeline (ROC-derived
  criteria, CQMU database) is needed, not just pairwise LFP scoring
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Jaccard similarity algorithm
  role: Optimized Jaccard similarity (equation 1) computed longitudinally per fragment
    across collision energies, treating predominant and minute fragments independently
- name: Jupyter Notebook
  role: Environment in which the 001-007.ipynb notebooks implementing the similarity
    computations are run
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Python 3.8.13
  role: Runtime for the LFP similarity scoring code (conda env 'longprof')
- name: mzbatch.py
  role: Library of mass spectrometry data processing methods used within the similarity-calculation
    notebooks
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:de0775081ee8c5bb7edc0daee6793bae70bcad674f5ded6f3744c4b2a4c029a2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# msms-fragment-lfp-similarity-scoring

## Summary

Computes longitudinal Jaccard similarity between fragment ion-intensity profiles (LFPs) of compound pairs measured across 13 collision energies plus the stepped NCE 20/40/60% combined spectrum, to differentiate isomers such as leucine/isoleucine that cross-sectional spectral matching cannot separate.

## When to use

Use when you have chained MS/MS fragment ion-intensity profiles of two (or more) isomeric compounds across multiple collision energies and need per-fragment similarity scores to distinguish them.

## When NOT to use

- Only a single MS/MS spectrum per compound is available (no multi-collision-energy profiles) — cross-sectional algorithms like entropy similarity or dot product are the appropriate choice instead
- The goal is general metabolite identification against a spectral library rather than pairwise isomer differentiation — the full identification pipeline (ROC-derived criteria, CQMU database) is needed, not just pairwise LFP scoring

## Inputs

- Chained LFP inputs: fragment ion-intensity profiles of leucine and isoleucine across 13 individual collision energies
- Stepped NCE 20/40/60% combined MS/MS spectrum for each compound
- Fragment m/z lists for each compound's LFPs

## Outputs

- lfp_similarity_scores.csv — table of per-fragment longitudinal Jaccard scores at each of the 13 collision energies
- Cross-sectional comparison scores from the stepped NCE 20/40/60% combined spectrum (Jaccard, classic dot product, modified dot product, entropy similarity)

## How to apply

Load the chained LFP inputs — fragment ion-intensity profiles of leucine and isoleucine across 13 individual collision energies plus the stepped NCE 20/40/60% combined spectrum — produced by the upstream LFP-construction sub-task. For each fragment m/z, compute the longitudinal Jaccard similarity (equation 1 of the paper, with the optimized Jaccard algorithm treating predominant and minute fragments independently) between the two compounds' LFPs at each of the 13 individual collision energies. Repeat the same Jaccard computation on the stepped NCE 20/40/60% combined spectrum to obtain cross-sectional comparison scores. Compute the classic dot product, modified dot product, and entropy similarity on the stepped combined spectrum as cross-sectional benchmarks. Assemble all per-fragment and per-algorithm scores into a table written to lfp_similarity_scores.csv; rationale: cross-sectional evaluation weighs predominant fragments heavily and diminishes minute fragments that reflect subtle structural differences, whereas the optimized Jaccard treats the two independently.

## Related tools

- **Jaccard similarity algorithm** (Optimized Jaccard similarity (equation 1) computed longitudinally per fragment across collision energies, treating predominant and minute fragments independently)
- **Jupyter Notebook** (Environment in which the 001-007.ipynb notebooks implementing the similarity computations are run) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Python 3.8.13** (Runtime for the LFP similarity scoring code (conda env 'longprof'))
- **mzbatch.py** (Library of mass spectrometry data processing methods used within the similarity-calculation notebooks) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; jupyter notebook 001-007.ipynb
```

## Evaluation signals

- Visually similar LFPs must receive Jaccard similarity scores of at least 0.9
- Visually different LFPs must receive Jaccard similarity scores of less than 0.1
- Scores must be produced at each of the 13 individual collision energies plus the stepped NCE 20/40/60% combined spectrum
- Benchmark columns (classic dot product, modified dot product, entropy similarity) must appear alongside Jaccard scores in lfp_similarity_scores.csv so cross-sectional vs longitudinal behavior can be compared
- Scores should be reproducible by running 001-007.ipynb cells in sequence in the longprof conda environment (python 3.8.13)

## Limitations

- Scores depend on upstream LFP construction; the skill assumes chained LFP inputs across 13 collision energies are already available
- The Jaccard-based differentiation was demonstrated for leucine/isoleucine among 19 algorithms; performance on other isomer pairs is not established by this evidence
- Validation of the method's identification criteria relies on ROC analysis of the in-house database CQMD, whose availability/accession is not stated

## Evidence

- [other] threshold behavior of Jaccard scores: [evidence span withheld — anchor status: not_found]
