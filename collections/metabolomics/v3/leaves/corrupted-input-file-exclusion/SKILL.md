---
name: corrupted-input-file-exclusion
description: Use when building an MGF input set from repository-downloaded (PRIDE/MassIVE)
  antibody MS/MS raw files and one or more converted files fail to parse as m/z-intensity
  spectra or are reported corrupted in the source study.
when_to_use_negative:
- The raw files are already in an open, parseable format (e.g. mzML) and no conversion
  step is in the pipeline — exclusion logic based on MGF parseability does not apply
- The analysis is a database-driven search rather than de novo peptide sequencing;
  corruption handling then follows the search engine's own input validation instead
  of an MGF-level pre-check
- A file is missing entirely from the repository rather than corrupted — that is a
  retrieval/inventory gap, not a corrupted-input exclusion
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ProteoWizard
  role: msConvert performs the vendor-raw-to-MGF conversion whose outputs are parsed
    and validated before corrupted files are excluded
- name: Novor
  role: Downstream de novo sequencing tool that consumes the curated, corruption-free
    MGF input set
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:3898cd83b9d1811a35574c9753fe742bdacd9b3fbee027ee7962bad347aa2a9f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Novor
  - ProteoWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# corrupted-input-file-exclusion

## Summary

Identifies and excludes corrupted or unreadable vendor MS/MS raw files (or their converted outputs) from a de novo sequencing analysis set, so that downstream tools such as Novor and Casanovo only receive parseable MGF spectra covering the intended experiments.

## When to use

Use when building an MGF input set from repository-downloaded (PRIDE/MassIVE) antibody MS/MS raw files and one or more converted files fail to parse as m/z-intensity spectra or are reported corrupted in the source study.

## When NOT to use

- The raw files are already in an open, parseable format (e.g. mzML) and no conversion step is in the pipeline — exclusion logic based on MGF parseability does not apply
- The analysis is a database-driven search rather than de novo peptide sequencing; corruption handling then follows the search engine's own input validation instead of an MGF-level pre-check
- A file is missing entirely from the repository rather than corrupted — that is a retrieval/inventory gap, not a corrupted-input exclusion

## Inputs

- Vendor-specific raw MS/MS data files for three antibody data sets (IgG1-Human, Herceptin, WIgG1-Mouse) downloaded from PRIDE and MassIVE
- ProteoWizard msConvert output files in Mascot Generic Format (MGF)
- Study experiment inventory (25 MS/MS experiments, per Supplementary Figure S1)
- Study-reported list of corrupted input files (IgG1-Human-LC Glu-C file; WIgG1-Mouse-LC trypsin file)

## Outputs

- Curated set of parseable MGF files covering the 25 evaluated MS/MS experiments minus the two excluded corrupted files
- Exclusion log recording each dropped file, its dataset/enzyme digest, and the corruption reason

## How to apply

After converting each vendor-specific raw file to Mascot Generic Format (MGF) with ProteoWizard msConvert, parse every output file to confirm it contains valid m/z-intensity pairs for multiple mass spectra. Cross-check the converted set against the study's experiment inventory (here, the 25 MS/MS experiments inventoried in Supplementary Figure S1 across the IgG1-Human, Herceptin and WIgG1-Mouse data sets) so every expected experiment is either present or explicitly excluded. Drop the files the source study itself flagged as corrupted — in this evaluation the Glu-C digest file of IgG1-Human-LC and the trypsin digest file of WIgG1-Mouse-LC — rather than silently passing them to the de novo sequencing tools. Record each exclusion (file, enzyme, dataset, reason) so the final validation rule can be stated as: MGF files for all 25 evaluated MS/MS experiments minus the two excluded corrupted files, each parseable as m/z-intensity spectrum text.

## Related tools

- **ProteoWizard** (msConvert performs the vendor-raw-to-MGF conversion whose outputs are parsed and validated before corrupted files are excluded)
- **Novor** (Downstream de novo sequencing tool that consumes the curated, corruption-free MGF input set)

## Examples

```
msconvert IgG1-Human-LC_GluC.raw --mgf -o mgf/ && head mgf/IgG1-Human-LC_GluC.mgf
```

## Evaluation signals

- Every retained .mgf file parses as text containing m/z and intensity pairs for multiple mass spectra
- The final file count equals the study's 25 MS/MS experiments minus the 2 excluded corrupted files (23 MGF files)
- Both study-reported corrupted files (IgG1-Human-LC Glu-C; WIgG1-Mouse-LC trypsin) are absent from the input set and present in the exclusion log
- No downstream de novo sequencing run emits an empty or null prediction set traceable to an unreadable input spectrum file

## Limitations

- Exclusion relies on corruption being reported by the source study or detected by MGF parseability; subtle data-quality problems (noisy spectra, missing fragmentation ions) are not caught and are known to limit all de novo sequencing tools
- Excluding a digest file (e.g. the Glu-C file of IgG1-Human-LC or the trypsin file of WIgG1-Mouse-LC) reduces enzyme-specific coverage and may lower achievable sequence coverage for the affected antibody chain

## Evidence

- [other] finding: raw files reformatted to MGF: [evidence span withheld — anchor status: not_found]
- [intro] data provenance from repositories: "The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36]."
- [other] validation rule for converted set: [evidence span withheld — anchor status: not_found]
