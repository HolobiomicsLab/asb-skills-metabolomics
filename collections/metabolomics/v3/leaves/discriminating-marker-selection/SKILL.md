---
name: discriminating-marker-selection
description: Use when reproducing or benchmarking the marker-selection stage of an
  untargeted metabolomics workflow on MTBLS736 (SA vs SB groups), where a ground-truth
  set of 68 true discriminating markers is available to score sensitivity and specificity.
when_to_use_negative:
- When the goal is only spectral matching or structural/formula prediction (SIRIUS,
  CSI:FingerID, CANOPUS, GNPS FBMN/IIMN) rather than benchmarking marker selection
  between sample groups.
- When no ground truth set of known discriminating markers exists — the 59/68 true
  + 1 false scoring depends on the MTBLS736 benchmark list; without it the method
  cannot be evaluated.
- When the input is already a re-quantified feature matrix and only downstream statistics
  are needed — UmetaFlow's Snakemake conversion/pre-processing would be redundant.
edam_operation: http://edamontology.org/operation_3809
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: UmetaFlow
  role: Snakemake/pyOpenMS workflow performing the pre-processing and re-quantification
    whose output feeds the marker-selection statistics
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Snakemake
  role: Orchestrates file conversion, pre-processing and re-quantification steps and
    supports dry-run validation of the pipeline before marker selection
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS 3.0
  role: Provides feature detection, map alignment, adduct annotation, re-quantification
    and feature linking algorithms used to generate the feature matrix prior to statistical
    marker testing
- name: pyOpenMS
  role: Python bindings to the OpenMS algorithms used in the Jupyter implementation
    of the same workflow that yields the feature matrix for marker selection
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:243d525432a5d6557bdc63086db982c3a274e6b32e18024b1d65bf03b9d6ca97
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
  edam_operation: http://edamontology.org/operation_3809
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# discriminating-marker-selection

## Summary

Benchmark UmetaFlow's ability to pick discriminating markers between sample groups on the MTBLS736 dataset (AB SCIEX TripleTOF 6600), scoring 59 of 68 true and 1 false marker against competing tools (MarkerView, MS-Dial, MZmine2, XCMS).

## When to use

Use when reproducing or benchmarking the marker-selection stage of an untargeted metabolomics workflow on MTBLS736 (SA vs SB groups), where a ground-truth set of 68 true discriminating markers is available to score sensitivity and specificity.

## When NOT to use

- When the goal is only spectral matching or structural/formula prediction (SIRIUS, CSI:FingerID, CANOPUS, GNPS FBMN/IIMN) rather than benchmarking marker selection between sample groups.
- When no ground truth set of known discriminating markers exists — the 59/68 true + 1 false scoring depends on the MTBLS736 benchmark list; without it the method cannot be evaluated.
- When the input is already a re-quantified feature matrix and only downstream statistics are needed — UmetaFlow's Snakemake conversion/pre-processing would be redundant.

## Inputs

- MTBLS736 raw mass-spectrometry files (AB SCIEX TripleTOF 6600) downloaded from MetaboLights into data/raw/ (or pre-converted mzML in data/mzML/)
- config/dataset.tsv sample table listing filenames and comments (e.g. ISP2_blank / NBC_00162)
- config/blanks.tsv and config/samples.tsv separating blank/QC and true samples
- config/config.yaml with dataset-specific ionisation mode and pipeline step toggles (TRUE/FALSE)
- Ground-truth list of 68 true discriminating markers between SA and SB sample groups
- Ground-truth list of 970 annotated features for detection-rate benchmarking

## Outputs

- Annotated feature matrix (TSV) from UmetaFlow pre-processing and re-quantification
- Ranked list of discriminating markers between SA and SB groups
- Benchmark counts: number of detected features vs 970 ground truth, % accurately quantified features, count of true/false discriminating markers
- Comparison table of marker-selection performance vs MarkerView, MS-Dial, MZmine2 and XCMS (Table S7 / Table S5 in supplementary)

## How to apply

Run the UmetaFlow Snakemake pipeline on the MTBLS736 files with file conversion, pre-processing (feature detection, map alignment, adduct annotation) and re-quantification enabled so you obtain an annotated, gap-filled feature matrix; convert vendor raw files to centroid mzML via ThermoRawFileParser. Use config/dataset.tsv, config/blanks.tsv and config/samples.tsv to define the true samples and blank/QC groups, set ionisation mode in config/config.yaml, and validate with `snakemake --use-conda --dry-run`. After pre-processing, apply statistical testing on the re-quantified feature matrix between the SA and SB sample groups to rank features as discriminating markers. Score the selection against the ground truth: 970 annotated features should yield ~874 detected features (90.1%), ~81.7% accurately quantified, and 59 of 68 true markers plus 1 false positive. Compare the same statistics against MarkerView, MS-Dial, MZmine2 and XCMS to place UmetaFlow's ranking.

## Related tools

- **UmetaFlow** (Snakemake/pyOpenMS workflow performing the pre-processing and re-quantification whose output feeds the marker-selection statistics) — https://github.com/biosustain/snakemake_UmetaFlow
- **Snakemake** (Orchestrates file conversion, pre-processing and re-quantification steps and supports dry-run validation of the pipeline before marker selection) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS 3.0** (Provides feature detection, map alignment, adduct annotation, re-quantification and feature linking algorithms used to generate the feature matrix prior to statistical marker testing)
- **pyOpenMS** (Python bindings to the OpenMS algorithms used in the Jupyter implementation of the same workflow that yields the feature matrix for marker selection) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- Detection rate: annotated features should exceed 90% of the 970 ground-truth features (~874) on MTBLS736.
- Quantification accuracy: ~81.7% of features should be accurately quantified.
- Marker selection: exactly 59 true and 1 false discriminating markers should be recovered from the 68-marker ground truth.
- Dry-run check: `snakemake --use-conda --dry-run` resolves all rules without error before the full run.
- Cross-tool benchmark: compare the marker-selection statistics against MarkerView, MS-Dial, MZmine2 and XCMS on the same dataset.

## Limitations

- Only tested on MTBLS733 and MTBLS736 benchmarks; no changelog or software-version numbers are provided in the article.
- Benchmark performance numbers (Table S5, S7) appear only in supplementary files, not in the main text.
- Marker-selection scoring depends on the existence of a ground truth marker list — without it, sensitivity/specificity cannot be computed.

## Evidence

- [abstract] MTBLS736 marker-selection performance: [evidence span withheld — anchor status: not_found]
- [intro] OpenMS 3.0 toolset: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
- [methods] Optional blank/QC removal: "This step includes an optional removal of blank/QC samples if defined by the user."
- [methods] Optional minfrac filtering: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] Config file and dry-run usage: "Test your configuration by performing a dry-run via snakemake --use-conda --dry-run"
- [readme] Data preparation and execution: "Add all your files in the data/raw/ or data/mzML/ directory and generate the dataset.tsv table to specify the samples (filenames) that will be processed."
