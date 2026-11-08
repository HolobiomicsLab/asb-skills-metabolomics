---
name: msms-pairwise-similarity-computation
description: Use when you have MS/MS spectra (MGF files) from a spectral library or
  extract library and need pairwise similarity scores that reflect structural relatedness,
  e.g., to build a molecular network or find structural analogues beyond cosine-based
  matching.
when_to_use_negative:
- When a conventional, heuristic similarity is sufficient and fully interpretable
  by construction (e.g., Cosine or Modified Cosine for simple aligned-peak comparisons)
  — TransExION is a deep-learning model whose scores require the trained model checkpoint
- 'When you do not have a trained TransExION model and cannot train one: training
  additionally requires structural (Tanimoto) similarity matrices computed with ms_compute_structural_similarity.py
  from known structures'
- When inputs are not MS/MS peak lists (e.g., raw LC-MS/MS instrument files) — MGF
  conversion must happen first
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TransExION
  role: Transformer-based model whose ms_test_spectral_similarity.py script computes
    pairwise MS/MS spectral similarity scores; also provides post-hoc explanation
    heatmaps via ms_explain_spectral_similarity.py
  repo: https://github.com/banhdzui/TransExION
- name: ms_convert_mgf2h5py.py
  role: Converts input MGF spectral files into h5py format required for large-scale
    similarity computation
  repo: https://github.com/banhdzui/TransExION
- name: meRgeION
  role: Downstream tool that consumes the thresholded TransExION pairwise similarity
    matrix to build molecular networks of extract libraries
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:fa8f4ee9a87e7db934a9a4bdb319471dd31599c3bfdfc4be031516cf0ee739d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - TransExION
  - meRgeION
  - ms_convert_mgf2h5py.py
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# msms-pairwise-similarity-computation

## Summary

Compute all-pairs spectral similarity between MS/MS spectra using the pre-trained TransExION Transformer model, whose scores correlate with structural similarity and can feed downstream molecular networking (e.g., meRgeION) or spectral library analogue search.

## When to use

Use when you have MS/MS spectra (MGF files) from a spectral library or extract library and need pairwise similarity scores that reflect structural relatedness, e.g., to build a molecular network or find structural analogues beyond cosine-based matching.

## When NOT to use

- When a conventional, heuristic similarity is sufficient and fully interpretable by construction (e.g., Cosine or Modified Cosine for simple aligned-peak comparisons) — TransExION is a deep-learning model whose scores require the trained model checkpoint
- When you do not have a trained TransExION model and cannot train one: training additionally requires structural (Tanimoto) similarity matrices computed with ms_compute_structural_similarity.py from known structures
- When inputs are not MS/MS peak lists (e.g., raw LC-MS/MS instrument files) — MGF conversion must happen first

## Inputs

- MS/MS spectra in MGF format (e.g., GNPS_MassBank_test.mgf or experimental spectra from an extract library)
- Spectra converted to h5py format (.db files)
- Pre-trained TransExION model file (e.g., GNPS_MassBank.ms.model from Zenodo record 8175528)
- Optional spectrum pair-id file in h5py format (converted from CSV) to limit which pairs are scored

## Outputs

- CSV file of pairwise spectral similarity scores (e.g., GNPS_MassBank_result.csv)
- Similarity matrix that can be thresholded (e.g., edges > 0.6) and imported into meRgeION to build a molecular network
- Top analogue hits per query spectrum when searching against a spectral library

## How to apply

First convert MGF spectra to h5py format with ms_convert_mgf2h5py.py, since the library works with large-scale data in h5py format. Then run ms_test_spectral_similarity.py, passing the reference spectra (--db), query spectra (--query), and the pre-trained TransExION model (--model, e.g., from Zenodo record 8175528); if --pairs is left empty, every query spectrum is paired with every reference spectrum. Optionally restrict computation to specific spectrum pairs by supplying a pair-id file in h5py format (convertible from CSV via ms_covert_csv2h5py.py). The output CSV holds the pairwise TransExION similarity scores; for molecular networking (e.g., in meRgeION), filter edges above a score threshold such as 0.6, with no minimum matched-peak requirement since TransExION handles alignment internally via mass-difference-aware fragment associations.

## Related tools

- **TransExION** (Transformer-based model whose ms_test_spectral_similarity.py script computes pairwise MS/MS spectral similarity scores; also provides post-hoc explanation heatmaps via ms_explain_spectral_similarity.py) — https://github.com/banhdzui/TransExION
- **ms_convert_mgf2h5py.py** (Converts input MGF spectral files into h5py format required for large-scale similarity computation) — https://github.com/banhdzui/TransExION
- **meRgeION** (Downstream tool that consumes the thresholded TransExION pairwise similarity matrix to build molecular networks of extract libraries)

## Examples

```
python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv
```

## Evaluation signals

- The output CSV contains a similarity score for every query-reference pair (when --pairs is empty, all combinations are computed)
- Similarity scores should correlate with structural (Tanimoto) similarity; TransExION shows low prediction error and high stability across the entire structure similarity range
- In molecular networking use, thresholded edges (> 0.6) imported into meRgeION should recover sub-networks representing confidently-identified chemical families
- TransExION should outperform Cosine, Modified Cosine, Spec2Vec, and MS2DeepScore at retrieving structural analogues, especially at high Tanimoto distance cut-offs
- Paired post-hoc explanation heatmaps (ms_explain_spectral_similarity.py) should show query-reference product ion associations with coherent mass differences

## Limitations

- Requires a pre-trained model; the released model was trained on GNPS-derived MS/MS library data, so performance on spectral types far outside GNPS library training data is untested in this context
- TransExION similarity is a learned estimate, not a deterministic peak-alignment score, so it lacks the direct interpretability of cosine unless the post-hoc explanation module is run
- The README gives no documented quantitative performance thresholds or matched-peak criteria; downstream edge filtering thresholds (e.g., 0.6) come from the molecular-networking use case, not the scoring tool itself

## Evidence

- [readme] TransExION is a supervised learning method estimating spectral similarity between MS/MS spectra that are strongly correlated to their structural similarity.: "TransExION is a supervised learning method estimating ***spectral similarity*** between MS/MS spectra that are strongly correlated to their structural similarity."
- [readme] If the parameter --pairs is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity.: "If the parameter ```--pairs``` is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity."
- [discussion] Using the pair-wise spectral similarities computed by TransExION, we built a molecular network of bacteria extracts and retrieved sub-networks representing confidently-identified chemical families.: "Using the pair-wise spectral similarities computed by TransExION, we built a molecular network of bacteria extracts and retrieved sub-networks representing confidently-identified chemical families"
- [abstract] TransExION detects related fragments between MS/MS spectra through their mass difference: "TransExION detects related fragments between MS/MS spectra through their mass difference"
- [intro] The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered: "The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity"
