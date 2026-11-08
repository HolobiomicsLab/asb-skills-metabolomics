---
name: massive-dataset-retrieval
description: Use when a workflow requires the public AutoCCS demo inputs — Agilent
  6560 tune-mix frame-metadata (ImsMetadata), Features_csv feature tables, TargetList.csv,
  and autoCCS_config.xml — available under MassIVE accession MSV000085979.
when_to_use_negative:
- You need raw instrument acquisitions (e.g. UIMF files) for direct re-extraction
  rather than the packaged demo inputs — the UIMF-Library/PNNL tooling covers UIMF
  handling separately.
- Your target analytes are not Agilent tune-mix ions; the demo dataset covers tune-mix
  samples on three platforms and cannot validate platform-specific CCS behavior for
  other chemistries.
tools:
- name: AutoCCS
  role: Downstream Python tool that consumes the retrieved MassIVE demo inputs to
    compute CCS values via stepped-field, single-field, and traveling-wave methods
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: UIMF-Library
  role: PNNL companion library for reading/writing UIMF (SQLite) ion mobility mass
    spectrometry files associated with this data ecosystem
  repo: https://github.com/PNNL-Comp-Mass-Spec/UIMF-Library
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:5edf44509278f382f84a4b0a728ab8dd2244d64432043172d73c3bedfdbfd1f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - UIMF-Library
  - conda
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# massive-dataset-retrieval

## Summary

Retrieve the AutoCCS demo dataset from the MassIVE repository (MSV000085979), which packages the Agilent tune-mix inputs needed for stepped-field DTIMS, single-field RapidFire-DTIMS, and SLIM-based IMS CCS determination.

## When to use

Use when a workflow requires the public AutoCCS demo inputs — Agilent 6560 tune-mix frame-metadata (ImsMetadata), Features_csv feature tables, TargetList.csv, and autoCCS_config.xml — available under MassIVE accession MSV000085979.

## When NOT to use

- You need raw instrument acquisitions (e.g. UIMF files) for direct re-extraction rather than the packaged demo inputs — the UIMF-Library/PNNL tooling covers UIMF handling separately.
- Your target analytes are not Agilent tune-mix ions; the demo dataset covers tune-mix samples on three platforms and cannot validate platform-specific CCS behavior for other chemistries.

## Inputs

- MassIVE accession MSV000085979 (public demo dataset URL: https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000085979)
- Within the dataset: TargetList.csv target definitions
- Within the dataset: autoCCS_config.xml configuration file
- Within the dataset: ImsMetadata frame-metadata text files (per dataset)
- Within the dataset: Features_csv feature tables (per dataset)

## Outputs

- Locally downloaded AutoCCS demo data tree with SteppedField-DTIMS, SingleField-RapidFire-DTIMS, and SLIM IMS input files ready for autoCCS.py execution

## How to apply

Navigate to the MassIVE dataset page for accession MSV000085979 and download the dataset files. Locate the subfolders for the three demo platforms: data/SteppedField-DTIMS/, data/SingleField-RapidFire-DTIMS/, and the SLIM-based IMS data, each containing ImsMetadata frame-metadata text files, Features_csv feature tables, an autoCCS_config.xml configuration, and (for stepped-field) TargetList.csv. Verify the files needed by the intended AutoCCS mode are present before running — e.g. stepped-field mode needs TargetList.csv, autoCCS_config.xml, ImsMetadata/*.txt and Features_csv/*.csv globs, plus the --threshold_n_fields 5 filter. The retrieved inputs feed directly into 'python -u autoCCS.py --mode multi' (stepped-field) or '--mode single' (single-field) with results written to the corresponding Results/ directory.

## Related tools

- **AutoCCS** (Downstream Python tool that consumes the retrieved MassIVE demo inputs to compute CCS values via stepped-field, single-field, and traveling-wave methods) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **UIMF-Library** (PNNL companion library for reading/writing UIMF (SQLite) ion mobility mass spectrometry files associated with this data ecosystem) — https://github.com/PNNL-Comp-Mass-Spec/UIMF-Library

## Examples

```
curl -L -o MSV000085979.html "https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000085979"
```

## Evaluation signals

- Dataset page for accession MSV000085979 resolves and lists downloadable files.
- Downloaded tree contains the expected subpaths: ImsMetadata/*.txt and Features_csv/*.txt/.csv globs plus autoCCS_config.xml for each demo platform.
- Stepped-field folder includes TargetList.csv, required by --target_list_file for --mode multi.
- Retrieved inputs are sufficient to reproduce the documented run ending in Results/ and LogFiles/multi.log with per-ion CCS and regression fit statistics.

## Limitations

- The demo dataset covers Agilent tune-mix samples on three platforms (stepped-field DTIMS-MS, single-field RapidFire-DTIMS-MS, SLIM-based IMS) only; findings such as <0.2% differences versus vendor software apply to those cases.
- The article's discussion section content is absent, so no documented reproducibility gaps or dataset caveats can be extracted from the paper itself.

## Evidence

- [abstract] Demo datasets are publicly available at MassIVE (Dataset ID: MSV000085979).: "Demo datasets are publicly available at MassIVE (Dataset ID: MSV000085979)."
- [methods] we demonstrated the CCS determination using AutoCCS for the Agilent tune-mix samples in three different platforms: stepped-field DTIMS-MS: "we demonstrated the CCS determination using AutoCCS for the Agilent tune-mix samples in three different platforms: stepped-field DTIMS-MS"
- [readme] Demo dataset is publicly available at MassIVE (Accession: [MSV000085979](https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000085979)): "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979](https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000085979))"
