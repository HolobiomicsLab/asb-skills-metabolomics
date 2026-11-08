---
name: untargeted-metabolomics-preprocessing
description: Use when starting from centroided (or raw Thermo/Agilent/Bruker) LC-MS/MS
  mzML files in an untargeted metabolomics study and you need a per-sample feature
  table with m/z, RT, adduct, and intensities for downstream FBMN/IIMN or statistics.
when_to_use_negative:
- Input is already a consensus feature table (e.g. from XCMS or a prior UmetaFlow
  run) rather than raw/centroided mzML — use downstream annotation or re-quantification
  instead.
- The goal is re-quantification of missing values — the README defines a separate
  optional Re-quantification step (3_Requantification.ipynb) that runs after pre-processing.
- The goal is structural annotation or molecular networking — those are handled by
  the SIRIUS/CSI:FingerID, GNPS FBMN/IIMN, and spectral matching steps, not the pre-processing
  module.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: OpenMS
  role: 'Provides the 3.0 algorithms executed in this module: HighResPrecursorMassCorrector,
    FeatureFinderMetabo, MapAlignerPoseClustering, MapRTTransformer, MetaboliteAdductDecharger,
    IDMapper, FeatureLinkerUnlabeledKD'
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: pyOpenMS
  role: Python bindings used to call the OpenMS pre-processing tools in the Jupyter
    notebook implementation (2_Preprocessing.ipynb)
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: Snakemake
  role: Workflow engine that chains the pre-processing rules, driven by config.yaml
    TRUE/FALSE flags and the dataset/blanks/samples TSV tables
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: ThermoRawFileParser
  role: Upstream optional FileConversion step producing the centroided mzML files
    this module consumes
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:b2174469019651e194bc9490953fd3cb7f2912a993ad214f5ec6e047eaf71249
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - OpenMS
  - Python
  - Snakemake
  - ThermoRawFileParser
  - UmetaFlow
  - pyOpenMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# untargeted-metabolomics-preprocessing

## Summary

UmetaFlow's pre-processing module converts centroided LC-MS/MS mzML files into a consensus feature table using OpenMS 3.0 algorithms (feature detection, RT alignment, adduct annotation, feature linking) orchestrated by Snakemake or pyOpenMS notebooks.

## When to use

Use when starting from centroided (or raw Thermo/Agilent/Bruker) LC-MS/MS mzML files in an untargeted metabolomics study and you need a per-sample feature table with m/z, RT, adduct, and intensities for downstream FBMN/IIMN or statistics.

## When NOT to use

- Input is already a consensus feature table (e.g. from XCMS or a prior UmetaFlow run) rather than raw/centroided mzML — use downstream annotation or re-quantification instead.
- The goal is re-quantification of missing values — the README defines a separate optional Re-quantification step (3_Requantification.ipynb) that runs after pre-processing.
- The goal is structural annotation or molecular networking — those are handled by the SIRIUS/CSI:FingerID, GNPS FBMN/IIMN, and spectral matching steps, not the pre-processing module.

## Inputs

- centroided mzML files listed in config/dataset.tsv
- Thermo raw files in data/raw/ (optional, converted via FileConversion step)
- config/dataset.tsv sample table (sample_name, comment)
- config/samples.tsv and config/blanks.tsv (optional QC/blank definitions)
- config.yaml (ionisation mode, per-step TRUE/FALSE flags, parameters)

## Outputs

- feature_table.tsv: tab-separated matrix with m/z, RT, adduct, and intensity columns per input sample
- consensus feature map from FeatureLinkerUnlabeledKD (results/interim files)

## How to apply

Load the centroided mzML files listed in config/dataset.tsv and run HighResPrecursorMassCorrector on each map to correct precursor masses. Detect LC-MS features per sample with FeatureFinderMetabo, then align retention times across maps with MapAlignerPoseClustering and apply the transformations via MapRTTransformer. Annotate adduct states with MetaboliteAdductDecharger and attach identifications with IDMapper, then link features across samples with FeatureLinkerUnlabeledKD to build the consensus feature map. Apply the optional blank/QC sample removal defined in config/blanks.tsv and the optional 'minfrac' filter to drop consensus features with too many missing values. Export the result as a tab-separated feature_table.tsv with columns for m/z, RT, adduct, and per-sample intensities; re-quantification (step 3) is a separate optional step that recovers missing values after this stage.

## Related tools

- **OpenMS** (Provides the 3.0 algorithms executed in this module: HighResPrecursorMassCorrector, FeatureFinderMetabo, MapAlignerPoseClustering, MapRTTransformer, MetaboliteAdductDecharger, IDMapper, FeatureLinkerUnlabeledKD) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **pyOpenMS** (Python bindings used to call the OpenMS pre-processing tools in the Jupyter notebook implementation (2_Preprocessing.ipynb)) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **Snakemake** (Workflow engine that chains the pre-processing rules, driven by config.yaml TRUE/FALSE flags and the dataset/blanks/samples TSV tables) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **ThermoRawFileParser** (Upstream optional FileConversion step producing the centroided mzML files this module consumes) — https://github.com/biosustain/snakemake_UmetaFlow.git

## Examples

```
git clone https://github.com/biosustain/snakemake_UmetaFlow.git && snakemake --use-conda --cores all
```

## Evaluation signals

- feature_table.tsv exists, is tab-separated, and contains m/z, RT, adduct, and one intensity column per input sample file.
- Number of feature rows in feature_table.tsv reflects consensus features surviving the optional minfrac missing-value filter.
- Consensus features were linked across all samples defined in config/dataset.tsv, with blank/QC samples (e.g. ISP2_blank) removed when listed in config/blanks.tsv.
- Each input map passed through precursor mass correction (HighResPrecursorMassCorrector) and RT alignment transformations before feature linking.
- Benchmarking expectation: on MTBLS733 and MTBLS736, UmetaFlow pre-processing detected more than 90% of all ground truth features.

## Limitations

- Removal of blank/QC samples and the minfrac filter are optional and only applied if defined by the user in the configuration tables — omitting them leaves features with many missing values.
- Pre-processing can leave missing values across samples; the workflow provides an optional separate re-quantification step 'to avoid missing values resulted by the pre-processing workflow'.
- Only Thermo raw files are converted in-workflow; Agilent or Bruker files must be converted independently using proteowizard before pre-processing.
- No software version numbers are stated for UmetaFlow, OpenMS, or Snakemake in the article text (the README separately pins snakemake ≥7.14.0 and OpenMS 3.3.0 for MacOS Silicon).

## Evidence

- [readme] Pre-processing: converting raw data to a feature table with a series of algorithms through feature detection, alignment and grouping.: "Pre-processing: converting raw data to a feature table with a series of algorithms through feature detection, alignment and grouping."
- [methods] Optional minfrac step allows removal of consensus features with too many missing values.: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [methods] This step includes an optional removal of blank/QC samples if defined by the user.: "This step includes an optional removal of blank/QC samples if defined by the user."
- [intro] using OpenMS 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking"
- [abstract] UmetaFlow detected more than 90% of all ground truth features on benchmark datasets.: "UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection"
