---
name: modified-cosine-score-computation
description: Use when benchmarking a learned spectral similarity score (e.g. Spec2Vec)
  against a classical peak-matching score, or when building molecular networks that
  account for neutral-loss shifts between fragment peaks of related molecules.
when_to_use_negative:
- When a trained embedding model is available and the goal is structural-analogue
  search — Spec2Vec scores correlate better with structural similarity than cosine-based
  scores
- When peak shifts between compared spectra do not correspond to chemically meaningful
  neutral losses (plain cosine is then the simpler baseline)
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Loads, filters (relative intensity >= 0.01), and computes pairwise modified
    cosine similarity scores between spectra
  repo: https://github.com/matchms/matchms
- name: Spec2Vec
  role: Learned-embedding similarity score that the modified cosine baseline is compared
    against
  repo: https://github.com/iomega/spec2vec
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:158470c0a2b9f60298f05d72845ab2289abb4d915c66299d8abe32cf1ad0c35d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Spec2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# modified-cosine-score-computation

## Summary

Computes all-pairs modified cosine similarity scores between MS/MS spectra as a baseline spectral similarity measure, used in the Spec2Vec paper to benchmark embedding-based scoring for molecular networking clustering quality.

## When to use

Use when benchmarking a learned spectral similarity score (e.g. Spec2Vec) against a classical peak-matching score, or when building molecular networks that account for neutral-loss shifts between fragment peaks of related molecules.

## When NOT to use

- When a trained embedding model is available and the goal is structural-analogue search — Spec2Vec scores correlate better with structural similarity than cosine-based scores
- When peak shifts between compared spectra do not correspond to chemically meaningful neutral losses (plain cosine is then the simpler baseline)

## Inputs

- Processed positive-ionization MS/MS spectra (MGF/JSON, e.g. AllPositive dataset, 95,320 spectra, Zenodo doi:10.5281/zenodo.3979010)
- Spectra parsed and metadata-cleaned with matchms

## Outputs

- All-pairs modified cosine similarity score matrix (sparse scores array)
- Molecular network with top-10 links per node above similarity threshold, split into Louvain clusters
- Fraction of well-clustered (mean Tanimoto >= 0.5) vs poorly-clustered spectra, e.g. fig6_well_poorly_clustered_fractions.png

## How to apply

Using matchms, load the processed spectra (e.g. the AllPositive dataset of 95,320 positive-ionization MS/MS spectra from Zenodo 10.5281/zenodo.3979010) and compute all-pairs modified cosine scores as the baseline comparison to Spec2Vec embeddings. Before scoring, filter peaks with relative intensity <0.01 compared to the highest-intensity peak, since the paper ignores such peaks for both cosine and modified cosine calculations. The modified cosine, unlike plain cosine, matches peaks that are shifted by a consistent mass difference (neutral loss) between two spectra, making it sensitive to structurally related molecules. Build the network by retaining the top-10 links per node above a similarity threshold, apply the Louvain algorithm to split large poorly connected clusters, and evaluate cluster quality by mean Tanimoto structural similarity across edges (well-clustered = mean Tanimoto >= 0.5).

## Related tools

- **matchms** (Loads, filters (relative intensity >= 0.01), and computes pairwise modified cosine similarity scores between spectra) — https://github.com/matchms/matchms
- **Spec2Vec** (Learned-embedding similarity score that the modified cosine baseline is compared against) — https://github.com/iomega/spec2vec

## Examples

```
from matchms import calculate_scores; from matchms.similarity import ModifiedCosineGreedy; scores = calculate_scores(spectra, spectra, ModifiedCosineGreedy(tolerance=0.1))
```

## Evaluation signals

- All-pairs computation over ~95,320 spectra (~82 million pairs) completes; the paper reports Spec2Vec finishing in ~300 s on an Intel i7-8550U CPU, so cosine-family runs should be of comparable or better order
- Peaks with relative intensity <0.01 are excluded before scoring, matching the paper's filtering rule for cosine and modified cosine
- In the well/poorly-clustered fraction plot, the modified cosine curve shows a lower fraction of well-clustered spectra (mean Tanimoto >= 0.5) than Spec2Vec at comparable settings
- Each network node retains at most top-10 links above the similarity threshold; Louvain is applied to split large, poorly connected clusters

## Limitations

- Cosine-based scores correlate less well with structural (Tanimoto) similarity than Spec2Vec scores, limiting clustering quality in molecular networking
- Unlike cosine scores, embedding-based alternatives need training data; conversely, modified cosine requires no training set but scales differently
- Performance demonstrated on LC-MS data only; utility for GC-MS data was not assessed

## Evidence

- [discussion] For both the cosine and modified cosine score calculations we ignored all peaks with relative intensities <0.01 compared to the highest intensity peak.: "For both the cosine and modified cosine score calculations we ignored all peaks with relative intensities <0.01 compared to the highest intensity peak."
- [intro] we further apply the Louvain algorithm to split up large, poorly connected clusters [26,27].: "we further apply the Louvain algorithm to split up large, poorly connected clusters [26,27]."
- [full_text] Spec2Vec clusters higher fractions of spectra into high structural similarity clusters than the modified cosine score: [evidence span withheld — anchor status: not_found]
- [readme] matchms is a versatile open-source Python package developed for importing, processing, cleaning, and comparing mass spectrometry data (MS/MS).: [evidence span withheld — anchor status: external_doc]
