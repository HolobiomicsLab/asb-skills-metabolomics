---
name: mass-accuracy-drift-assessment
description: Use when repeatedly acquiring LC-MS/MS runs (e.g. untargeted metabolomics
  on a Thermo Orbitrap) with a reference standard in a well-defined matrix, and you
  need to detect in-run or across-run mass accuracy drift and RT shifts against a
  previously curated .msp annotation library.
when_to_use_negative:
- Data comes from non-Thermo vendors (Agilent, Bruker, Sciex, Waters) — Rapid QC-MS
  has only been tested extensively on Thermo acquisition sequences and RAW files and
  may contain bugs for other formats
- The analysis goal is de novo metabolite identification or feature annotation rather
  than instrument QC benchmarking against a reference library
- No reference-standard .msp library has been previously curated from well-annotated
  specimens, so there is no gold standard to compare against
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: Rapid QC-MS
  role: Hosts the Biological Standards settings, aligns current RS annotations against
    the uploaded .msp library, computes mass accuracy and RT deviation metrics, and
    writes rs_vs_msp_comparison.csv
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts raw vendor data into open mzML format before routing it to the data
    processing pipeline
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: MS-DIAL
  role: Processes mzML files to produce the RS annotations extracted from the current
    Rapid QC-MS run
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:f79c96f1f70d93fc16234051d8283a2cfd17519ecae6e7432b9126ea5deab1c8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  repo_url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - MS-DIAL
  - MSConvert
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mass-accuracy-drift-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark current LC-MS/MS instrument performance against prior analyses by aligning Rapid QC-MS reference standard (RS) annotations against a user-supplied gold-standard .msp library, quantifying mass accuracy drift and retention time shifts. It turns routine QC specimens into a longitudinal instrument-performance reference.

## When to use

Use when repeatedly acquiring LC-MS/MS runs (e.g. untargeted metabolomics on a Thermo Orbitrap) with a reference standard in a well-defined matrix, and you need to detect in-run or across-run mass accuracy drift and RT shifts against a previously curated .msp annotation library.

## When NOT to use

- Data comes from non-Thermo vendors (Agilent, Bruker, Sciex, Waters) — Rapid QC-MS has only been tested extensively on Thermo acquisition sequences and RAW files and may contain bugs for other formats
- The analysis goal is de novo metabolite identification or feature annotation rather than instrument QC benchmarking against a reference library
- No reference-standard .msp library has been previously curated from well-annotated specimens, so there is no gold standard to compare against

## Inputs

- Thermo .raw acquisition files from runs containing the reference standard
- Vendor .raw files converted to open mzML format via MSConvert
- User-supplied .msp spectral library of high-confidence RS annotations (analyte name, m/z, RT, MS2)
- Sequence .csv defining the acquisition (supplementary example files on Zenodo 10.5281/zenodo.10525183)

## Outputs

- rs_vs_msp_comparison.csv with one row per aligned RS-to-library feature match, including retention time shift and mass accuracy deviation fields
- Browser-viewable Rapid QC-MS dashboard plots of mass accuracy and retention time across specimens

## How to apply

First curate high-confidence annotations from previously annotated reference specimens into a .msp-format file combining analyte name, m/z, RT, and MS2 information, then upload it in Rapid QC-MS under Settings > Biological Standards. During each run, MSConvert converts raw vendor files to open mzML format, which MS-DIAL processes to extract RS annotations from the current Rapid QC-MS run. The current RS annotations are aligned against the .msp library annotations by matching features between the two annotation sets, and deviation metrics such as mass accuracy drift and retention time shifts are computed relative to the gold-standard entries. Results are written to rs_vs_msp_comparison.csv and surfaced on the interactive dashboard, enabling recognition of gradual drifts over hours, days, months, or years.

## Related tools

- **Rapid QC-MS** (Hosts the Biological Standards settings, aligns current RS annotations against the uploaded .msp library, computes mass accuracy and RT deviation metrics, and writes rs_vs_msp_comparison.csv) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts raw vendor data into open mzML format before routing it to the data processing pipeline) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **MS-DIAL** (Processes mzML files to produce the RS annotations extracted from the current Rapid QC-MS run) — http://prime.psc.riken.jp/compms/msdial/main.html

## Examples

```
rapidqcms
```

## Evaluation signals

- rs_vs_msp_comparison.csv exists in the output directory after the run completes
- The CSV contains one row per aligned RS-to-library feature match, not per raw feature
- The CSV includes explicit fields for retention time shift and mass accuracy deviation of current annotations relative to the gold-standard .msp entries
- Dashboard plots of m/z and RT across samples are consistent with the deviation values written to the comparison CSV
- The extracted RS annotations derive from MS-DIAL processing of mzML files converted by MSConvert, not from unconverted .raw files

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs and issues
- MSConvert and MS-DIAL are external dependencies that must be downloaded and installed manually (MS-DIAL ver.4.9.22121823; MSConvert version 3.0.23332-7faa7f422)
- Full functionality requires Windows because of the MSConvert dependency; MacOS users can still use Rapid QC-MS to monitor / view their instrument run data

## Evidence

- [other] RS benchmarking workflow computes drift metrics: [evidence span withheld — anchor status: not_found]
