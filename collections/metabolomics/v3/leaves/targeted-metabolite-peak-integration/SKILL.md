---
name: targeted-metabolite-peak-integration
description: Use when performing semi-targeted quantification of metabolites in LC-MS
  data (e.g. TripleTOF or QE HF .aird files) where EICs must be extracted, peaks selected/integrated,
  and compounds identified against mz/rt libraries with per-stage method choices.
when_to_use_negative:
- Input is vendor raw data (e.g. .raw, .wiff2, .d) that has not yet been converted
  to Aird format by AirdPro — run file conversion first
- The analysis is untargeted feature discovery rather than semi-targeted detection
  against a defined mz/rt library
- Acquisition is not DDA; MetaPro's documented workflow 'supports MS data acquired
  with the data-dependent acquisition (DDA) method'
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetaPro
  role: Web-based application hosting the eight-step targeted quantification workflow,
    Method parameter configuration, manual inspection/re-integration, and .xlsx report
    export
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: Converts vendor raw files (DDA) into the Aird computation-oriented format
    that MetaPro's EIC extraction reads
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: Redis
  role: Started as part of the MetaPro runtime stack when launching the application
    server
- name: MongoDB
  role: Started as part of the MetaPro runtime stack when launching the application
    server
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:a4873c5e1a8bbd82a79f5ec47382be1628e1a9589d4bb1e07136c726726b0b09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# targeted-metabolite-peak-integration

## Summary

Reconstructs MetaPro's eight-step automatic quantification pipeline (EIC extraction, smoothing, noise estimation, peak selection, integration, screening, identification, batch optimization) for semi-targeted LC-MS metabolomics, exposing configurable method counts at each stage: 4 smoothing, 3 noise estimation, 3 peak selection, 2 integration, 5 screening criteria, and 3 identification methods.

## When to use

Use when performing semi-targeted quantification of metabolites in LC-MS data (e.g. TripleTOF or QE HF .aird files) where EICs must be extracted, peaks selected/integrated, and compounds identified against mz/rt libraries with per-stage method choices.

## When NOT to use

- Input is vendor raw data (e.g. .raw, .wiff2, .d) that has not yet been converted to Aird format by AirdPro — run file conversion first
- The analysis is untargeted feature discovery rather than semi-targeted detection against a defined mz/rt library
- Acquisition is not DDA; MetaPro's documented workflow 'supports MS data acquired with the data-dependent acquisition (DDA) method'

## Inputs

- Converted LC-MS instrument data in Aird format (.aird data file plus . index file), e.g. TripleTOF dataset (8 converted instrument data / 16 files) and QE HF dataset (10 converted instrument data / 20 files)
- Analyte and internal standard library tables (.txt/.tsv with at least mz and rt columns)
- MetaPro Method parameter object (smoothing, noise estimation, peak selection, integration, screening, identification settings)

## Outputs

- Per-overview targeted detection results for all batches (per-compound peak boundaries, areas, identification)
- Quantification report exported as a named .xlsx file

## How to apply

Load converted instrument files (.aird/. index pairs produced by AirdPro) together with analyte and internal standard libraries (.txt/.tsv containing at least mz and rt columns). Extract EICs against the Aird computational format, which rearranges, compresses, and indexes spectra for rapid MS data access. Configure the Method object by choosing among 4 EIC smoothing methods, 3 noise estimation methods, 3 peak selection methods, 2 peak integration methods, 5 peak screening criteria, and 3 peak identification strategies (drawing on HMDB/METLIN-style spectral comparison). For compounds failing automated results, apply the manual hooks 'Reset integration params', 'Reset TargetRT', and 'Manual Integration' (choosing two data points as start/end boundaries) and propagate the corrected boundaries batch-wide via batch optimization. Export the quantification results of an overview into a named .xlsx report.

## Related tools

- **MetaPro** (Web-based application hosting the eight-step targeted quantification workflow, Method parameter configuration, manual inspection/re-integration, and .xlsx report export) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (Converts vendor raw files (DDA) into the Aird computation-oriented format that MetaPro's EIC extraction reads) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **Redis** (Started as part of the MetaPro runtime stack when launching the application server)
- **MongoDB** (Started as part of the MetaPro runtime stack when launching the application server)

## Examples

```
cd INSTALLATION_PATH/MetaproForUbuntu && bash MetaproStart.sh  # then open localhost:8080, Load file (.aird) + Load library (.txt/.tsv mz,rt), Set Method parameters, Target Search
```

## Evaluation signals

- The module runs end-to-end (script_runs) over all batches of both the TripleTOF and QE HF tutorial datasets without errors
- The parameterized algorithm suite exposes the exact method counts: 4 smoothing, 3 noise estimation, 3 peak selection, 2 integration, 5 screening criteria, 3 identification strategies
- A non-empty .xlsx quantification report is produced (file_exists / file_format_is checks)
- Loaders accept the tutorial .aird files and library .txt/.tsv tables containing at least mz and rt columns
- Manual operations ('Reset integration params', 'Reset TargetRT', two-point 'Manual Integration') apply corrected boundaries batch-wide

## Limitations

- MetaPro supports DDA-acquired MS data; other acquisition modes fall outside the documented targeted workflow
- Identification accuracy depends on spectral library quality, since metabolite identification via spectral database comparison is highly dependent on the quality of the spectral library
- Academic-use restriction: the distributed MetaPro version is 'for academic use only'; commercial use requires contacting the authors
- Manual inspection/re-integration remains indispensable for verifying metabolite peak quantification across hundreds of samples despite algorithmic output

## Evidence

- [methods] eight-step workflow: [evidence span withheld — anchor status: not_found]
- [methods] method counts: [evidence span withheld — anchor status: not_found]
- [methods] DDA support: "MetaPro supports MS data acquired with the data-dependent acquisition (DDA) method."
- [methods] manual re-integration hooks: "users can `Reset integration params` and `Reset TargetRT` of a compound."
- [abstract] web application scope: "MetaPro supports semi-targeted analysis workflow and functions for fast QC inspection and self-made spectral library curation with easy-to-use interfaces."
- [readme] academic-only license: "### This version is for academic use only. For commercial use, please contact us."
