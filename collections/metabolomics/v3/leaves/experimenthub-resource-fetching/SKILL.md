---
name: experimenthub-resource-fetching
description: Use when a protocol needs a small, citable Thermo Fisher .raw example
  file for demonstration or testing and the article pins it to an ExperimentHub accession
  (EH4547) or a MassIVE dataset (MSV000086542).
when_to_use_negative:
- The analysis targets user-supplied or proprietary .raw files that are not registered
  in ExperimentHub or MassIVE — fetch them directly instead.
- The goal is conversion of raw data to exchange formats like mzML via ProteoWizard/MSconvert
  rather than obtaining a curated example dataset.
- Downstream tools require non-raw formats (e.g. mzML) that the EH4547 resource does
  not provide.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ExperimentHub
  role: Bioconductor client used to fetch and cache the example raw file resource
    EH4547 and resolve its local path
- name: tartare
  role: Bioconductor data package providing additional raw data for demonstration
    and extended testing
- name: rawR
  role: Consumer of the fetched raw file; its readSpectrum/readChromatogram calls
    take the normalized EH4547 path as input
  repo: https://github.com/fgcz/rawR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.10.30.362533
    title: rawR - Direct access to raw mass spectrometry data in R
  source_content_hash: sha256:1665cd150a69d2af140d712032c1f7fd5c8b2f94d6a4faec7fdb0caf7053cc24
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/fgcz/rawR
  tools:
  - ExperimentHub
  - R
  - rawR
  - tartare
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# experimenthub-resource-fetching

## Summary

Fetches reproducible example raw mass spectrometry data (e.g. 20181113_010_autoQC01.raw from MassIVE MSV000086542 or resource EH4547) through the Bioconductor ExperimentHub framework so downstream rawrr workflows start from a checksum-verified, locally normalized file path.

## When to use

Use when a protocol needs a small, citable Thermo Fisher .raw example file for demonstration or testing and the article pins it to an ExperimentHub accession (EH4547) or a MassIVE dataset (MSV000086542).

## When NOT to use

- The analysis targets user-supplied or proprietary .raw files that are not registered in ExperimentHub or MassIVE — fetch them directly instead.
- The goal is conversion of raw data to exchange formats like mzML via ProteoWizard/MSconvert rather than obtaining a curated example dataset.
- Downstream tools require non-raw formats (e.g. mzML) that the EH4547 resource does not provide.

## Inputs

- ExperimentHub accession identifier (e.g. EH4547)
- MassIVE dataset identifier (e.g. MSV000086542)
- expected MD5 checksum for the example raw file (a1f5df9627cf9e0d51ec1906776957ab)

## Outputs

- Locally cached, checksum-verified Thermo Fisher .raw file (20181113_010_autoQC01.raw)
- Normalized absolute file path suitable for rawrr::readSpectrum / rawrr::readChromatogram

## How to apply

Load the ExperimentHub library and retrieve the pinned resource by accession — in the rawR paper the example Thermo Fisher raw file is EH4547 — then normalize the cached path with normalizePath(eh[['EH4547']]) so downstream rawrr::readSpectrum and rawrr::readChromatogram calls receive an absolute file path. Verify the fetched file's integrity with its published MD5 checksum (a1f5df9627cf9e0d51ec1906776957ab for 20181113_010_autoQC01.raw) before analysis. As an alternative provenance route, the file can be downloaded from the MassIVE dataset MSV000086542, and additional demonstration/test raw data is available through the Bioconductor data package tartare. Decision point: use ExperimentHub when an R-session-resolved cached path is preferred; use the MassIVE/USI route (e.g. mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2) when public-repository provenance must be cited directly.

## Related tools

- **ExperimentHub** (Bioconductor client used to fetch and cache the example raw file resource EH4547 and resolve its local path)
- **tartare** (Bioconductor data package providing additional raw data for demonstration and extended testing)
- **rawR** (Consumer of the fetched raw file; its readSpectrum/readChromatogram calls take the normalized EH4547 path as input) — https://github.com/fgcz/rawR

## Examples

```
library(ExperimentHub); eh <- ExperimentHub(); rawfile <- normalizePath(eh[["EH4547"]])
```

## Evaluation signals

- The retrieved file's MD5 matches the published checksum a1f5df9627cf9e0d51ec1906776957ab.
- eh[['EH4547']] resolves to an existing file and normalizePath() returns a valid absolute path on disk.
- Downstream rawrr::readSpectrum on the fetched file returns an object of class rawrrSpectrumSet (e.g. scan 9594 is accessible).
- The fetched file matches the 20181113_010_autoQC01.raw example whose 35 min LC-MS run produced 2.1881 x 10^4 scans.
- USI-style provenance (mzspec:MSV000086542:20181113_010_autoQC01:scan:9594:LGGNEQVTR/2) resolves to the same underlying dataset.

## Limitations

- ExperimentHub resources are pinned to specific accessions; if EH4547 is unavailable, the fallback is manual download from MassIVE MSV000086542.
- The rawrr package itself cannot run until the rawrr executable runtime is installed with rawrr::installRawrrExe() — fetching the data alone does not enable spectrum reading.
- On Windows, the decimal symbol must be configured as '.' for mZ and intensity vectors parsed from the raw file to have correct lengths.

## Evidence
