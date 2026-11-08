---
name: adduct-annotation-feature-linking
description: Use when a targeted re-quantification (FeatureFinderMetaboIdent) has
  replaced intensity values in the feature matrix and the resulting consensus map
  needs adduct re-annotation, ID re-mapping and re-linking across all samples.
when_to_use_negative:
- The feature matrix has no missing intensity values after pre-processing — the re-quantification
  and its annotation/linking re-runs are an optional UmetaFlow step and can be skipped
- Input data are raw vendor files (e.g. Thermo .raw) rather than centroided mzML /
  a pre-processed consensus map — file conversion and pre-processing must happen first
- You only need to fill previously missing intensities — note UmetaFlow deliberately
  replaces ALL intensity values with targeted extraction results, so partial in-place
  patching of missing cells is out of scope for this skill
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: Provides MetaboliteAdductDecharger (adduct re-annotation), IDMapper (identification
    transfer) and FeatureLinkerUnlabeledKD (cross-sample linking) run on re-quantified
    features
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings used to drive the re-quantification annotation/linking steps
    in the Jupyter notebook implementation of UmetaFlow
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: Snakemake
  role: Orchestrates the re-quantification module as interdependent rules in the snakemake_UmetaFlow
    workflow
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: UmetaFlow
  role: The parent untargeted metabolomics workflow containing the optional re-quantification,
    adduct annotation and feature linking module
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:10f382d7faf3718ee1158308ed82d8414c2e488220555bacdea973704d578e5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# adduct-annotation-feature-linking

## Summary

Post-targeted-extraction annotation and consolidation steps from UmetaFlow's optional re-quantification module: after FeatureFinderMetaboIdent fills missing intensities, MetaboliteAdductDecharger, IDMapper and FeatureLinkerUnlabeledKD are re-run to restore adduct annotations, identifications and cross-sample feature linking.

## When to use

Use when a targeted re-quantification (FeatureFinderMetaboIdent) has replaced intensity values in the feature matrix and the resulting consensus map needs adduct re-annotation, ID re-mapping and re-linking across all samples.

## When NOT to use

- The feature matrix has no missing intensity values after pre-processing — the re-quantification and its annotation/linking re-runs are an optional UmetaFlow step and can be skipped
- Input data are raw vendor files (e.g. Thermo .raw) rather than centroided mzML / a pre-processed consensus map — file conversion and pre-processing must happen first
- You only need to fill previously missing intensities — note UmetaFlow deliberately replaces ALL intensity values with targeted extraction results, so partial in-place patching of missing cells is out of scope for this skill

## Inputs

- Re-quantified feature matrix produced by OpenMS FeatureFinderMetaboIdent on centroided mzML files in data/mzML/
- Pre-processed consensus map / consensus file from UmetaFlow pre-processing
- Target library of consensus features with at least one missing intensity value across samples
- config.yaml with user-defined minfrac threshold

## Outputs

- requantified_feature_table.consensusXML with filled intensities, adduct annotations and re-established feature links across all samples
- Final re-quantified consensus map from OpenMS FeatureLinkerUnlabeledKD

## How to apply

After FeatureFinderMetaboIdent targeted extraction replaces all intensity values in the feature matrix (not only the previously missing ones), three OpenMS 3.0 tools are re-run in sequence: MetaboliteAdductDecharger to re-annotate adducts on the re-quantified features, IDMapper to transfer identifications onto them, and FeatureLinkerUnlabeledKD to link features across samples into the final re-quantified consensus map. Before building the target library, apply the optional user-defined minfrac threshold from config.yaml to remove consensus features with too many missing values. The pipeline exists both as a Snakemake workflow (snakemake_UmetaFlow) and as Jupyter/pyOpenMS notebooks, with the re-quantification step toggled TRUE/FALSE in config.yaml. Validate that the output requantified_feature_table.consensusXML contains filled intensities for the targeted features across all samples with adduct annotations and feature linking re-established.

## Related tools

- **OpenMS** (Provides MetaboliteAdductDecharger (adduct re-annotation), IDMapper (identification transfer) and FeatureLinkerUnlabeledKD (cross-sample linking) run on re-quantified features) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings used to drive the re-quantification annotation/linking steps in the Jupyter notebook implementation of UmetaFlow) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **Snakemake** (Orchestrates the re-quantification module as interdependent rules in the snakemake_UmetaFlow workflow) — https://github.com/biosustain/snakemake_UmetaFlow
- **UmetaFlow** (The parent untargeted metabolomics workflow containing the optional re-quantification, adduct annotation and feature linking module) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- requantified_feature_table.consensusXML contains filled intensities for the targeted features across all samples
- Adduct annotations are re-established on the re-quantified features (MetaboliteAdductDecharger output present)
- Identifications are transferred onto the re-quantified features (IDMapper output present)
- Consensus features exceeding the minfrac missing-value threshold (set in config.yaml) have been removed
- Feature linking is re-established in the final consensus map produced by FeatureLinkerUnlabeledKD

## Limitations

- The re-quantification step replaces ALL intensity values with targeted extraction results, not only the previously missing ones
- minfrac is optional and user-defined in config.yaml; if misconfigured, consensus features with too many missing values may be retained or valid features dropped
- Benchmark numbers are in supplementary Tables S5 and S7, not the main text; no explicit version numbers are stated for UmetaFlow/OpenMS/SIRIUS/SnakeMake in the provided text

## Evidence

- [methods] Re-run OpenMS MetaboliteAdductDecharger on the re-quantified features to re-annotate adducts.: [evidence span withheld — anchor status: not_found]
- [methods] Re-run OpenMS FeatureLinkerUnlabeledKD to link features across samples into the final re-quantified consensus map.: [evidence span withheld — anchor status: not_found]
- [methods] Replace all intensity values in the feature matrix with the targeted extraction results, not only the previously missing ones.: [evidence span withheld — anchor status: not_found]
- [intro] using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking"
- [readme] Re-quantification (optional): Re-quantify all features with missing values across samples resulted from the pre-processing step: "Re-quantification (optional)**: Re-quantify all features with missing values across samples resulted from the pre-processing step"
- [readme] Optional minfrac step here allows for removal of consensus features with too many missing values.: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
