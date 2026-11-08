---
name: spectral-library-matching
description: Use when you have query MS/MS spectra with precursor m/z to match against
  a curated library (e.g. GNPS ALL_GNPS.json) and need to compare embedding-based
  (Spec2Vec) vs peak-matching (cosine) retrieval accuracy.
when_to_use_negative:
- 'No suitable training corpus exists: Spec2Vec needs training data to learn fragment/loss
  relationships, whereas cosine scores do not — if only an exact-peak-match comparison
  is needed, use cosine directly'
- Query spectra contain a large fraction of fragment/loss features absent from the
  Word2Vec training set (missing-words regime, e.g. small training sets or more m/z
  decimals), so embedding scores become unreliable
- Data are GC-MS spectra or already-processed feature tables rather than positive-mode
  LC-MS/MS peak lists — performance was only demonstrated on LC-MS data
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Spec2Vec
  role: Computes spectrum embeddings (intensity-weighted sums of Word2Vec word vectors)
    and the cosine similarity between spectrum vectors used as the library-matching
    score
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Imports, cleans and filters spectra (m/z range [0, 1000], min 10 peaks, relative-intensity
    cutoff for cosine) and provides the cosine baseline similarity score
  repo: https://github.com/matchms/matchms
- name: gensim (Word2Vec)
  role: Trains the Word2Vec model on all library documents of peak and loss words
- name: spec2vec_gnps_data_analysis
  role: Reference notebooks and plotting functions that reproduce the library-matching
    benchmark and Fig 4
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:dda7e8bcc65290706af6d2b677975ca20f4faded9be5c7ad7e1893625f75d544
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - MS2DECIDE
  - Spec2Vec
  - gensim (Word2Vec)
  - matchms
  - spec2vec_gnps_data_analysis
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# spectral-library-matching

## Summary

Matches unknown query MS/MS spectra against a spectral library by scoring query-candidate pairs with Spec2Vec spectral embeddings and benchmarking against cosine scores, sweeping similarity thresholds to build accuracy and true/false-positive curves (Fig 4 of the Spec2Vec paper).

## When to use

Use when you have query MS/MS spectra with precursor m/z to match against a curated library (e.g. GNPS ALL_GNPS.json) and need to compare embedding-based (Spec2Vec) vs peak-matching (cosine) retrieval accuracy.

## When NOT to use

- No suitable training corpus exists: Spec2Vec needs training data to learn fragment/loss relationships, whereas cosine scores do not — if only an exact-peak-match comparison is needed, use cosine directly
- Query spectra contain a large fraction of fragment/loss features absent from the Word2Vec training set (missing-words regime, e.g. small training sets or more m/z decimals), so embedding scores become unreliable
- Data are GC-MS spectra or already-processed feature tables rather than positive-mode LC-MS/MS peak lists — performance was only demonstrated on LC-MS data

## Inputs

- Cleaned AllPositive GNPS library spectra (94,320 spectra; positive ionization mode, from GNPS ALL_GNPS.json cleaned via matchms; Zenodo record 3979010)
- 1000 held-out query spectra (from AllPositive InChIKey-annotated subset, non-overlapping with training data)
- Trained Spec2Vec Word2Vec model (Zenodo 4173596 for AllPositive; 3978054/3979074 for UniqueInchikey)

## Outputs

- Accuracy and true/false-positive match rates as functions of the similarity threshold (Fig 4 curves)
- Threshold sweep table written to library_matching_results.csv
- Best-matching library candidate per query spectrum at each threshold

## How to apply

Load the cleaned AllPositive GNPS dataset (Zenodo record 3979010) and process with matchms: keep peaks with m/z in [0, 1000], discard spectra with fewer than 10 peaks, and add neutral losses between 5.0 and 200.0 Da as loss words. Convert the ~94,320 library spectra into documents where each peak is a word encoding its m/z position to a defined decimal precision, capping kept peaks at 0.5 * parent_mass for Spec2Vec, then train a gensim Word2Vec model on all documents (pre-trained models on Zenodo 4173596 for AllPositive or 3978054/3979074 for UniqueInchikey can be used). Compute spectrum vectors as intensity-weighted sums of word vectors (spec2vec calc_vector) and score query-candidate pairs with the cosine between spectrum vectors; for the cosine baseline, ignore peaks with relative intensity <0.01. Pre-select candidate matches for each of the 1000 query spectra by precursor m/z within 1 ppm tolerance, pick the highest-scoring candidate above each similarity threshold, and optionally apply a missing-fraction threshold (e.g. <0.05) to avoid scoring spectra far outside the learned peak/loss vocabulary. Sweep the threshold, recording the fraction of correctly matched queries (accuracy) plus true-positive and false-positive match rates, and plot curves versus threshold.

## Related tools

- **Spec2Vec** (Computes spectrum embeddings (intensity-weighted sums of Word2Vec word vectors) and the cosine similarity between spectrum vectors used as the library-matching score) — https://github.com/iomega/spec2vec
- **matchms** (Imports, cleans and filters spectra (m/z range [0, 1000], min 10 peaks, relative-intensity cutoff for cosine) and provides the cosine baseline similarity score) — https://github.com/matchms/matchms
- **gensim (Word2Vec)** (Trains the Word2Vec model on all library documents of peak and loss words)
- **spec2vec_gnps_data_analysis** (Reference notebooks and plotting functions that reproduce the library-matching benchmark and Fig 4) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Examples

```
conda create --name spec2vec_analysis python=3.7 && conda activate spec2vec_analysis && conda install --channel nlesc --channel bioconda --channel conda-forge spec2vec
```

## Evaluation signals

- Best threshold yields up to 88% correctly matched query spectra on the AllPositive library-matching split
- Accuracy and true/false-positive curves versus similarity threshold qualitatively match the published Fig 4
- Spec2Vec shows both higher accuracy and retrieval rates than cosine matching, and a better true/false-positive ratio at all thresholds
- Candidate pre-selection restricts matches to precursor m/z within 1 ppm tolerance before scoring
- Sweep results are written to library_matching_results.csv with one row per threshold

## Limitations

- Spec2Vec requires training data to learn fragment peak relationships, unlike cosine scores
- The training set must include a large fraction of the fragment/loss features present in the query data; missing words become relevant for small training sets or when more m/z decimals are included
- Set a missing-fraction threshold (e.g. <0.05) to avoid returning Spec2Vec scores for spectra far outside the learned peaks and losses
- Performance was only demonstrated on LC-MS data; GC-MS utility was not assessed

## Evidence

- [intro] precursor m/z pre-selection: "potentially matching spectra were pre-selected based on precursor-m/z matches (tolerance = 1ppm) before the highest scoring candidate above a similarity threshold was chosen."
- [discussion] peak filtering parameters: "we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks."
- [discussion] Spec2Vec peak cap: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] loss words: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] spectrum vector definition: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [results] 88% accuracy finding: [evidence span withheld — anchor status: not_found]
- [readme] environment setup from README: [evidence span withheld — anchor status: external_doc]
