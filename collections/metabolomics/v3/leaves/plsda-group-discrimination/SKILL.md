---
name: plsda-group-discrimination
description: Use when an annotated lipidomics feature table (many features, fewer
  samples/groups) must be separated into known group labels — e.g. antimicrobial-resistant
  vs parent strains — and the discriminating features must be ranked or visualized.
when_to_use_negative:
- Input is raw IM-MS spectra rather than an annotated feature table — identification/prediction
  modules (CCS, retention time) are the wrong path for this skill
- Group labels are unknown and the goal is purely exploratory variance structure —
  unsupervised PCA alone is appropriate; PLS-DA would impose labels
- The dataset has few features relative to samples, where the dimensionality problem
  motivating multivariate projection does not apply
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LiPydomics
  role: Provides the stats module (PLS-DA projections, Pearson correlation) and plotting
    module (Log2 fold-change heat maps) operating on a lipydomics Dataset
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Backend used by the lipydomics stats module for multivariate analyses including
    PLS-DA and PCA
- name: SciPy
  role: Correlation functions backend for computing Pearson correlations between groups
    in the stats module
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:2ec068d4586f83651921af798ee4ab276493557ca7439a575a59c8eee563cb3c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# plsda-group-discrimination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform PLS-DA supervised multivariate projection with Pearson correlation in the lipydomics stats module to discriminate strain groups (e.g. daptomycin-resistant Dap2 vs parent Par MRSA) in HILIC-IM-MS lipidomics feature tables, then characterize discriminating features via Log2(fold-change) heat maps.

## When to use

Use when an annotated lipidomics feature table (many features, fewer samples/groups) must be separated into known group labels — e.g. antimicrobial-resistant vs parent strains — and the discriminating features must be ranked or visualized.

## When NOT to use

- Input is raw IM-MS spectra rather than an annotated feature table — identification/prediction modules (CCS, retention time) are the wrong path for this skill
- Group labels are unknown and the goal is purely exploratory variance structure — unsupervised PCA alone is appropriate; PLS-DA would impose labels
- The dataset has few features relative to samples, where the dimensionality problem motivating multivariate projection does not apply

## Inputs

- Annotated lipidomics feature table (e.g. 3647 features, four replicates per strain: Par, Dap2, Dal2, Van4) loadable into a lipydomics Dataset
- Known group/strain labels for supervised PLS-DA
- Group pair of interest for correlation and fold-change (e.g. Dap2 vs Par)

## Outputs

- PLS-DA projection figures exported as PNG images
- Pearson correlation statistics for the Dap2-vs-Par comparison
- Log2(fold-change) heat map and exported fold_change_heatmap_table.csv

## How to apply

Load the annotated feature table (e.g. 3647 features with four replicates each of Par, Dap2, Dal2, Van4 MRSA strains) into a lipydomics Dataset via the data module, then compute PLS-DA projections and Pearson correlations (SciPy backend) comparing Dap2 vs Par using the stats module. This complements an unsupervised PCA step — verify first that three components capture ~90% of the variance with visible strain-cluster separation before trusting supervised projections. Next compute Log2(fold-change) of Dap2 relative to Par for all features and render heat maps with the lipydomics plotting module. Export projection figures as PNG images and the fold-change statistics as fold_change_heatmap_table.csv. Validation hinges on the multivariate projection reproducing the biology: the heatmap table must show decreased DGDG, increased LysylPG, and increased 15–21 carbon fatty acids in Dap2 relative to Par.

## Related tools

- **LiPydomics** (Provides the stats module (PLS-DA projections, Pearson correlation) and plotting module (Log2 fold-change heat maps) operating on a lipydomics Dataset) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Backend used by the lipydomics stats module for multivariate analyses including PLS-DA and PCA)
- **SciPy** (Correlation functions backend for computing Pearson correlations between groups in the stats module)

## Examples

```
from lipydomics.data import Dataset; from lipydomics.stats import plsda, pearson; dataset = Dataset('MRSA_features.csv'); plsda(dataset, ('Par', 'Dap2')); pearson(dataset, 'Dap2', 'Par')
```

## Evaluation signals

- PCA prerequisite check: three components capture approximately 90% of the variance with visible separation of strain clusters before interpreting PLS-DA
- fold_change_heatmap_table.csv reproduces the reported Dap2-vs-Par pattern of decreased DGDG, increased LysylPG, and increased 15–21 carbon fatty acids
- PLS-DA projections show discrimination between the labeled strain groups in the exported PNG figures
- The full analysis is reproducible in a small scripted workflow (~19 lines of Python) on minimally processed data
- Computed statistics are stored on the lipydomics Dataset instance, matching the package's design of persisting stats with the data

## Limitations

- HILIC-IM-MS with this workflow generally supports Level 3 lipid identifications (lipid class/subclass and fatty acid sum composition), so discriminating features may not be fully resolved molecular species
- The dimensionality of lipidomics data sets (many features, fewer samples) makes interpretation difficult; supervised PLS-DA results should be checked against the unsupervised PCA structure
- Statistical functions assume an annotated feature table; prior databases/pipelines covering mainly mammalian lipids with limited bacterial lipid coverage can bias feature interpretation

## Evidence

- [other] PLS-DA with correlations: [evidence span withheld — anchor status: not_found]
- [abstract] stats module scope: [evidence span withheld — anchor status: not_found]
