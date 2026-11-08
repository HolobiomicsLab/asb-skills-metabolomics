---
name: instrument-format-file-io
description: Use when input is raw/profile-mode LC-IM-MS data in an instrument format
  such as Agilent .d or UIMF, and the output must remain in the same instrument format
  without overwriting the original file.
when_to_use_negative:
- The input is already a feature table, centroided peak list, or annotated lipid list
  — the PreProcessor operates at the raw/profile data stage, not on processed feature
  tables
- A centroided or open-format (mzML) output is required — the skill's contract is
  same-instrument-format, profile-mode output
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PNNL PreProcessor
  role: Standalone software that reads instrument-format MS-files, runs the preprocessing
    modules, and writes new MS-files in the same instrument format (with User Guide
    and executable at the project site)
  repo: https://omics.pnl.gov/software/pnnl-preprocessor
- name: UIMF-Viewer
  role: Viewer for UIMF files, the intermediate format used to make preprocessing
    enhancements portable across IM-MS instrumentation
  repo: PNNL-Comp-Mass-Spec/UIMF-Viewer
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00425
    title: A Preprocessing Tool for Enhanced Ion Mobility–Mass Spectrometry-Based
      Omics Workflows
  source_content_hash: sha256:75f4533eebcc1ba4d032d3d8a249b0efcd24004641bdb3f92662e3bd0ca99e79
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ChemCalc
  - PNNL PreProcessor
  - R
  - UIMF-Viewer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# instrument-format-file-io

## Summary

Read and write raw MS data in instrument-native formats (Agilent MassHunter .d, UIMF profile mode) as part of the PNNL PreProcessor workflow, so preprocessed files stay vendor-readable and can be compared against deposited originals (e.g., MassIVE MSV000087295).

## When to use

Use when input is raw/profile-mode LC-IM-MS data in an instrument format such as Agilent .d or UIMF, and the output must remain in the same instrument format without overwriting the original file.

## When NOT to use

- The input is already a feature table, centroided peak list, or annotated lipid list — the PreProcessor operates at the raw/profile data stage, not on processed feature tables
- A centroided or open-format (mzML) output is required — the skill's contract is same-instrument-format, profile-mode output

## Inputs

- Raw LC-IM-MS MS-file in Agilent MassHunter .d format (e.g., human plasma lipid-extract triplicates from MassIVE MSV000087295)
- Profile-mode UIMF file (e.g., output of an upstream smoothing step)
- User parameters: low intensity threshold (e.g., 20 or 50 counts), spike-removal adjacent-point criterion, saturation threshold (e.g., 40% of the abundance limit)

## Outputs

- New MS-file in the same instrument format as the input, in profile mode (not centroided), with a suffix appended (e.g., '_SR' for saturation repair)
- Log file recording the applied algorithms and parameters
- Original input file left unmodified

## How to apply

Load the instrument-format MS-file (e.g., Agilent MassHunter .d triplicate files from MassIVE MSV000087295, or UIMF profile-mode data). Apply preprocessing modules (smoothing, low intensity threshold, spike removal, saturation repair) while writing the filtered/repaired spectra in profile mode — without centroiding — to a NEW file in the same instrument format as the input, never overwriting the original, using a suffix such as '_SR' for saturation-repaired output. Record the applied algorithms and parameters in a log file. The rationale is that keeping vendor format and profile mode preserves compatibility with downstream tools like Lipid Annotator (which directly processes Agilent .d files) and allows structural comparison against deposited reference files; converting to UIMF is an established intermediate step that makes enhancements applicable across other IM-MS instrumentation.

## Related tools

- **PNNL PreProcessor** (Standalone software that reads instrument-format MS-files, runs the preprocessing modules, and writes new MS-files in the same instrument format (with User Guide and executable at the project site)) — https://omics.pnl.gov/software/pnnl-preprocessor
- **UIMF-Viewer** (Viewer for UIMF files, the intermediate format used to make preprocessing enhancements portable across IM-MS instrumentation) — PNNL-Comp-Mass-Spec/UIMF-Viewer

## Examples

```
PNNL PreProcessor --input lipid_triplicate.d --saturation_repair --threshold 40% --output_mode profile --suffix _SR --log preprocess.log
```

## Evaluation signals

- The output file exists and parses as a valid MS-file in the same instrument format as the input (e.g., .d or UIMF)
- The output is in profile mode, i.e., data were not centroided
- The original input file is unchanged and the output was written to a new file (e.g., with the '_SR' suffix)
- The filtered/repaired file is smaller than its input when noise filtering was applied, since background points and spikes were removed
- The output structurally matches the deposited preprocessed reference file (e.g., MSV000087295) within format/tolerance

## Limitations

- Enhancements are tied to specific IM-MS instrumentation and may require converting to UIMF as an intermediate step to be applicable to other vendors' data
- No tool version is specified in the article's discussion; the software executable and User Guide must be obtained from the omics.pnl.gov site
- Random, isolated spike points exceeding the threshold but lacking adjacent neighbors (e.g., 1 adjacent point criterion) are deleted, so genuinely isolated signals could be removed

## Evidence

- [other] Write the filtered spectra in profile mode (without centroiding) to a new MS-file in the same instrument format as the input, without overwriting the original data, and record the applied algorithms and parameters in a log file.: [evidence span withheld — anchor status: not_found]
