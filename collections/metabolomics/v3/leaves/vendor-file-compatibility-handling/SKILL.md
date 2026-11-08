---
name: vendor-file-compatibility-handling
description: Use when raw MS/MS data downloaded from PRIDE or MassIVE come in vendor-specific
  proprietary formats that de novo sequencing tools cannot read directly, and spectra
  must be converted to MGF before peptide prediction or assembly.
when_to_use_negative:
- The input files are already in MGF or another open format (e.g. mzML) readable by
  the downstream de novo sequencing tool — re-conversion adds no value.
- The analysis requires spectral metadata (precursor charge, retention time, instrument
  fields) that a minimal m/z-intensity MGF export may not preserve faithfully for
  a particular tool.
- The target pipeline is a database-search engine that accepts vendor raw files or
  mzML directly and does not need MGF.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ProteoWizard
  role: msConvert performs the vendor-format-to-MGF conversion of the raw MS/MS files
- name: PRIDE
  role: source repository from which the raw antibody MS/MS data sets are downloaded
- name: MassIVE
  role: source repository from which the raw antibody MS/MS data sets are downloaded
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:e270cd251da9497469fc86dd70eced65d0d047ed0585d24f7d38d3466d67e6a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MassIVE
  - Novor
  - PRIDE
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# vendor-file-compatibility-handling

## Summary

Converts vendor-specific raw MS/MS files into an open, tool-agnostic text format (Mascot Generic Format, MGF) using ProteoWizard, so that de novo peptide sequencing tools such as Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo can consume the spectra. It is the file-format compatibility bridge between proteomics repositories (PRIDE, MassIVE) and downstream algorithm pipelines.

## When to use

Use when raw MS/MS data downloaded from PRIDE or MassIVE come in vendor-specific proprietary formats that de novo sequencing tools cannot read directly, and spectra must be converted to MGF before peptide prediction or assembly.

## When NOT to use

- The input files are already in MGF or another open format (e.g. mzML) readable by the downstream de novo sequencing tool — re-conversion adds no value.
- The analysis requires spectral metadata (precursor charge, retention time, instrument fields) that a minimal m/z-intensity MGF export may not preserve faithfully for a particular tool.
- The target pipeline is a database-search engine that accepts vendor raw files or mzML directly and does not need MGF.

## Inputs

- Vendor-specific raw MS/MS data files for antibody digests (IgG1-Human, Herceptin, WIgG1-Mouse) downloaded from the PRIDE and MassIVE proteomics repositories

## Outputs

- MGF text files, one per MS/MS experiment, each parseable as m/z-intensity pairs for multiple mass spectra

## How to apply

Retrieve the raw MS/MS files for the antibody data sets (e.g. IgG1-Human, Herceptin, WIgG1-Mouse) from PRIDE and MassIVE, then run ProteoWizard msConvert on each file to produce Mascot Generic Format (MGF) output — a single text format storing m/z and intensity pairs for multiple mass spectra. Keep the conversion tied to the study's experiment inventory (the 25 MS/MS experiments enumerated in Supplementary Figure S1) so no experiment is silently dropped. Exclude known-corrupted inputs during conversion — in this study the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC — rather than letting them fail silently downstream. The rationale is that MGF's plain-text m/z-intensity structure is the common input format accepted across the evaluated de novo sequencing algorithms, making the conversion a prerequisite for a fair, tool-independent benchmark.

## Related tools

- **ProteoWizard** (msConvert performs the vendor-format-to-MGF conversion of the raw MS/MS files)
- **PRIDE** (source repository from which the raw antibody MS/MS data sets are downloaded)
- **MassIVE** (source repository from which the raw antibody MS/MS data sets are downloaded)

## Examples

```
msconvert IgG1-Human_raw.raw --mgf -o converted_mgf
```

## Evaluation signals

- The converted set comprises MGF files covering all 25 MS/MS experiments inventoried in Supplementary Figure S1, minus the two excluded corrupted files (IgG1-Human-LC Glu-C and WIgG1-Mouse-LC trypsin).
- Each output .mgf file parses as plain text containing m/z-intensity pairs for multiple mass spectra.
- Every vendor raw file retrieved from PRIDE/MassIVE for the three antibody data sets has a corresponding converted MGF output (no silent drops).
- The two known-corrupted input files are explicitly excluded from the converted set rather than producing empty or malformed MGF files.

## Limitations

- Corrupted repository inputs can block the pipeline — this study had to exclude the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC.
- MGF stores only m/z and intensity pairs; vendor-specific spectral metadata may be lost in conversion, and MGF must be re-verified as parseable after each conversion.

## Evidence

- [methods] MGF conversion finding: [evidence span withheld — anchor status: not_found]
- [intro] Data source: "The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36]."
- [abstract] Tool panel consuming MGF: "we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo)"
