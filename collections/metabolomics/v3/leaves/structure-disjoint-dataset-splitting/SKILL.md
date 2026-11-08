---
name: structure-disjoint-dataset-splitting
description: Use when training or benchmarking spectral similarity or library-matching
  models (e.g. MS2Deepscore, Spec2Vec) on annotated spectral libraries like GNPS,
  where spectra of the same or closely related compounds must not appear in both train
  and test splits.
when_to_use_negative:
- Input is a set of query spectra to annotate with a pre-trained MS2Query library
  — no splitting or retraining is involved; just run inference.
- Input spectra lack consistent structural annotations (InChIKeys or resolvable names/formulas),
  making structure-disjoint splitting infeasible without additional curation.
- The goal is hyperparameter tuning within a single model's training pipeline rather
  than leakage-free benchmarking across MS2Deepscore, Spec2Vec, and MS2Query.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: matchms
  role: Cleaning the GNPS library (peak filtering, intensity normalization, metadata
    harmonization) and adding missing InChIKeys via PubChem lookup before splitting
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Supervised siamese model trained on the training partition of spectrum pairs
    labeled by chemical structural similarity
  repo: https://github.com/iomega/ms2query
- name: Spec2Vec
  role: Unsupervised embedding model trained (30 epochs, peaks binned to 2 decimals)
    on the training partition only
  repo: https://github.com/iomega/ms2query
- name: MS2Query
  role: Hosts the random forest re-ranker (250 estimators, max depth 5, MSE loss)
    trained on the training partition and packages the models with the split manifest
  repo: https://github.com/iomega/ms2query
- name: GNPS
  role: Source of the annotated spectral library (ALL_GNPS.mgf) used to build the
    splits
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:158fdfaf9c046356c8e9f8ab9f48c24d505bc5560906d3cc4bc9486e924c3a26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - GNPS
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Structure-disjoint dataset splitting for spectral ML model training

## Summary

Split mass spectral datasets into train/test sets based on compound structure (InChIKey) rather than spectrum identity, so no test spectrum's 2D structure appears in the training set of MS2Deepscore, Spec2Vec, or the MS2Query re-ranker, preventing data leakage between benchmark and training data.

## When to use

Use when training or benchmarking spectral similarity or library-matching models (e.g. MS2Deepscore, Spec2Vec) on annotated spectral libraries like GNPS, where spectra of the same or closely related compounds must not appear in both train and test splits.

## When NOT to use

- Input is a set of query spectra to annotate with a pre-trained MS2Query library — no splitting or retraining is involved; just run inference.
- Input spectra lack consistent structural annotations (InChIKeys or resolvable names/formulas), making structure-disjoint splitting infeasible without additional curation.
- The goal is hyperparameter tuning within a single model's training pipeline rather than leakage-free benchmarking across MS2Deepscore, Spec2Vec, and MS2Query.

## Inputs

- GNPS ALL_GNPS.mgf library spectra (MGF format, downloaded e.g. 15 Nov 2021) or ALL_GNPS_NO_PROPOGATED.mgf (1 Nov 2022)
- Spectrum metadata with compound names/formulas for InChIKey resolution (via PubChem lookup through matchms extras)
- Train/test split manifest file recording the structure-disjoint partition

## Outputs

- Structure-disjoint train and test spectrum sets keyed on InChIKey
- Data-split manifest file shipped with the trained models
- Trained MS2Deepscore, Spec2Vec, and random forest model files trained only on the training partition

## How to apply

Download the GNPS library (e.g. ALL_GNPS.mgf) and clean it with matchms: remove spectra with fewer than 3 peaks, remove peaks above 1000 Da and peaks below 0.1% of the highest peak intensity, and normalize intensities. Create structure-disjoint splits keyed on InChIKey (fetching missing InChIKeys via matchms extras by searching compound name and molecular formula on PubChem) so the analogue search test set contains no 2D structures present in the training set, and record the split in a manifest file. Then train MS2Deepscore (supervised siamese model on spectrum pairs labeled by chemical structural similarity), Spec2Vec (unsupervised, 30 epochs, peaks binned to 2 decimals), and the random forest re-ranker (250 estimators, max depth 5, MSE loss) strictly on the training partition. The rationale is that spectral similarity between spectra of the same compound would let models memorize test structures, inflating benchmark performance.

## Related tools

- **matchms** (Cleaning the GNPS library (peak filtering, intensity normalization, metadata harmonization) and adding missing InChIKeys via PubChem lookup before splitting) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Supervised siamese model trained on the training partition of spectrum pairs labeled by chemical structural similarity) — https://github.com/iomega/ms2query
- **Spec2Vec** (Unsupervised embedding model trained (30 epochs, peaks binned to 2 decimals) on the training partition only) — https://github.com/iomega/ms2query
- **MS2Query** (Hosts the random forest re-ranker (250 estimators, max depth 5, MSE loss) trained on the training partition and packages the models with the split manifest) — https://github.com/iomega/ms2query
- **GNPS** (Source of the annotated spectral library (ALL_GNPS.mgf) used to build the splits)

## Examples

```
ms2query --library ./library_folder --download --ionmode positive  # then retrain with structure-disjoint splits: python create_split.py --library ALL_GNPS.mgf --key inchikey --manifest split_manifest.csv
```

## Evaluation signals

- The analogue search test set contains no 2D structures (InChIKey first blocks) present in the training set — verify by intersecting InChIKey sets of both partitions.
- None of the benchmarking test spectra appear in the training data of MS2Deepscore, Spec2Vec, or the MS2Query random forest re-ranker.
- The split manifest is saved alongside the trained model files (MS2Deepscore, Spec2Vec, random forest) so the partition is reproducible.
- Library cleaning invariants hold: no spectrum with fewer than 3 peaks, no peaks above 1000 Da, no peaks below 0.1% of the highest peak intensity, intensities normalized.
- Benchmark performance on the structure-disjoint test set (MS2Query outperforming cosine, modified cosine, and MS2Deepscore) is not inflated relative to expectations for unseen structures.

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the structure-disjoint benchmarking test set.
- Missing InChIKeys must be recovered by searching compound name and molecular formula on PubChem; failures leave spectra unassignable to a structural split key.
- Details on some training choices (e.g. motivation for selecting the top 100 highest scoring spectra for training the re-ranker) are only in Supplementary Note 5 and not fully documented in the main text.

## Evidence

- [results] Training used structure-disjoint splits: the analogue search test set contained no 2D structures present in the training set: [evidence span withheld — anchor status: not_found]
- [discussion] Test spectra excluded from all model training: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [discussion] InChIKey completion via PubChem: "A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem."
- [discussion] Peak filtering and normalization during library cleaning: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [readme] README: pretrained library and benchmark claim: [evidence span withheld — anchor status: external_doc]
- [readme] README: random forest re-ranking of top 2000 MS2Deepscore candidates: "The top 2000 spectra with the highest MS2Deepscore are selected."
