---
name: blank-sample-removal-qc
description: Use when raw LC-MS/MS files include blank media or QC injections and
  you want to strip those signals from the feature matrix before marker selection
  or quantification, as configured through UmetaFlow's blanks.tsv/config.yaml.
when_to_use_negative:
- When the dataset contains no blank or QC injections (blanks.tsv would be empty),
  skip the removal rule instead of running it on all samples.
- When starting from an already-generated feature table rather than raw vendor/mzML
  files — the blank/QC filter operates during UmetaFlow pre-processing, not on downstream
  annotation tables.
- When you intentionally want blank/QC signals retained (e.g. for QC-drift visualisation);
  enabling the rule would remove them from the consensus map.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: UmetaFlow
  role: Overall workflow whose pre-processing step hosts the optional blank/QC sample
    removal
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS 3.0
  role: Feature detection/alignment/consensus tools around which the blank/QC filter
    is applied
- name: Snakemake
  role: Workflow engine that executes the blank/QC-removal rule as part of the pre-processing
    DAG
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Python
  role: Used via data_files.py / Create_dataset_tsv notebook to generate dataset.tsv,
    blanks.tsv and samples.tsv inputs
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:1edd11695cdb13f741fcc62e0abe7bc52ab84c84165954310f769c56377ceb95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - Jupyter
  - OpenMS
  - OpenMS 3.0
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# blank-sample-removal-qc

## Summary

An optional UmetaFlow pre-processing sub-step that removes blank and QC samples and their associated consensus features (via a minfrac filter on missing values) so that feature detection, alignment, and downstream statistics operate only on true biological samples.

## When to use

Use when raw LC-MS/MS files include blank media or QC injections and you want to strip those signals from the feature matrix before marker selection or quantification, as configured through UmetaFlow's blanks.tsv/config.yaml.

## When NOT to use

- When the dataset contains no blank or QC injections (blanks.tsv would be empty), skip the removal rule instead of running it on all samples.
- When starting from an already-generated feature table rather than raw vendor/mzML files — the blank/QC filter operates during UmetaFlow pre-processing, not on downstream annotation tables.
- When you intentionally want blank/QC signals retained (e.g. for QC-drift visualisation); enabling the rule would remove them from the consensus map.

## Inputs

- Thermo/Agilent/Bruker vendor raw files or centroid mzML files placed in data/raw/ or data/mzML/
- config/dataset.tsv listing every file (e.g. MTBLS736 AB SCIEX TripleTOF 6600 runs)
- config/blanks.tsv naming blank media / QC injections
- config/samples.tsv naming true biological samples (e.g. SA and SB groups)
- config/config.yaml with ionisation mode and blank/QC-removal rule toggles (TRUE/FALSE)

## Outputs

- Cleaned consensus feature table (TSV) with blank/QC-derived features removed
- Feature matrix with missing-value-pruned consensus features after optional minfrac filtering
- Aligned mzML/featureXML intermediates in results/interim suitable for re-quantification

## How to apply

In the Snakemake UmetaFlow workflow, list blank/QC injections in config/blanks.tsv and biological samples in config/samples.tsv (generated from the MTBLS736-style raw file listing via `python data_files.py`), then enable the blank/QC-removal rule by writing TRUE for it in config/config.yaml. The pre-processing pipeline converts vendor files (e.g. Thermo) to centroid mzML via ThermoRawFileParser, runs OpenMS 3.0 FeatureFinderMetabolomics, MapAlignment and ConsensusMap tools, and removes consensus features that appear in blanks or in QC samples when those are designated. A second optional 'minfrac' filter drops consensus features with too many missing values across samples. Rationale: excluding blank-derived features prevents them from contaminating quantification tables and discriminating-marker lists (e.g. the SA/SB groups of MTBLS736), keeping the ground-truth comparison (90.1% detection, 81.7% accurate quantification, 59 true / 1 false markers) unbiased.

## Related tools

- **UmetaFlow** (Overall workflow whose pre-processing step hosts the optional blank/QC sample removal) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS 3.0** (Feature detection/alignment/consensus tools around which the blank/QC filter is applied)
- **Snakemake** (Workflow engine that executes the blank/QC-removal rule as part of the pre-processing DAG) — https://github.com/biosustain/snakemake_UmetaFlow
- **Python** (Used via data_files.py / Create_dataset_tsv notebook to generate dataset.tsv, blanks.tsv and samples.tsv inputs) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
python data_files.py && snakemake --use-conda --cores all
```

## Evaluation signals

- After enabling the rule, no feature rows are present that exist only in the samples listed in config/blanks.tsv.
- Sample counts in the consensus feature table equal the number of entries in config/samples.tsv (blanks/QC excluded).
- Minfrac-filtered matrix no longer contains consensus features with too many missing values (feature count should decrease after the optional minfrac step).
- Downstream marker selection on the MTBLS736 SA/SB groups should yield ~59 true markers and ≤1 false marker, consistent with blank contamination being removed.
- Dry-run (`snakemake --use-conda --dry-run`) shows the blank/QC-removal rule only scheduled when config.yaml flag is TRUE and blanks.tsv is non-empty.

## Limitations

- The step is optional — if the user fails to add blank/QC files to blanks.tsv or leaves the rule FALSE, blank-derived features remain in the final matrix.
- Only applicable during the pre-processing (and re-quantification) stage; features already exported to GNPS FBMN/IIMN cannot be filtered retroactively.
- Agilent/Bruker files must be converted independently with proteowizard before the workflow can apply this filter.

## Evidence

- [methods] Optional blank/QC removal: "This step includes an optional removal of blank/QC samples if defined by the user."
- [methods] minfrac removal of missing-value features: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] README workflow overview step 2: "This step includes an optional removal of blank/QC samples if defined by the user. Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] blanks.tsv / samples.tsv configuration: "If there are blanks/QC samples in the file list, add them to the appropriate table."
- [results] benchmark rationale: "UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection"
