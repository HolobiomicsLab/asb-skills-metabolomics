---
name: proteomics-raw-file-conversion
description: Use when the input consists of vendor raw mass-spectrometry files (e.g.
  Thermo .raw downloaded from PRIDE) and a downstream clustering or search tool requires
  open peak list formats like MGF, mzML, or mzXML.
when_to_use_negative:
- Input peak lists are already in an open format (mzML, mzXML, or MGF) that the downstream
  tool reads directly
- The downstream tool requires native vendor raw files rather than converted peak
  lists
edam_operation: http://edamontology.org/operation_3258
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ThermoRawFileParser 1.2.3
  role: Converts the downloaded Thermo raw files to MGF format prior to falcon spectrum
    clustering
- name: falcon
  role: Downstream consumer of the converted MGF peak files for MS/MS spectrum clustering
  repo: https://github.com/bittremieux/falcon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:88369b44ab7012f57c826f39da00d50f0e7d9ff89a2771607eeba4ed70f0b97d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python
  - ThermoRawFileParser
  - ThermoRawFileParser 1.2.3
  - falcon
  - msCRUSH
  edam_operation: http://edamontology.org/operation_3258
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# proteomics-raw-file-conversion

## Summary

Convert vendor proprietary mass-spectrometry raw files (e.g. Thermo .raw) into open peak list formats such as MGF using ThermoRawFileParser, so that downstream tools like falcon can cluster the MS/MS spectra.

## When to use

Use when the input consists of vendor raw mass-spectrometry files (e.g. Thermo .raw downloaded from PRIDE) and a downstream clustering or search tool requires open peak list formats like MGF, mzML, or mzXML.

## When NOT to use

- Input peak lists are already in an open format (mzML, mzXML, or MGF) that the downstream tool reads directly
- The downstream tool requires native vendor raw files rather than converted peak lists

## Inputs

- Vendor proprietary MS raw files (e.g. Thermo .raw from PRIDE project PXD000561)

## Outputs

- MGF peak list files containing the MS/MS spectra
- Optionally mzML or mzXML peak files, also accepted as falcon input

## How to apply

Download the vendor raw files (in the falcon paper, the 2212 raw files of the Kim et al. draft human proteome dataset, PRIDE PXD000561) and run ThermoRawFileParser (version 1.2.3) to convert them to MGF format. The converted MGF files then serve as input peak files for falcon clustering with settings such as precursor_tol 20 ppm, fragment_tol 0.05 Da, and a cosine distance eps threshold (0.05 yields approximately 1% incorrectly clustered spectra). ThermoRawFileParser is invoked from the command line on the raw files, preserving MS/MS spectra and precursor information needed for precursor-mass-bucketed nearest neighbor searching. Validation: the converted MGF files should contain all MS/MS spectra from the raw files and should be directly readable by falcon (which also accepts mzML and mzXML).

## Related tools

- **ThermoRawFileParser 1.2.3** (Converts the downloaded Thermo raw files to MGF format prior to falcon spectrum clustering)
- **falcon** (Downstream consumer of the converted MGF peak files for MS/MS spectrum clustering) — https://github.com/bittremieux/falcon

## Examples

```
ThermoRawFileParser --input=raw/*.raw --output=mgf --format=MGF; falcon mgf/*.mgf falcon --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05
```

## Evaluation signals

- The number of MS/MS spectra in the converted MGF files matches the raw files (25 million spectra for the full draft human proteome dataset)
- falcon runs without input errors on the converted MGF files and exports cluster assignments (falcon.csv)
- All raw files from the PRIDE project (PXD000561) were converted, i.e. all 2212 peak files are present
- Precursor mass and charge metadata are preserved so falcon can partition spectra into 1 m/z buckets per precursor charge

## Limitations

- ThermoRawFileParser conversion applies to Thermo raw files; other vendor formats may need different converters
- falcon requires Python 3.8+ and is only available on Linux and OSX platforms, which constrains where the converted files can be processed

## Evidence

- [methods] converted to MGF files using ThermoRawFileParser (version 1.2.3): [evidence span withheld — anchor status: not_found]
