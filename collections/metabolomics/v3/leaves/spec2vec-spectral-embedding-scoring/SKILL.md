---
name: spec2vec-spectral-embedding-scoring
description: Use when comparing large collections of positive-ionization MS/MS spectra
  (e.g. GNPS libraries, ~95k spectra) where cosine or modified cosine poorly reflects
  structural similarity, or when scalable all-pairs analogue search / molecular networking
  is needed.
when_to_use_negative:
- When no training data covering the query fragment/loss features exists — Spec2Vec
  needs training data unlike cosine scores, and missing words become an issue on smaller
  training sets or with more m/z decimals included
- For GC-MS data — the study only demonstrated performance on LC-MS data
- When input spectra are already embedded vectors or a feature table rather than peak
  lists in MS/MS file formats matchms can import
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: spec2vec
  role: Core library converting spectra to documents, training/loading Word2Vec models,
    and computing Spec2Vec similarity scores
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Imports, cleans, and filters MS/MS spectra (metadata cleaning, peak filtering)
    and computes cosine/modified cosine baseline scores
  repo: https://github.com/matchms/matchms
- name: Word2Vec (gensim)
  role: Trains the embedding model on documents of peak and loss words from the chosen
    dataset
- name: spec2vec_gnps_data_analysis
  role: Notebooks and plotting functions reproducing the molecular networking benchmark
    (fig6) on GNPS data
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:668b9a9623b83fd2db79a8ea8bc6ef1e1f53387f4391914e8a6ad9edb2d46295
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - Spec2Vec
  - Word2Vec (gensim)
  - matchms
  - spec2vec
  - spec2vec_gnps_data_analysis
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 29
---

# spec2vec-spectral-embedding-scoring

## Summary

Scores MS/MS spectral similarity using Spec2Vec embeddings: spectra are converted to documents of peak/loss words, a Word2Vec model (gensim) learns fragment/loss relationships, and similarity is the cosine between intensity-weighted spectrum vectors. It correlates better with structural (Tanimoto) similarity than cosine-based scores and scales to ~82M all-pairs comparisons in ~300s.

## When to use

Use when comparing large collections of positive-ionization MS/MS spectra (e.g. GNPS libraries, ~95k spectra) where cosine or modified cosine poorly reflects structural similarity, or when scalable all-pairs analogue search / molecular networking is needed.

## When NOT to use

- When no training data covering the query fragment/loss features exists — Spec2Vec needs training data unlike cosine scores, and missing words become an issue on smaller training sets or with more m/z decimals included
- For GC-MS data — the study only demonstrated performance on LC-MS data
- When input spectra are already embedded vectors or a feature table rather than peak lists in MS/MS file formats matchms can import

## Inputs

- Positive-ionization MS/MS spectra (e.g. GNPS ALL_GNPS.json library, MGF/mzML/JSON readable by matchms)
- AllPositive dataset (95,320 spectra; Zenodo deposition 10.5281/zenodo.3979010) parsed with matchms
- Pre-trained Spec2Vec Word2Vec model (Zenodo record 4173596 for AllPositive; 3978054 for UniqueInchikey)
- Structural annotations (InChIKeys linked via PubChem lookup) for Tanimoto-based cluster evaluation

## Outputs

- All-pairs Spec2Vec similarity score matrix (95,320^2 ≈ 82M pairs, computed in ~300s on an Intel i7-8550U CPU)
- Spectrum embedding vectors (intensity-weighted sums of learned peak/loss word vectors)
- Molecular networks (top-10 links per node, Louvain-split clusters) with per-cluster mean Tanimoto similarity, e.g. fig6_well_poorly_clustered_fractions.png

## How to apply

Clean spectra with matchms: remove peaks outside m/z [0, 1000], discard spectra with fewer than 10 peaks, and remove spectra lacking InChIKey annotation when training/query overlap must be avoided. Convert each spectrum to a document where every peak is a word encoding its m/z position to a defined decimal precision, and add neutral losses between 5.0 and 200.0 Da as 'loss@xxx.xx' words; for Spec2Vec cap kept peaks at 0.5·parent_mass and ignore peaks with relative intensity <0.01 for cosine/modified cosine baselines. Train (or load a pre-trained, e.g. Zenodo record 4173596 trained on the AllPositive 95,320-spectrum dataset) a Word2Vec model via gensim, then compute a spectrum vector vS as the intensity-weighted sum of its word vectors. Score pairs by the cosine between the two spectrum vectors; for spectra with many out-of-model words, apply a missing-fraction threshold (e.g. <0.05) to avoid unreliable scores. For molecular networking, keep top-10 links per node above the similarity threshold and apply the Louvain algorithm to split large, poorly connected clusters, then judge clusters by mean Tanimoto similarity across edges (well-clustered if >= 0.5).

## Related tools

- **spec2vec** (Core library converting spectra to documents, training/loading Word2Vec models, and computing Spec2Vec similarity scores) — https://github.com/iomega/spec2vec
- **matchms** (Imports, cleans, and filters MS/MS spectra (metadata cleaning, peak filtering) and computes cosine/modified cosine baseline scores) — https://github.com/matchms/matchms
- **Word2Vec (gensim)** (Trains the embedding model on documents of peak and loss words from the chosen dataset)
- **spec2vec_gnps_data_analysis** (Notebooks and plotting functions reproducing the molecular networking benchmark (fig6) on GNPS data) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Examples

```
conda create --name spec2vec_analysis python=3.7; conda activate spec2vec_analysis; conda install --channel nlesc --channel bioconda --channel conda-forge spec2vec
```

## Evaluation signals

- Spec2Vec curve shows a higher fraction of well-clustered spectra (mean Tanimoto >= 0.5 per cluster) than the modified cosine baseline at comparable settings
- All-pairs Spec2Vec computation over 95,320 spectra (~82M pairs) completes on the order of a few minutes (~300s on an Intel i7-8550U CPU)
- Missing-word fraction per spectrum falls below the chosen threshold (e.g. <0.05) before trusting returned scores
- Spectra correctly converted to documents: peak words encode m/z to the defined decimal precision and losses are formatted as 'loss@xxx.xx' within 5.0–200.0 Da
- Pre-trained model vocabulary covers the query data — large fractions of out-of-model peaks/losses indicate the model must be retrained on the target dataset

## Limitations

- Spec2Vec requires training data to learn fragment peak relationships, unlike cosine scores; the training set must include a large fraction of the fragment ions and losses seen in query data
- Network analysis applications may benefit from a massive training set or retraining including new experimental spectra; missing words become relevant with smaller training sets or more m/z decimals
- Only demonstrated on LC-MS data; GC-MS utility not assessed, and Tanimoto on fingerprints is a pragmatic (not definitive) structural similarity measure

## Evidence

- [discussion] Spectrum vector as weighted sum of word vectors: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [discussion] Loss words added between 5.0 and 200.0 Da: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] Peak cap scales with parent mass: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] Missing-fraction threshold avoids out-of-model scores: "By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided."
- [discussion] Training data requirement vs cosine: "one limitation of Spec2Vec as compared to cosine scores is that it needs training data to learn the fragment peak relationships"
- [intro] Louvain applied to split poorly connected clusters: "we further apply the Louvain algorithm to split up large, poorly connected clusters [26,27]."
- [readme] README: install via conda and run notebooks: [evidence span withheld — anchor status: external_doc]
- [readme] README: molecular networking tutorial part III: [evidence span withheld — anchor status: external_doc]
