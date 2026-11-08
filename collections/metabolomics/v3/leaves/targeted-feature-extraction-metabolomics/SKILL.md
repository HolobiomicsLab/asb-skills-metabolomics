---
name: targeted-feature-extraction-metabolomics
description: Use when an untargeted LC-MS/MS feature table (post feature detection/linking)
  contains consensus features with missing intensities across samples and quantitative
  completeness is needed for statistics or downstream analysis.
when_to_use_negative:
- The dataset already has complete intensities (no consensus features with missing
  values) — re-quantification adds compute for nothing.
- Input is raw vendor data not yet converted/centroided (e.g. Thermo .raw or Agilent/Bruker
  files needing proteowizard conversion to mzML).
- The goal is pure untargeted de novo detection rather than gap-filling of detected
  consensus features; run pre-processing first.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: Provides FeatureFinderMetaboIdent for targeted extraction plus MetaboliteAdductDecharger,
    IDMapper, and FeatureLinkerUnlabeledKD for the post-requantification re-annotation
    steps
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: pyOpenMS
  role: Python bindings used to drive the re-quantification notebook 3_Requantification(optional).ipynb
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow.git
- name: Snakemake
  role: Implements the optional re-quantification step as workflow rules toggled TRUE/FALSE
    in config.yaml
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:03a7176ea71915441a273aed061a5e849ecf6f331c7ddaf2cd3953d1228bfab5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Targeted feature extraction for metabolomics re-quantification

## Summary

Recovers missing feature intensities in untargeted metabolomics by building a target library from consensus features that have missing values and running OpenMS FeatureFinderMetaboIdent targeted extraction, replacing the entire feature matrix intensities rather than only the gaps.

## When to use

Use when an untargeted LC-MS/MS feature table (post feature detection/linking) contains consensus features with missing intensities across samples and quantitative completeness is needed for statistics or downstream analysis.

## When NOT to use

- The dataset already has complete intensities (no consensus features with missing values) — re-quantification adds compute for nothing.
- Input is raw vendor data not yet converted/centroided (e.g. Thermo .raw or Agilent/Bruker files needing proteowizard conversion to mzML).
- The goal is pure untargeted de novo detection rather than gap-filling of detected consensus features; run pre-processing first.

## Inputs

- Centroided mzML files in data/mzML/
- Pre-processed consensus map (consensusXML) from feature detection/linking
- config.yaml with user parameters (minfrac threshold, ionization mode)
- config/dataset.tsv listing sample filenames

## Outputs

- requantified_feature_table.consensusXML with filled intensities, adduct annotations, and re-linked features
- Re-quantified feature matrix (TSV) with targeted-extraction intensities across all samples

## How to apply

Build a target library from the pre-processed consensus map, selecting every consensus feature with at least one missing intensity value across samples; optionally apply the user-defined minfrac threshold (set in config.yaml) to remove consensus features with too many missing values. Run OpenMS FeatureFinderMetaboIdent targeted extraction on every centroided mzML file in data/mzML/ against this library, then replace ALL intensity values in the feature matrix with the targeted extraction results — not only the previously missing ones — for consistent quantification. Re-run MetaboliteAdductDecharger to re-annotate adducts, IDMapper to transfer identifications, and FeatureLinkerUnlabeledKD to produce the final re-quantified consensus map. In UmetaFlow this is implemented both as Snakemake rules (Snakemake >=7.14.0) and as the optional notebook 3_Requantification(optional).ipynb using pyOpenMS.

## Related tools

- **OpenMS** (Provides FeatureFinderMetaboIdent for targeted extraction plus MetaboliteAdductDecharger, IDMapper, and FeatureLinkerUnlabeledKD for the post-requantification re-annotation steps) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **pyOpenMS** (Python bindings used to drive the re-quantification notebook 3_Requantification(optional).ipynb) — https://github.com/biosustain/pyOpenMS_UmetaFlow.git
- **Snakemake** (Implements the optional re-quantification step as workflow rules toggled TRUE/FALSE in config.yaml) — https://github.com/biosustain/snakemake_UmetaFlow.git

## Examples

```
jupyter notebook 3_Requantification(optional).ipynb
```

## Evaluation signals

- requantified_feature_table.consensusXML shows filled intensities for targeted features across all samples
- Adduct annotations are re-established (MetaboliteAdductDecharger re-run on re-quantified features)
- Feature linking re-established via FeatureLinkerUnlabeledKD into the final re-quantified consensus map
- Consensus features exceeding the minfrac missing-value threshold are removed
- Intensity column values match targeted-extraction results (whole matrix replaced, not only previously missing cells)

## Limitations

- minfrac threshold is optional and user-defined; misconfiguration can either drop valid low-missingness features or retain features with too many missing values.
- Re-quantification replaces all intensity values with targeted-extraction results, so if targeted extraction fails for a feature, previous untargeted values are not retained.
- Re-quantification is an optional step and disabled by writing FALSE for the rule in config.yaml; skipping it leaves missing values in the feature table.

## Evidence

- [readme] Re-quantification (optional): Re-quantify all raw files to avoid missing values resulted by the pre-processing workflow for statistical analysis and data exploration: [evidence span withheld — anchor status: not_found]
- [readme] Optional minfrac step allows removal of consensus features with too many missing values: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [intro] OpenMS 3.0 tools list including re-quantification: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
