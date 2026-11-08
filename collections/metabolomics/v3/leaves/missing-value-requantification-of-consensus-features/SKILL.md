---
name: missing-value-requantification-of-consensus-features
description: Use when an untargeted LC-MS/MS pre-processing consensus map contains
  consensus features with missing intensity values in some sample files, and complete
  quantification is needed for statistical analysis.
when_to_use_negative:
- The feature matrix is already complete — no consensus features have missing intensity
  values, so targeted re-extraction offers no benefit.
- You want to keep original untargeted intensities untouched and only impute a few
  cells; UmetaFlow replaces all intensity values with targeted extraction results.
- Input is raw vendor data not yet pre-processed — re-quantification operates on the
  pre-processing consensus map, not raw files.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: Provides FeatureFinderMetaboIdent, MetaboliteAdductDecharger, IDMapper, and
    FeatureLinkerUnlabeledKD used in the re-quantification step
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings to the OpenMS algorithms used in the Jupyter-notebook implementation
    of re-quantification
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: Snakemake
  role: Workflow engine orchestrating the optional re-quantification rule and config.yaml
    minfrac settings
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: UmetaFlow
  role: Workflow whose step 3 ('Re-quantification (optional)') implements this skill
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:9c5310e7513beeb490d4be53d080b670bd326cc10f23123711a86026257078ff
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# missing-value-requantification-of-consensus-features

## Summary

UmetaFlow's optional re-quantification step fills missing intensities arising from untargeted pre-processing by running targeted extraction (FeatureFinderMetaboIdent) against a target library of consensus features that have at least one missing value across samples.

## When to use

Use when an untargeted LC-MS/MS pre-processing consensus map contains consensus features with missing intensity values in some sample files, and complete quantification is needed for statistical analysis.

## When NOT to use

- The feature matrix is already complete — no consensus features have missing intensity values, so targeted re-extraction offers no benefit.
- You want to keep original untargeted intensities untouched and only impute a few cells; UmetaFlow replaces all intensity values with targeted extraction results.
- Input is raw vendor data not yet pre-processed — re-quantification operates on the pre-processing consensus map, not raw files.

## Inputs

- Pre-processed consensus map (consensusXML) from UmetaFlow pre-processing with missing intensity values
- Centroided mzML files in data/mzML/
- config.yaml with minfrac threshold and workflow step flags

## Outputs

- requantified_feature_table.consensusXML with filled intensities across all samples
- Re-annotated adduct assignments (MetaboliteAdductDecharger)
- Re-linked consensus map (FeatureLinkerUnlabeledKD)
- Re-transferred identifications (IDMapper)

## How to apply

Build a target library from the pre-processed consensus map by selecting every consensus feature that has at least one missing intensity value across sample files, optionally applying the user-defined minfrac threshold (set in config.yaml) to drop consensus features with too many missing values. Run OpenMS FeatureFinderMetaboIdent targeted extraction on every centroided mzML file in data/mzML/ against this library, then replace ALL intensity values in the feature matrix with the targeted extraction results — not only the previously missing ones. Re-run MetaboliteAdductDecharger to re-annotate adducts, IDMapper to re-transfer identifications, and FeatureLinkerUnlabeledKD to link features into the final re-quantified consensus map. Rationale: targeted extraction recovers signals the untargeted feature detection missed, giving a more complete matrix for downstream statistics.

## Related tools

- **OpenMS** (Provides FeatureFinderMetaboIdent, MetaboliteAdductDecharger, IDMapper, and FeatureLinkerUnlabeledKD used in the re-quantification step) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings to the OpenMS algorithms used in the Jupyter-notebook implementation of re-quantification) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **Snakemake** (Workflow engine orchestrating the optional re-quantification rule and config.yaml minfrac settings) — https://github.com/biosustain/snakemake_UmetaFlow
- **UmetaFlow** (Workflow whose step 3 ('Re-quantification (optional)') implements this skill) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
snakemake --use-conda --cores all  # with config.yaml requantification=TRUE and minfrac set; alternatively run notebook 3_Requantification(optional).ipynb
```

## Evaluation signals

- requantified_feature_table.consensusXML contains filled intensities for the targeted features across all samples
- Consensus features exceeding the minfrac missing-value threshold are removed from the re-quantified output
- Adduct annotations are re-established after MetaboliteAdductDecharger re-runs on re-quantified features
- Feature linking is re-established: FeatureLinkerUnlabeledKD produces a final re-quantified consensus map spanning samples
- Identifications are transferred onto re-quantified features via IDMapper

## Limitations

- minfrac threshold is a user decision in config.yaml; too aggressive a setting removes genuine low-prevalence features before re-quantification.
- Replacing ALL intensity values (not just missing ones) means pre-existing quantifications are overwritten by targeted extraction results.
- No software version numbers stated for UmetaFlow/OpenMS in the article text; MacOS setup pins OpenMS 3.3.0 manually.

## Evidence

- [readme] Re-quantification (optional): "3) **Re-quantification (optional)**: Re-quantify all features with missing values across samples resulted from the pre-processing step for more reliable statistical analysis and data exploration."
- [readme] README step 2/3 minfrac: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [intro] OpenMS 3.0 toolchain: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
- [abstract] pyOpenMS bindings: "Jupyter notebooks using the Python programming language and a set of Python bindings to the OpenMS algorithms (pyOpenMS)"
- [methods] task workflow card: [evidence span withheld — anchor status: not_found]
