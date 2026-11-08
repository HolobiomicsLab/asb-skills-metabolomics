---
name: consensus-feature-linking
description: Use when you have RT-aligned, centroided mzML-derived feature maps from
  multiple LC-MS/MS samples and need a single consensus feature table linking each
  feature's m/z, RT, adduct and per-sample intensities.
when_to_use_negative:
- Input is already a consensus feature table or TSV feature matrix (e.g. you are at
  the re-quantification or annotation stage instead)
- The dataset is raw vendor files (Thermo .raw, Agilent, Bruker) that have not yet
  been converted to centroided mzML and undergone feature detection and RT alignment
- Only a single sample map is available, since linking consensus feature groups across
  samples requires multiple input maps
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FeatureLinkerUnlabeledKD
  role: OpenMS 3.0 tool that links consensus feature groups across the aligned, adduct-annotated
    sample feature maps
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: UmetaFlow
  role: Snakemake/pyOpenMS workflow that orchestrates feature linking together with
    minfrac and blank/QC filtering and exports feature_table.tsv
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings used in the Jupyter-notebook implementation of the pre-processing
    and linking step
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: Snakemake
  role: Workflow engine that schedules the linking rule and passes minfrac/config
    parameters from config.yaml
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:4a3107318f8067f5ce7d5918ec8b1fa52483bbc30d7c3234e87e8ad2c0b6ece5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - FeatureLinkerUnlabeledKD
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# consensus-feature-linking

## Summary

Groups aligned, adduct-annotated LC-MS features across samples into a consensus feature map using OpenMS FeatureLinkerUnlabeledKD, then optionally filters consensus features with too many missing values (minfrac) before exporting a tab-separated feature matrix.

## When to use

Use when you have RT-aligned, centroided mzML-derived feature maps from multiple LC-MS/MS samples and need a single consensus feature table linking each feature's m/z, RT, adduct and per-sample intensities.

## When NOT to use

- Input is already a consensus feature table or TSV feature matrix (e.g. you are at the re-quantification or annotation stage instead)
- The dataset is raw vendor files (Thermo .raw, Agilent, Bruker) that have not yet been converted to centroided mzML and undergone feature detection and RT alignment
- Only a single sample map is available, since linking consensus feature groups across samples requires multiple input maps

## Inputs

- RT-aligned feature maps (featureXML) from centroided mzML files, after FeatureFinderMetabo and MapAlignerPoseClustering/MapRTTransformer
- Adduct-annotated feature maps from MetaboliteAdductDecharger with identifications mapped via IDMapper
- config/dataset.tsv listing sample files to be processed
- config/blanks.tsv and config/samples.tsv defining blank/QC and biological samples (optional)

## Outputs

- Consensus feature map (featureXML) from FeatureLinkerUnlabeledKD
- feature_table.tsv: tab-separated matrix with m/z, RT, adduct, and intensity columns per sample file

## How to apply

After FeatureFinderMetabo detection, MapAlignerPoseClustering alignment (applied via MapRTTransformer), MetaboliteAdductDecharger adduct annotation and IDMapper identification mapping, run FeatureLinkerUnlabeledKD (OpenMS 3.0) on the aligned feature maps to link consensus feature groups across all input files. Optionally apply the 'minfrac' filter to drop consensus features with too many missing values, and optionally remove blank/QC samples as defined in config/blanks.tsv and config/samples.tsv. Export the linked consensus map as feature_table.tsv, a tab-separated matrix with m/z, RT, adduct, and one intensity column per sample file. The rationale is that cross-sample grouping after alignment yields reliable per-sample quantities for downstream FBMN/IIMN export and marker selection, which UmetaFlow benchmarked on MTBLS733 and MTBLS736.

## Related tools

- **FeatureLinkerUnlabeledKD** (OpenMS 3.0 tool that links consensus feature groups across the aligned, adduct-annotated sample feature maps) — https://github.com/biosustain/snakemake_UmetaFlow
- **UmetaFlow** (Snakemake/pyOpenMS workflow that orchestrates feature linking together with minfrac and blank/QC filtering and exports feature_table.tsv) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings used in the Jupyter-notebook implementation of the pre-processing and linking step) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **Snakemake** (Workflow engine that schedules the linking rule and passes minfrac/config parameters from config.yaml) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- feature_table.tsv exists and is tab-separated with expected columns: m/z, RT, adduct, and one intensity column per input sample file
- Number of feature rows equals the consensus feature count after the optional minfrac filtering (rows with too many missing values are removed)
- Blank/QC sample intensities are absent from the table when config/blanks.tsv samples were defined for removal
- Feature m/z and RT values fall within the ranges of the aligned input maps and adduct annotations from MetaboliteAdductDecharger are present in the table
- On benchmark datasets MTBLS733/MTBLS736, the detected consensus features should cover more than 90% of ground truth features

## Limitations

- minfrac and blank/QC removal are optional and user-defined; without them, consensus features with many missing values and blank artefacts remain in the table
- Missing values from the pre-processing step persist unless the optional re-quantification step is run
- The paper does not state software version numbers for UmetaFlow, OpenMS, or Snakemake in the main text, so exact linking parameters must be taken from the repository config

## Evidence

- [intro] using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking"
- [readme] Optional minfrac step: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [methods] This step includes an optional removal of blank/QC samples if defined by the user.: "This step includes an optional removal of blank/QC samples if defined by the user."
