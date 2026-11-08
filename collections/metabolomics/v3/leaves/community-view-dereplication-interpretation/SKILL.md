---
name: community-view-dereplication-interpretation
description: Use when an NP Analyst community lacks a single dominant bioactive MS
  feature but contains multiple high Activity-Score features that co-retain, requiring
  RT-vs-m/z stripe grouping for dereplication.
when_to_use_negative:
- The community already contains a single clear individual bioactive MS feature that
  can be dereplicated directly without stripe grouping
- The input is raw instrument data rather than a scored NP Analyst feature table —
  scoring and community formation must occur first
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
tools:
- name: NP Analyst
  role: Generates the scored feature table and community-view scatter/network/community
    plots whose retention-time vs m/z plot is the basis for stripe grouping
  repo: https://github.com/liningtonlab/npanalyst
- name: pandas
  role: Loads the community-15 feature table and writes the reproduced grouping to
    community15_dereplication.csv
- name: MZmine 2
  role: Upstream producer of the processed MS feature peak lists that NP Analyst ingests
    for community scoring
- name: GNPS
  role: Upstream producer of network/feature files that NP Analyst accepts as MS input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:5147c5a0b8a0f75b668b75336dd2fd9748f67b6f285813f05f425b04e3ecb2f2
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# community-view-dereplication-interpretation

## Summary

Interprets NP Analyst community-view output by grouping scored MS features into retention-time 'stripes' so that adducts, fragments, and isotopes of one bioactive molecule are recognized as a single compound, enabling dereplication of communities like community 15 (collismycin A, SF2738D, collismycin B isobar).

## When to use

Use when an NP Analyst community lacks a single dominant bioactive MS feature but contains multiple high Activity-Score features that co-retain, requiring RT-vs-m/z stripe grouping for dereplication.

## When NOT to use

- The community already contains a single clear individual bioactive MS feature that can be dereplicated directly without stripe grouping
- The input is raw instrument data rather than a scored NP Analyst feature table — scoring and community formation must occur first

## Inputs

- NP Analyst community feature table with m/z, retention time, Activity Score, and Cluster Score per feature (processed from MZmine 2 or GNPS output)
- Retention time vs m/z community-view plot data
- Extracted-ion chromatogram (EIC) peak shapes for feature-shape grouping

## Outputs

- Feature-to-compound assignment table grouping each RT stripe to a dereplicated molecule (e.g. community15_dereplication.csv)
- Adduct/fragment/isotope classification per feature with m/z-offset rationale

## How to apply

Load the community's feature table (m/z, retention time, Activity Score, Cluster Score) from NP Analyst output processed from MZmine 2 or GNPS peak lists. In the community view's retention time vs m/z ratio plot, identify sets of active MS features sharing the same retention time — because NP Analyst scores each feature independently, all MS features from one bioactive molecule carry similar Activity and Cluster Scores but different m/z ratios. Within each RT stripe, compare m/z offsets to classify features as adducts, fragments, or isotopes of a single parent. Assign each stripe to a compound identity (e.g. in community 15: 2.64 min → collismycin A, 3.42 min → SF2738D, 3.97 min → collismycin B isobar) using EIC peak-shape grouping as supporting evidence, then export the feature-to-compound assignments (e.g. to CSV via pandas) for validation against the reported dereplication.

## Related tools

- **NP Analyst** (Generates the scored feature table and community-view scatter/network/community plots whose retention-time vs m/z plot is the basis for stripe grouping) — https://github.com/liningtonlab/npanalyst
- **pandas** (Loads the community-15 feature table and writes the reproduced grouping to community15_dereplication.csv)
- **MZmine 2** (Upstream producer of the processed MS feature peak lists that NP Analyst ingests for community scoring)
- **GNPS** (Upstream producer of network/feature files that NP Analyst accepts as MS input)

## Examples

```
import pandas as pd; ft = pd.read_csv('community15_features.csv'); ft['stripe'] = pd.cut(ft.rt, [2.5, 2.7, 3.5, 4.1], labels=['collismycin A','SF2738D','collismycin B isobar']); ft.to_csv('community15_dereplication.csv', index=False)
```

## Evaluation signals

- The output CSV contains exactly three RT stripes (2.64, 3.42, 3.97 min) covering all six strong-Activity-Score features
- Each stripe is assigned to the expected compound: collismycin A (2.64 min), SF2738D (3.42 min), collismycin B isobar (3.97 min)
- Features within one stripe share similar Activity and Cluster Scores but distinct m/z values, per the NP Analyst scoring convention
- Stripe membership is corroborated by matching EIC peak shapes across co-retaining features

## Limitations

- NP Analyst scores each MS feature independently, so stripe interpretation (adducts/fragments/multiply-charged) must be done manually and is subjective without standards
- Some communities lack clear individual bioactive MS features, making this RT-stripe strategy necessary but reliant on sufficient co-retaining features
- Low minimum signal-intensity cutoffs can inflate community complexity with uninformative features, obscuring true stripes

## Evidence

- [discussion] In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same: "In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same"
- [discussion] Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged): "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
- [discussion] The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
