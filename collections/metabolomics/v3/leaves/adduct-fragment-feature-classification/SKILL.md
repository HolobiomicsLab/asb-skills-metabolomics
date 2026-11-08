---
name: adduct-fragment-feature-classification
description: Use when an NP Analyst community contains multiple strong Activity-Score
  MS features with similar Activity and Cluster Scores but different m/z values, and
  you need to determine how many distinct bioactive molecules they represent.
when_to_use_negative:
- The community has a single clear individual bioactive MS feature rather than a multi-feature
  stripe needing manual interpretation
- Features lack comparable Activity and Cluster Scores, so m/z differences cannot
  be attributed to adducts/fragments of one molecule
- Input is raw mass-list data that has not been processed into unique MS features
  (e.g. via MZmine 2 or GNPS peak lists)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: NP Analyst
  role: Provides the community feature table, Activity/Cluster Scores, and the retention
    time vs m/z community view used to identify feature stripes
  repo: https://github.com/liningtonlab/npanalyst
- name: pandas
  role: Loads and groups the community-15 feature table by retention time and writes
    the dereplication CSV output
- name: MZmine 2
  role: Source of the processed MS feature peak lists that feed NP Analyst communities
    prior to stripe classification
- name: GNPS
  role: Source of network files processed into MS features for NP Analyst input prior
    to dereplication
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:f237d722e52b13079010e31e7ae90c959280d269b067b4c8558997f38d18c57d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  - pandas
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-fragment-feature-classification

## Summary

Groups high-scoring MS features from an NP Analyst community by retention time into feature stripes, then classifies within-stripe m/z differences as adducts, fragments, or isotopes of a single parent molecule for dereplication.

## When to use

Use when an NP Analyst community contains multiple strong Activity-Score MS features with similar Activity and Cluster Scores but different m/z values, and you need to determine how many distinct bioactive molecules they represent.

## When NOT to use

- The community has a single clear individual bioactive MS feature rather than a multi-feature stripe needing manual interpretation
- Features lack comparable Activity and Cluster Scores, so m/z differences cannot be attributed to adducts/fragments of one molecule
- Input is raw mass-list data that has not been processed into unique MS features (e.g. via MZmine 2 or GNPS peak lists)

## Inputs

- NP Analyst community feature table (CSV) with m/z, retention time, Activity Score, and Cluster Score per MS feature
- Retention-time vs m/z community view from NP Analyst (identifies sets of active MS features with the same retention time)
- Extracted-ion chromatogram (EIC) peak shapes for corroboration of co-elution

## Outputs

- Feature-to-compound assignment table (e.g. community15_dereplication.csv) mapping each strong-Activity-Score feature to a retention-time set, feature role (adduct/fragment/isotope), and dereplicated compound name

## How to apply

Load the community feature table (m/z, retention time, Activity Score, Cluster Score for each strong-scoring feature, e.g. the six strong Activity-Score features in community 15 of the NP Analyst demonstration dataset). Group features by retention time to identify co-retaining feature stripes (in community 15 these cluster at 2.64, 3.42, and 3.97 min), optionally corroborated by EIC peak-shape grouping. Within each retention-time set, compare m/z differences and offsets between features and classify them as adducts, fragments, or isotopes of one parent molecule, following the NP Analyst convention that all MS features from one bioactive molecule share similar Activity and Cluster Scores but different m/z ratios because NP Analyst scores each feature independently. Assign each stripe to a dereplicated compound (e.g. 2.64 min to collismycin A, 3.42 min to SF2738D, 3.97 min to the collismycin B isobar) and write the feature-to-compound assignments with rationale to an output table such as community15_dereplication.csv. Validate that the output covers all input features across the expected number of retention-time sets with each set assigned to the expected compound.

## Related tools

- **NP Analyst** (Provides the community feature table, Activity/Cluster Scores, and the retention time vs m/z community view used to identify feature stripes) — https://github.com/liningtonlab/npanalyst
- **pandas** (Loads and groups the community-15 feature table by retention time and writes the dereplication CSV output)
- **MZmine 2** (Source of the processed MS feature peak lists that feed NP Analyst communities prior to stripe classification)
- **GNPS** (Source of network files processed into MS features for NP Analyst input prior to dereplication)

## Examples

```
import pandas as pd; feats = pd.read_csv('community15_features.csv'); stripes = feats.groupby('retention_time'); stripes.apply(classify_adducts_fragments).to_csv('community15_dereplication.csv')
```

## Evaluation signals

- Output CSV reproduces exactly three retention-time sets (2.64, 3.42, 3.97 min) covering all six strong Activity-Score features in community 15
- Each retention-time set is assigned to the expected compound: collismycin A (2.64 min), SF2738D (3.42 min), collismycin B isobar (3.97 min), matching the article's reported community 15 dereplication
- Features within a retention-time set share similar Activity and Cluster Scores while differing in m/z, consistent with the NP Analyst scoring convention
- EIC peak shapes corroborate co-retention grouping of features within each stripe

## Limitations

- NP Analyst scores each MS feature independently, so adduct/fragment/multiply-charged stripe assignment is a manual interpretation step, not an automated output
- Some communities lack clear individual bioactive MS features, requiring this retention-time stripe strategy instead of direct feature prioritization
- Low-resolution MS data limit confidence in m/z-difference-based classification of adducts, fragments, and isotopes

## Evidence

- [discussion] Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [discussion] Identify sets of active MS features with the same retention time in the community view: "identify sets of active MS features with the same retention time in the plot of retention time vs m/z ratio in the community view"
- [discussion] In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same retention time: "In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same"
- [abstract] We demonstrate the utility of this new tool for both the dereplication of known compounds and the discovery of novel bioactive natural products: "We demonstrate the utility of this new tool for both the dereplication of known compounds and the discovery of novel bioactive natural products using a challenging low-resolution antimicrobial"
- [readme] npanalyst --help should show all the features and usage: "`npanalyst --help` should show all the features and usage."
