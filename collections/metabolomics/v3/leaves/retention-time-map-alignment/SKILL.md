---
name: retention-time-map-alignment
description: Use when multiple centroided LC-MS mzML maps must be brought onto a common
  retention-time scale (e.g. before FeatureLinkerUnlabeledKD consensus grouping) and
  RT drift between runs would break cross-sample feature matching.
when_to_use_negative:
- Input is already a consensus/feature table — alignment operates on per-sample feature
  maps, not exported TSV feature tables.
- Only a single sample map is being processed — there are no inter-run RT shifts to
  correct.
- Data are unconverted vendor raw files (Thermo .raw, Agilent, Bruker) — convert to
  centroided mzML first (UmetaFlow file conversion or ProteoWizard).
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: OpenMS
  role: provides MapAlignerPoseClustering for RT alignment and MapRTTransformer for
    applying the estimated RT transformations
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings used to call the OpenMS alignment tools in the Jupyter-notebook
    implementation of the pre-processing step
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: UmetaFlow
  role: Snakemake workflow that orchestrates the alignment between feature detection
    and FeatureLinkerUnlabeledKD grouping
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Snakemake
  role: workflow engine executing the pre-processing rules including RT alignment
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:196b5e1d796329ac54ba873717579528d1c577f18abc5c4571604a5b41eb6f75
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# retention-time-map-alignment

## Summary

Aligns retention times across LC-MS/MS sample maps before feature linking, as implemented in UmetaFlow with the OpenMS tools MapAlignerPoseClustering and MapRTTransformer on centroided mzML feature maps.

## When to use

Use when multiple centroided LC-MS mzML maps must be brought onto a common retention-time scale (e.g. before FeatureLinkerUnlabeledKD consensus grouping) and RT drift between runs would break cross-sample feature matching.

## When NOT to use

- Input is already a consensus/feature table — alignment operates on per-sample feature maps, not exported TSV feature tables.
- Only a single sample map is being processed — there are no inter-run RT shifts to correct.
- Data are unconverted vendor raw files (Thermo .raw, Agilent, Bruker) — convert to centroided mzML first (UmetaFlow file conversion or ProteoWizard).

## Inputs

- centroided mzML files listed in config/dataset.tsv (per-sample LC-MS/MS runs)
- per-sample feature maps produced by FeatureFinderMetabo

## Outputs

- retention-time-transformed (aligned) feature maps for downstream adduct annotation and consensus feature linking
- consensus feature table (feature_table.tsv) with m/z, RT, adduct, and intensity per sample after linking

## How to apply

First run FeatureFinderMetabo on each centroided mzML file (after optional HighResPrecursorMassCorrector precursor mass correction) so each sample is a feature map. Then compute retention-time transformations across all sample maps with MapAlignerPoseClustering, and apply those transformations to the maps via MapRTTransformer — UmetaFlow separates the alignment estimation from its application. Aligned, adduct-annotated (MetaboliteAdductDecharger) and ID-mapped (IDMapper) maps are then passed to FeatureLinkerUnlabeledKD to build consensus features. Alignment is a prerequisite for reliable feature linking and for the minfrac missing-value filter to behave meaningfully downstream.

## Related tools

- **OpenMS** (provides MapAlignerPoseClustering for RT alignment and MapRTTransformer for applying the estimated RT transformations) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings used to call the OpenMS alignment tools in the Jupyter-notebook implementation of the pre-processing step) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **UmetaFlow** (Snakemake workflow that orchestrates the alignment between feature detection and FeatureLinkerUnlabeledKD grouping) — https://github.com/biosustain/snakemake_UmetaFlow
- **Snakemake** (workflow engine executing the pre-processing rules including RT alignment) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- Aligned feature maps show consistent RTs across samples for the same metabolite features, enabling FeatureLinkerUnlabeledKD to group them into consensus features.
- Downstream feature_table.tsv exists and is tab-separated with expected columns (m/z, RT, adduct, intensity per sample file), and row counts reflect consensus features after minfrac filtering.
- Alignment step completes without errors in a `snakemake --use-conda --dry-run` configuration test and in the full run on the mzML files in data/mzML/.
- The number of consensus features linked is plausible relative to ground truth (on benchmark datasets MTBLS733/MTBLS736 UmetaFlow detected >90% of ground truth features).

## Limitations

- Alignment quality depends on prior feature detection on centroided data; UmetaFlow pre-processing assumes centroided mzML input.
- Version drift: the paper used OpenMS 3.0, while the current README installation targets OpenMS 3.3.0 (with manual codesign on MacOS Silicon); no software version numbers were stated for all tools in the article text.
- Alignment precedes an optional re-quantification step because pre-processing itself leaves missing values across samples; alignment alone does not recover missing features.

## Evidence

- [methods] Align retention times across all sample maps with MapAlignerPoseClustering and apply the resulting transformations via MapRTTransformer.: [evidence span withheld — anchor status: not_found]
- [intro] using OpenMS 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
- [readme] converting raw data to a feature table with a series of algorithms through feature detection, alignment and grouping.: "converting raw data to a feature table with a series of algorithms through feature detection, alignment and grouping."
