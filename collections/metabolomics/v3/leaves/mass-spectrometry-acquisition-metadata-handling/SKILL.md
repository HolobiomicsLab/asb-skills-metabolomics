---
name: mass-spectrometry-acquisition-metadata-handling
description: Use when a study provides proprietary Thermo .raw files plus a sample-metadata
  CSV and centroided mzML inputs are required for downstream LC-MS preprocessing (e.g.,
  Asari in pcpfm).
when_to_use_negative:
- The input data are already .mzML files — pcpfm skips this step entirely and proceeds
  directly to preprocessing.
- Acquisitions come from non-Thermo instruments, since ThermoRawFileParser targets
  Thermo .raw files; the README notes '.raw files but support is limited'.
- A feature table already exists and no raw-level reprocessing is needed — this skill
  only produces conversion-ready mzML inputs.
edam_operation: http://edamontology.org/operation_3254
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ThermoRawFileParser
  role: Converts proprietary Thermo .raw acquisitions to centroided .mzML (invoked
    via a placeholder-substituted command, default -f=1 for peak picking).
- name: mono
  role: Runtime used to execute the ThermoRawFileParser binary in the default conversion
    command.
- name: pcpfm
  role: Pipeline that orchestrates the conversion step, reads the metadata CSV, writes
    .mzML files into converted_acquisitions/, and validates one output per .raw acquisition.
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python
  role: Language in which the pipeline drives the per-sample conversion loop and the
    file_exists validation check.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:1eb90513dab69b5b899cb9b8bc5f5796179c70ea0b9aa1fc97bd9da97c9ec8b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Python
  - ThermoRawFileParser
  - mono
  - pcpfm
  edam_operation: http://edamontology.org/operation_3254
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# mass-spectrometry-acquisition-metadata-handling

## Summary

Converts proprietary Thermo .raw LC-MS acquisitions into centroided .mzML files using ThermoRawFileParser, driven by an experiment metadata CSV that maps sample names to raw file paths, so the pcpfm pipeline can proceed to Asari preprocessing.

## When to use

Use when a study provides proprietary Thermo .raw files plus a sample-metadata CSV and centroided mzML inputs are required for downstream LC-MS preprocessing (e.g., Asari in pcpfm).

## When NOT to use

- The input data are already .mzML files — pcpfm skips this step entirely and proceeds directly to preprocessing.
- Acquisitions come from non-Thermo instruments, since ThermoRawFileParser targets Thermo .raw files; the README notes '.raw files but support is limited'.
- A feature table already exists and no raw-level reprocessing is needed — this skill only produces conversion-ready mzML inputs.

## Inputs

- Experiment metadata CSV mapping sample names to .raw acquisition file paths (minimal fields: sample name and file path)
- Thermo .raw mass spectrometry acquisition files
- User-supplied conversion command template with $RAW_PATH and $OUT_PATH placeholders (default: 'mono ThermoRawFileParser.exe -f=1 -i $RAW_PATH -b $OUT_PATH')

## Outputs

- One centroided .mzML file per .raw acquisition, written to converted_acquisitions/ (e.g., sample1.mzML, sample2.mzML)
- Validated 1:1 sample-to-mzML mapping ready for Asari preprocessing in the pcpfm experiment directory

## How to apply

Load the experiment metadata CSV that maps each sample name to its .raw acquisition filepath, then for each entry substitute the raw filepath and output filepath into a user-configurable conversion command via the $RAW_PATH and $OUT_PATH placeholders; the pcpfm default is 'mono ThermoRawFileParser.exe -f=1 -i $RAW_PATH -b $OUT_PATH', where the -f=1 flag performs peak picking so the output is centroided. Write each resulting .mzML into the experiment's converted_acquisitions subdirectory (e.g., sample1.mzML, sample2.mzML), preserving the sample-name-to-filename correspondence. If .mzML files were supplied instead of .raw, skip conversion entirely. Validate that exactly one centroided .mzML exists in converted_acquisitions for every .raw listed in the metadata CSV before invoking preprocessing.

## Related tools

- **ThermoRawFileParser** (Converts proprietary Thermo .raw acquisitions to centroided .mzML (invoked via a placeholder-substituted command, default -f=1 for peak picking).)
- **mono** (Runtime used to execute the ThermoRawFileParser binary in the default conversion command.)
- **pcpfm** (Pipeline that orchestrates the conversion step, reads the metadata CSV, writes .mzML files into converted_acquisitions/, and validates one output per .raw acquisition.) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python** (Language in which the pipeline drives the per-sample conversion loop and the file_exists validation check.)

## Examples

```
mono ThermoRawFileParser.exe -f=1 -i $RAW_PATH -b $OUT_PATH
```

## Evaluation signals

- file_exists check passes: exactly one centroided .mzML file in converted_acquisitions for every .raw acquisition listed in the metadata CSV.
- Each output .mzML filename corresponds to its sample name — a sample-name/mzML-filename mismatch bug existed before 2/28/24 and must be verified as fixed.
- Spectra in the output are centroided (peak-picked), consistent with the default '-f=1' ThermoRawFileParser flag.
- The experiment directory shows the expected structure: converted_acquisitions/ populated alongside raw_acquisitions/, asari_results/, and experiment.json.
- No conversion is attempted when .mzML inputs were supplied (step skipped, not duplicated).

## Limitations

- Support for .raw input is limited; the README recommends 'properly formatted metadata sheets' and notes 'You can work with .raw files but support is limited.'
- A historical bug caused sample names that do not match their mzML file names to break the pipeline; fixed as of 2/28/24 per VERSION_LOG.
- The default invocation requires mono to run the ThermoRawFileParser executable, and only Thermo .raw files are handled.

## Evidence

- [readme] Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path).: "Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path)."
- [readme] convert Thermo .raw to mzML (ThermoRawFileParser): "convert Thermo .raw to mzML (ThermoRawFileParser)"
- [readme] there was an issue regarding sample names that do not match their mzML file names. This has been fixed as of 2/28/24.: "there was an issue regarding sample names that do not match their mzML file names. This has been fixed as of 2/28/24."
- [abstract] Our pipeline takes experimental data in .mzML or .raw format and outputs annotated feature tables for subsequent biological interpretation.: "Our pipeline takes experimental data in .mzML or .raw format and outputs annotated feature tables for subsequent biological interpretation."
- [readme] You can work with .raw files but support is limited.: "You can work with .raw files but support is limited."
