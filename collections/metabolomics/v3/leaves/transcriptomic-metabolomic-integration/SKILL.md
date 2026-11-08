---
name: transcriptomic-metabolomic-integration
description: Use when a paired transcriptomic-metabolomic dataset is available and
  the goal is de novo, bait-free reconstruction or hypothesis generation for specialized-metabolite
  biosynthetic pathways.
when_to_use_negative:
- 'The analysis requires MS/MS-based annotation: MEANtools does not currently support
  loading MS/MS data or mass spectral library/networking-based annotation approaches.'
- 'The pathway of interest involves multi-substrate reactions or stereoisomeric conversions:
  RetroRules-based rules are designed for single-substrate reactions only, and stereoisomers
  are not captured by mass spectrometric data.'
- Prior knowledge of a bait compound or enzyme is already available and a bait-driven
  co-expression approach is sufficient for the goal.
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: MEANtools
  role: 'Core pipeline: MAD rescaling, MR-based correlation, ClusterONE clustering,
    FC merging, adduct-based structure prediction via LOTUS, and RetroRules-based
    reaction anticipation'
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of general reaction rules and rule-enzyme associations (strict dataset
    = experimentally validated entries only) used to predict putative reactions
  repo: https://zenodo.org/records/15697913
- name: LOTUS
  role: Metabolite structure database queried (SQLite, downloaded 10/10/2023) at 20
    ppm tolerance to assign molecular structures to mass features
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters transcripts and mass features with their exponential-decay MR edge
    weights into overlapping feature clusters (FCs)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:937f3bd096c09ccbc3253e947c955b8acf94443b4a3e36efd063d3e4f51e5525
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - ClusterONE
  - LOTUS
  - MEANtools
  - RetroRules
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# transcriptomic-metabolomic-integration

## Summary

Unsupervised integration of paired transcriptomic and metabolomic data with MEANtools to connect correlated enzyme-coding transcripts and mass features and predict biosynthetic reaction steps using RetroRules reaction rules and LOTUS structure annotations.

## When to use

Use when a paired transcriptomic-metabolomic dataset is available and the goal is de novo, bait-free reconstruction or hypothesis generation for specialized-metabolite biosynthetic pathways.

## When NOT to use

- The analysis requires MS/MS-based annotation: MEANtools does not currently support loading MS/MS data or mass spectral library/networking-based annotation approaches.
- The pathway of interest involves multi-substrate reactions or stereoisomeric conversions: RetroRules-based rules are designed for single-substrate reactions only, and stereoisomers are not captured by mass spectrometric data.
- Prior knowledge of a bait compound or enzyme is already available and a bait-driven co-expression approach is sufficient for the goal.

## Inputs

- paired transcriptomic-metabolomic dataset (e.g., the Jeon 2020 tomato dataset)
- RetroRules reaction rules datasets (strict/medium/loose, combined CSV from Zenodo record 15697913)
- LOTUS metabolite database in SQLite format (from the meantools GitHub repository)
- list of adducts for mass-feature to molecular-structure assignment

## Outputs

- overlapping feature clusters (FCs) of transcripts and mass features
- merged FCs sharing common mass features (via merge_clusters.py)
- predicted reaction steps linking transcript-correlated mass features (Ni->j networks)
- candidate enzyme assignments (e.g., Solyc12g100250.1) with correlation values and p-values
- comparison table of predicted vs. published pathway steps (e.g., pathway_steps_comparison.csv)

## How to apply

Rescale the paired transcript and mass-feature data with Median Absolute Deviation (MAD), compute Pearson correlations between transcripts and mass features, convert them to mutual ranks (MR), and apply an exponential decay function to obtain edge weights. Annotate each mass feature by identifying possible adducts and querying the LOTUS database (SQLite, downloaded 10/10/2023) at the default 20 ppm mass accuracy tolerance, using RetroRules general reaction rules for putative reactions. Cluster nodes and edge weights with ClusterONE to produce overlapping feature clusters (FCs), merge FCs sharing common mass features with merge_clusters.py, then run the reaction-anticipation step — excluding gene-metabolite pairs with edge score below 0.01 and removing cycles by eliminating the weakest enzyme-metabolite correlation edge among same-distance metabolites. Use the strict RetroRules rule set (experimentally validated entries only) whenever possible, since the loose set retains spurious reaction-Pfam links after the Pfam cutoff of 6; validate predictions against a published pathway (e.g., the seven-step tomato falcarindiol pathway from Jeon 2020, where five of seven steps were anticipated and Solyc12g100250.1 was predicted for the crepenynic-acid-to-dehydrocrepenynic-acid conversion, r=0.744, p=1.106E-10).

## Related tools

- **MEANtools** (Core pipeline: MAD rescaling, MR-based correlation, ClusterONE clustering, FC merging, adduct-based structure prediction via LOTUS, and RetroRules-based reaction anticipation) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of general reaction rules and rule-enzyme associations (strict dataset = experimentally validated entries only) used to predict putative reactions) — https://zenodo.org/records/15697913
- **LOTUS** (Metabolite structure database queried (SQLite, downloaded 10/10/2023) at 20 ppm tolerance to assign molecular structures to mass features) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters transcripts and mass features with their exponential-decay MR edge weights into overlapping feature clusters (FCs))

## Examples

```
python merge_clusters.py --clusters cluster1.txt cluster2.txt; python reaction_prediction.py --rules Combined_small_middle_big_datasets.csv --ruleset strict --edge_threshold 0.01
```

## Evaluation signals

- Benchmark against a published pathway: on the Jeon 2020 tomato dataset MEANtools anticipated five of seven falcarindiol pathway steps under strict settings.
- Known-step recovery check: the enzyme for the octadecadiene-diynoic acid to metabolite 6 conversion (Solyc12g100250.1, correlation 0.744, p=1.106E-10) must match the enzyme reported by Jeon and colleagues (2020).
- Ruleset sanity check: predictions made with the loose RetroRules set should be inspected for spurious Pfam associations (e.g., AminoTran_1_2, Glyco_transf_20) that survive the Pfam cutoff of 6; prefer strict-set results.
- Graph invariants: predicted reaction networks (Ni->j) must be acyclic — cycles among metabolites at the same reaction distance must have been resolved by removing the weakest enzyme-metabolite correlation edge.
- Edge-score threshold check: no gene-metabolite pair with edge score below 0.01 should appear in the Ni->j networks.

## Limitations

- Mass spectrometric data does not capture stereoisomers, which hindered reconstruction of steps such as phenylalanine to p-coumaroyl-CoA.
- RetroRules-based rules are designed for single-substrate reactions only, so two-substrate conversions (e.g., p-coumaroyltyramine from tyrosine and p-coumaroyl-CoA) cannot be reconstructed.
- FC size may be insufficient for forming a complete biosynthetic pathway, and changing the ClusterONE inbuilt parameter alters clustering of both mass features and transcripts.
- Pearson-based correlation integration alone yields many false-positive associations and thresholding is difficult; the MR/exponential-decay weighting mitigates but does not remove this.
- The mass spectrometer resolution directly impacts the precision of detected m/z values and confidence of adduct-based matching; the default 20 ppm tolerance balances sensitivity and specificity.
- No MS/MS data loading or spectral library/networking-based annotation is currently supported.

## Evidence

- [intro] MEANtools implements a mutual rank (MR)-based [14] correlation approach to capture mass features that are highly correlated with biosynthetic genes.: "MEANtools implements a mutual rank (MR)-based [14] correlation approach to capture mass features that are highly correlated with biosynthetic genes."
- [discussion] correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function that converts continuous MR values to numbers: "correlation values between mass features and transcripts are first converted to MRs [49], which are then subjected to an exponential decay function that converts continuous MR values to numbers"
- [discussion] Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs.: "Both node types and edge weights are further subjected to clustering using ClusterONE [47], which results in multiple overlapping FCs."
- [discussion] Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks.: "Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks."
- [discussion] By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity.: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [abstract] MEANtools correctly anticipated five out of seven steps of the characterized pathway: "MEANtools correctly anticipated five out of seven steps of the characterized pathway"
- [discussion] The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository.: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
