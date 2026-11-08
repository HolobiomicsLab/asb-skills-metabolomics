---
name: metabolite-adduct-annotation
description: Use when processing raw centroided LC-MS/MS mzML data through UmetaFlow
  and features must carry adduct/charge information before feature linking, re-quantification,
  or GNPS export.
when_to_use_negative:
- Input is already an adduct-annotated feature table (e.g. output of a prior MetaboliteAdductDecharger
  run or the optional re-quantification step) — re-annotating would be redundant
- Data has not yet been feature-detected and RT-aligned; adduct annotation must not
  replace the FeatureFinderMetabo/MapAlignerPoseClustering stages
- Working outside the OpenMS/UmetaFlow toolchain (e.g. XCMS-based pipelines) where
  MetaboliteAdductDecharger is not part of the workflow
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: OpenMS
  role: MetaboliteAdductDecharger (OpenMS 3.x TOPP tool) performs the adduct/decharge
    annotation on aligned feature maps
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: UmetaFlow
  role: Snakemake/pyOpenMS workflow that orders adduct annotation between RT alignment
    and feature linking
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings to the OpenMS adduct-annotation algorithm used in the Jupyter
    notebook implementation
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:51b33ee25aaa9ec9f7b307e517f20dfd9d41da6d2f9fd7a2a70dea279f811eba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - OpenMS
  - Python
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolite-adduct-annotation

## Summary

Annotates LC-MS/MS features with their adduct states using the OpenMS MetaboliteAdductDecharger tool inside the UmetaFlow pre-processing pipeline, so each consensus feature in feature_table.tsv carries an adduct column for downstream GNPS IIMN and annotation steps.

## When to use

Use when processing raw centroided LC-MS/MS mzML data through UmetaFlow and features must carry adduct/charge information before feature linking, re-quantification, or GNPS export.

## When NOT to use

- Input is already an adduct-annotated feature table (e.g. output of a prior MetaboliteAdductDecharger run or the optional re-quantification step) — re-annotating would be redundant
- Data has not yet been feature-detected and RT-aligned; adduct annotation must not replace the FeatureFinderMetabo/MapAlignerPoseClustering stages
- Working outside the OpenMS/UmetaFlow toolchain (e.g. XCMS-based pipelines) where MetaboliteAdductDecharger is not part of the workflow

## Inputs

- Retention-time-aligned feature maps (featureXML) from FeatureFinderMetabo and MapAlignerPoseClustering/MapRTTransformer
- config.yaml with the ionisation mode (positive/negative) set for the dataset

## Outputs

- Adduct-annotated feature maps feeding into FeatureLinkerUnlabeledKD
- Consensus feature table (feature_table.tsv) containing m/z, RT, adduct, and per-sample feature intensities

## How to apply

In the UmetaFlow pre-processing step, run adduct annotation on the feature maps AFTER feature detection (FeatureFinderMetabo) and retention-time alignment (MapAlignerPoseClustering + MapRTTransformer), and BEFORE FeatureLinkerUnlabeledKD. First set the ionisation mode (e.g. positive/negative) in config.yaml, since this determines which adduct states MetaboliteAdductDecharger considers. The tool decharges the aligned features by grouping mass signals into plausible adduct/charge states, and IDMapper then maps identifications onto the annotated features. The resulting adduct annotations propagate into the consensus feature table exported as feature_table.tsv (with m/z, RT, adduct, and per-sample intensity columns) and into the GNPS IIMN export where collapsed ion-identity edges rely on consistent adduct assignment.

## Related tools

- **OpenMS** (MetaboliteAdductDecharger (OpenMS 3.x TOPP tool) performs the adduct/decharge annotation on aligned feature maps) — https://github.com/biosustain/snakemake_UmetaFlow
- **UmetaFlow** (Snakemake/pyOpenMS workflow that orders adduct annotation between RT alignment and feature linking) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings to the OpenMS adduct-annotation algorithm used in the Jupyter notebook implementation) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
snakemake --use-conda --cores all  # runs UmetaFlow pre-processing incl. MetaboliteAdductDecharger adduct annotation (config/config.yaml)
```

## Evaluation signals

- feature_table.tsv contains a populated adduct column alongside m/z, RT, and per-sample intensity columns and is tab-separated
- Adduct assignments are consistent with the ionisation mode configured in config.yaml (e.g. only positive-mode adducts in positive-ion data)
- Annotated feature maps retain the same number of features as the RT-aligned input maps (annotation does not drop features)
- Downstream GNPS IIMN export shows collapsed ion-identity edges only for features sharing plausible adduct/charge relationships
- Consensus feature row count after the optional minfrac filter reflects the linked, adduct-annotated consensus features

## Limitations

- Adduct correctness depends on setting the ionisation mode and parameters in config.yaml appropriately for the dataset (e.g. positive/negative ionisation)
- The optional minfrac step can remove consensus features with too many missing values, so adduct-annotated features may be dropped before export
- Optional removal of blank/QC samples defined in config/blanks.tsv and config/samples.tsv occurs in the same pre-processing step and affects which features carry annotations

## Evidence

- [intro] OpenMS 3.x adduct annotation in pipeline: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
- [readme] pre-processing pipeline position: "Pre-processing: converting raw data to a feature table with a series of algorithms through feature detection, alignment and grouping."
- [readme] ionisation mode configuration: "Set parameters according to your dataset as explained in the commented section in the yaml file (e.g. positive/negative ionisation etc.)"
- [methods] minfrac filtering of consensus features: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
