---
name: mass-spectra-analog-search-feature-engineering
description: Use when re-ranking MS/MS analogue-search candidates (e.g. after selecting
  the top 2000 library spectra by MS2Deepscore) and you need to build or justify the
  five-feature vector that a random forest converts into a 0-1 match score.
when_to_use_negative:
- You are still preprocessing raw LC-MS data into features — use an LC-MS preprocessing
  tool (e.g. xcms) first; MS2Query performs no peak picking or clustering of similar
  MS2 spectra
- You only need exact spectral library matching by cosine or modified cosine score
  with fragment mass tolerance (e.g. 0.05 Da) and no analogue re-ranking
- Your analogue search strategy depends on precursor m/z preselection (e.g. 100 Da
  precursor m/z difference window) rather than embedding-based candidate selection
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2Query
  role: Hosts the random-forest re-ranking of the 2000 MS2Deepscore-preselected candidates
    using the five features
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Supervised spectral similarity model used to select top 2000 candidates and
    to supply the average MS2Deepscore feature over 10 similar library molecules
  repo: https://github.com/iomega/ms2query
- name: Spec2Vec
  role: Unsupervised spectral embedding providing the Spec2Vec similarity feature
    for the random forest
- name: scikit-learn
  role: Implements the random forest model and exposes the feature_importances_ attribute
    used for the importance analysis
- name: matchms
  role: Used for library cleaning (metadata cleanup, intensity normalization) and
    modified cosine score calculations on the spectra feeding the features
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:a683f6193fba4d586addd2a0813c474f35a546f4386e97b00311e236c45bff33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# mass-spectra-analog-search-feature-engineering

## Summary

Engineers the five candidate-ranking features used by MS2Query (Spec2Vec similarity, query precursor m/z, precursor m/z difference, average MS2Deepscore over 10 similar library molecules, and average Tanimoto score) and combines them with a scikit-learn random forest to re-rank MS2Deepscore-preselected library spectra, then quantifies per-feature importance via Gini feature_importances_.

## When to use

Use when re-ranking MS/MS analogue-search candidates (e.g. after selecting the top 2000 library spectra by MS2Deepscore) and you need to build or justify the five-feature vector that a random forest converts into a 0-1 match score.

## When NOT to use

- You are still preprocessing raw LC-MS data into features — use an LC-MS preprocessing tool (e.g. xcms) first; MS2Query performs no peak picking or clustering of similar MS2 spectra
- You only need exact spectral library matching by cosine or modified cosine score with fragment mass tolerance (e.g. 0.05 Da) and no analogue re-ranking
- Your analogue search strategy depends on precursor m/z preselection (e.g. 100 Da precursor m/z difference window) rather than embedding-based candidate selection

## Inputs

- Query MS2 spectra (mzML, MGF, msp, mzXML, json, USI, or pickled matchms object) in positive or negative ion mode
- Annotated mass spectral library spectra with pre-calculated MS2Deepscore and Spec2Vec embeddings (sqlite library folder from Zenodo 6124552 positive / 7104184 negative)
- Trained scikit-learn random forest model shipped with the MS2Query library files
- MS2Deepscore similarity scores for query vs. all library spectra

## Outputs

- Ranked list of library matches with random forest scores between 0 and 1 per library-query pair
- feature_importances.csv with the five impurity-based importance values sorted in descending order
- Filtered results after applying the minimum random-forest score threshold

## How to apply

First select the top 2000 library spectra by MS2Deepscore (no precursor m/z preselection, unlike modified cosine analogue search which filters on a 100 Da precursor m/z difference). For each of the 2000 preselected library-query pairs, calculate the five features: Spec2Vec similarity, query precursor m/z, precursor m/z difference, weighted average MS2Deepscore over 10 similar library molecules, and average Tanimoto score. Combine them with a scikit-learn random forest that predicts a score between 0 and 1 per library-query pair; the highest-scoring match is the analogue or exact match, and a minimum-score threshold filters unreliable matches. To validate which features drive performance, load the trained random forest, read the impurity-based feature_importances_ attribute for all five features, and write the values sorted in descending order; the paper reports that average MS2Deepscore has the biggest impact while all five features carry relevant information (Supplementary Table 1 and 2).

## Related tools

- **MS2Query** (Hosts the random-forest re-ranking of the 2000 MS2Deepscore-preselected candidates using the five features) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Supervised spectral similarity model used to select top 2000 candidates and to supply the average MS2Deepscore feature over 10 similar library molecules) — https://github.com/iomega/ms2query
- **Spec2Vec** (Unsupervised spectral embedding providing the Spec2Vec similarity feature for the random forest)
- **scikit-learn** (Implements the random forest model and exposes the feature_importances_ attribute used for the importance analysis)
- **matchms** (Used for library cleaning (metadata cleanup, intensity normalization) and modified cosine score calculations on the spectra feeding the features)

## Examples

```
import joblib; rf = joblib.load('ms2query_random_forest.model'); imp = sorted(zip(['spec2vec_sim','query_precursor_mz','precursor_mz_diff','avg_ms2deepscore','avg_tanimoto'], rf.feature_importances_), key=lambda x: -x[1])
```

## Evaluation signals

- The impurity-based feature_importances_ of the random forest must show Average MS2Deepscore over 10 similar library molecules as the highest-importance feature among the five
- All five features must have nonzero importance, consistent with Supplementary Table 1 and 2
- The five importance values must sum to 1.0 and be written sorted in descending order with a column naming each feature
- Feature computation must occur only for the top 2000 library spectra selected by MS2Deepscore, not the full library
- Predicted random forest scores must lie in the 0-1 range, with the top-scoring library spectrum returned as the match

## Limitations

- Details of other tested features are only in Supplementary Note 3, which was not included, so alternative feature sets cannot be reproduced from the main text
- A minimum-score threshold on the random forest output is needed to filter unreliable matches; choosing that threshold for new data is not specified
- Impurity-based feature importance is the analysis reported (Supplementary Tables 1 and 2); no permutation-based importance is described

## Evidence

- [intro] First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model.: "First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model."
- [discussion] To optimally rank these 2000 spectra, MS2Query calculates 5 features which are combined by a random forest model.: "To optimally rank these 2000 spectra, MS2Query calculates 5 features which are combined by a random forest model."
- [discussion] Additional analysis of the feature importance indicates that each of the five features used contain relevant information for correctly ranking candidate structures (Supplementary Table 1 and 2).: "Additional analysis of the feature importance indicates that each of the five features used contain relevant information for correctly ranking candidate structures (Supplementary Table 1 and 2)."
- [readme] MS2Query optimizes re-ranking the best analogue or exact match at the top by using a random forest that combines 5 features.: [evidence span withheld — anchor status: external_doc]
- [discussion] The implementation of scikit-learn37 was used for the random forest model.: "The implementation of scikit-learn37 was used for the random forest model."
