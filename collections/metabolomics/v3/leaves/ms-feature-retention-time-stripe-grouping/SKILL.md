---
name: ms-feature-retention-time-stripe-grouping
description: Use when an NP Analyst community contains multiple strong-Activity-Score
  MS features that cluster at the same retention time in the community view's RT vs
  m/z plot, suggesting adducts/fragments of one molecule rather than distinct compounds.
when_to_use_negative:
- Input is raw unprocessed LC-MS data rather than a processed, replicate-compared
  feature table from MZmine 2 or GNPS — unpreprocessed runs increase candidate bioactive
  MS features with junk peaks
- The community has a single clear individual bioactive MS feature — direct feature
  prioritization is more appropriate than stripe grouping
- No bioactivity scores are available, so Activity/Cluster Score similarity cannot
  be used to decide whether co-retaining features belong to one molecule
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: NP Analyst
  role: Provides the community view with the retention time vs m/z ratio plot in which
    stripes of active MS features at the same retention time are identified; scores
    each MS feature independently
  repo: https://github.com/liningtonlab/npanalyst
- name: MZmine 2
  role: Upstream processing of MS data into the unique-feature peak lists whose RT
    and m/z values are grouped into stripes
- name: pandas
  role: Grouping the community feature table by RT cluster and writing the dereplication
    assignments to CSV
- name: GNPS
  role: Alternative upstream source of network/peak-list files for NP Analyst input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:db6101276ec10cd9483ad351430b88d1a04071ed328d40b50a3bacc275e430af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  - pandas
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ms-feature-retention-time-stripe-grouping

## Summary

A dereplication step for NP Analyst communities: group co-eluting high-Activity-Score MS features into retention-time 'stripes' and interpret each stripe as adducts, fragments, or isotopes of a single parent molecule.

## When to use

Use when an NP Analyst community contains multiple strong-Activity-Score MS features that cluster at the same retention time in the community view's RT vs m/z plot, suggesting adducts/fragments of one molecule rather than distinct compounds.

## When NOT to use

- Input is raw unprocessed LC-MS data rather than a processed, replicate-compared feature table from MZmine 2 or GNPS — unpreprocessed runs increase candidate bioactive MS features with junk peaks
- The community has a single clear individual bioactive MS feature — direct feature prioritization is more appropriate than stripe grouping
- No bioactivity scores are available, so Activity/Cluster Score similarity cannot be used to decide whether co-retaining features belong to one molecule

## Inputs

- NP Analyst community feature table (CSV) with m/z, retention time, Activity Score, and Cluster Score per MS feature
- Processed MS peak lists from MZmine 2 or GNPS graphml output that fed the NP Analyst run
- Bioactivity (Activity/Cluster Score) data from the underlying bioassay screen

## Outputs

- Feature-to-compound assignment table mapping each MS feature to its retention-time stripe and parent molecule (e.g., community15_dereplication.csv)
- Per-stripe rationale (adduct/fragment/isotope assignments, RT clusters, m/z offset analysis)

## How to apply

Load the community's feature table (m/z, retention time, Activity Score, Cluster Score for the strong-Activity-Score features) and group features by retention time to identify co-retaining stripes — e.g., in the community 15 case, the six strong features fell into three RT sets at 2.64, 3.42, and 3.97 min. Apply the NP Analyst convention that all MS features from one bioactive molecule share similar Activity and Cluster Scores but different m/z ratios, so a stripe with near-identical Activity/Cluster Scores but distinct m/z values is a single molecule's ion family. Within each stripe, compare m/z differences and offsets to classify features as adducts, fragments, or isotopes of one parent. Confirm grouping with extracted-ion-chromatogram (EIC) peak-shape co-elution, then assign each stripe to a dereplicated compound — in community 15, the 2.64 min set to collismycin A, the 3.42 min set to SF2738D, and the 3.97 min set to the collismycin B isobar. Perform this only after upstream preprocessing (replicate comparison and an appropriate signal-intensity cutoff), since raw mass lists add junk features that can fragment or inflate stripes.

## Related tools

- **NP Analyst** (Provides the community view with the retention time vs m/z ratio plot in which stripes of active MS features at the same retention time are identified; scores each MS feature independently) — https://github.com/liningtonlab/npanalyst
- **MZmine 2** (Upstream processing of MS data into the unique-feature peak lists whose RT and m/z values are grouped into stripes)
- **pandas** (Grouping the community feature table by RT cluster and writing the dereplication assignments to CSV)
- **GNPS** (Alternative upstream source of network/peak-list files for NP Analyst input)

## Examples

```
import pandas as pd; df = pd.read_csv('community15_features.csv'); df.groupby(df['retention_time'].round(2))['mz'].apply(list).to_csv('community15_dereplication.csv')
```

## Evaluation signals

- The output grouping covers every strong-Activity-Score feature in the community (e.g., all six community-15 features) with no unassigned features
- The number of retention-time stripes matches the number of dereplicated parent molecules (three stripes at 2.64, 3.42, and 3.97 min for three compounds in community 15)
- Within each stripe, Activity and Cluster Scores are similar across members while m/z values differ, per the NP Analyst scoring convention
- m/z differences within a stripe are consistent with recognized adduct/fragment/isotope offsets of a single parent molecule
- EIC peak-shape analysis confirms co-elution of features assigned to the same stripe, and assignments match the article's reported dereplication (collismycin A, SF2738D, collismycin B isobar)

## Limitations

- NP Analyst scores each MS feature independently, so stripe interpretation (adducts vs fragments vs separate molecules sharing an RT window) is a manual judgment, not an automated output
- Replicate comparison to reduce noise peaks is only possible 'where possible'; without replicate data, junk features can contaminate stripes
- Low signal-intensity cutoffs dramatically increase network complexity without adding biologically informative MS features, making stripes harder to resolve

## Evidence

- [discussion] identify sets of active MS features with the same retention time in the retention time vs m/z ratio plot in the community view: "identify sets of active MS features with the same retention time in the plot of retention time vs m/z ratio in the community view"
- [discussion] Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [discussion] In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same: "In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same"
- [discussion] Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets and is strongly recommended where possible: [evidence span withheld — anchor status: not_found]
- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions).: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
