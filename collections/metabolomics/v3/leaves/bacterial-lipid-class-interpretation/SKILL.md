---
name: bacterial-lipid-class-interpretation
description: Use when an annotated bacterial lipidomics feature table (multiple strains/conditions,
  replicate structure) needs class-level biological interpretation — e.g. verifying
  that resistance (Dap2 vs Par MRSA) reproduces reported DGDG/LysylPG/fatty-acid-chain-length
  shifts.
when_to_use_negative:
- Input is raw IM-MS data rather than a minimally processed, annotated feature table
  — the workflow assumes peak picking/annotation has already been done
- The biological question is lipid identification or CCS/retention-time prediction
  rather than class-level differential interpretation between strains
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: LiPydomics
  role: Provides the data, stats, and plotting modules used to build the Dataset,
    compute PCA/PLS-DA/Pearson correlation/Log2(fold-change), and render class-change
    heat maps
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Backend for the lipydomics stats module's PCA computation on the Dataset
- name: SciPy
  role: Provides correlation functions used by the lipydomics stats module for Pearson
    correlations between strain contrasts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:81d72bde68bf3869df705d7d3a0817265306a33219567a6e9d25268b8cdbff2d
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bacterial-lipid-class-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets strain- or treatment-level bacterial lipidomic differences at the lipid-class level by combining multivariate separation (PCA/PLS-DA) with per-feature Log2(fold-change) heat maps, translating feature-level statistics into biologically meaningful class shifts (e.g. decreased DGDG, increased LysylPG) as demonstrated for daptomycin-resistant Staphylococcus aureus strains with LiPydomics.

## When to use

Use when an annotated bacterial lipidomics feature table (multiple strains/conditions, replicate structure) needs class-level biological interpretation — e.g. verifying that resistance (Dap2 vs Par MRSA) reproduces reported DGDG/LysylPG/fatty-acid-chain-length shifts.

## When NOT to use

- Input is raw IM-MS data rather than a minimally processed, annotated feature table — the workflow assumes peak picking/annotation has already been done
- The biological question is lipid identification or CCS/retention-time prediction rather than class-level differential interpretation between strains

## Inputs

- Annotated lipidomics feature table with replicate-grouped samples (e.g. 3647 features, four replicates each of Par, Dap2, Dal2, Van4 MRSA strains)
- Sample-group metadata defining parent vs resistant strain comparisons

## Outputs

- PCA/PLS-DA projection figures exported as PNG images
- fold_change_heatmap_table.csv with per-feature Log2(fold-change) statistics
- Heat map visualizations of lipid class changes (DGDG, LysylPG, fatty acid carbon length distributions)

## How to apply

Load the annotated feature table (e.g. 3647 MRSA features, four replicates each of Par, Dap2, Dal2, Van4) into a lipydomics Dataset via the data module, then compute three-component PCA with the stats module (scikit-learn backend) and confirm the components capture roughly 90% of variance with strain cluster separation before interpreting class changes. Compute PLS-DA projections and Pearson correlations (SciPy backend) for the resistant-vs-parent contrast, then calculate Log2(fold-change) of the resistant strain relative to the parent for all features and render heat maps with the plotting module. Export the projection figures (PNG) and the fold-change statistics (fold_change_heatmap_table.csv), and judge correctness by whether the heatmap table reproduces the reported class pattern: decreased DGDG, increased LysylPG, and increased 15–21 carbon fatty acids in Dap2 vs Par. The rationale is that HILIC-IM-MS annotation supports Level 3 identifications (lipid class/subclass and fatty acid sum composition), so class-level aggregation of feature fold-changes is the appropriate granularity for biological interpretation.

## Related tools

- **LiPydomics** (Provides the data, stats, and plotting modules used to build the Dataset, compute PCA/PLS-DA/Pearson correlation/Log2(fold-change), and render class-change heat maps) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Backend for the lipydomics stats module's PCA computation on the Dataset)
- **SciPy** (Provides correlation functions used by the lipydomics stats module for Pearson correlations between strain contrasts)

## Examples

```
from lipydomics.data import Dataset; from lipydomics.stats import pca, plsda, log2_fold_change; ds = Dataset('mrsa_features.csv'); pca(ds, 3); log2_fold_change(ds, 'Dap2', 'Par')
```

## Evaluation signals

- PCA built on three components captures approximately 90% of the variance and shows separation of strain clusters (Par, Dap2, Dal2, Van4)
- fold_change_heatmap_table.csv reproduces the reported Dap2-vs-Par pattern: decreased DGDG, increased LysylPG, and increased 15–21 carbon fatty acids
- PCA/PLS-DA projection figures are exported as PNG images without errors from the plotting module
- Statistical values stored on the lipydomics Dataset match those recomputed via SciPy Pearson correlation functions
- The complete reproducible analysis stays minimal (the reference reanalysis used only 19 lines of Python code)

## Limitations

- HILIC-IM-MS annotation generally supports only Level 3 identifications (lipid class/subclass and fatty acid sum composition), so class-level interpretation cannot resolve individual molecular species
- Bacterial lipid coverage in reference databases may be limited — previous CCS databases mostly cover mammalian lipid species and have limited coverage of bacterial lipids
- Interpretation depends on the quality of the upstream annotated feature table; minimally processed data is assumed

## Evidence

- [other] Reanalysis of MRSA data with LiPydomics reproduced the key findings of the previous report using only 19 lines of Python code on minimally processed data.: [evidence span withheld — anchor status: not_found]
- [abstract] LiPydomics enables stats, plotting, identification, and interactive analysis modules for HILIC-IM-MS lipidomics data.: [evidence span withheld — anchor status: not_found]
- [intro] HILIC-IM-MS supports Level 3 lipid identifications suitable for class-level interpretation.: "this method generally allows Level 3 lipid identifications (lipid class/subclass and fatty acid sum composition)"
- [intro] Bacterial lipid coverage in prior databases was limited, motivating class-level bacterial interpretation with this package.: "previous databases mostly cover mammalian lipid species, have limited coverage of bacterial lipids, and have no built-in statistical functions"
- [readme] The README confirms lipydomics performs standard lipidomics analysis protocols storing datasets and computed statistics.: "The `data` module is responsible for storing lipidomics datasets, along with their associated metadata and computed statistics."
