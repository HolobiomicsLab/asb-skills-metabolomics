---
name: raw-msms-file-format-conversion
description: Use when raw MS/MS files from proteomics repositories (PRIDE, MassIVE)
  are vendor-proprietary and must be reformatted to an open, parseable format (MGF)
  before feeding de novo sequencing tools such as Novor.
when_to_use_negative:
- Input files are already in MGF or another open, parseable format — reconversion
  adds no value
- An input file is known to be corrupted (e.g. the Glu-C file of IgG1-Human-LC or
  the trypsin file of WIgG1-Mouse-LC) — exclude it rather than attempting conversion
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ProteoWizard
  role: msConvert performs the vendor-raw-to-MGF format conversion of each MS/MS experiment
- name: Novor
  role: Downstream consumer of the converted MGF files for de novo peptide sequencing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:21c697e3cf365b85d870612c2f1c186c3b75ab081bb4fdc5836e93c5beeeb903
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Novor
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# raw-msms-file-format-conversion

## Summary

Converts vendor-proprietary raw MS/MS files from proteomics repositories (e.g. PRIDE, MassIVE) into Mascot Generic Format (MGF) text using ProteoWizard msConvert, so that de novo peptide sequencing tools like Novor can consume the spectra. MGF stores the m/z and intensity pairs of multiple mass spectra in a single open text format.

## When to use

Use when raw MS/MS files from proteomics repositories (PRIDE, MassIVE) are vendor-proprietary and must be reformatted to an open, parseable format (MGF) before feeding de novo sequencing tools such as Novor.

## When NOT to use

- Input files are already in MGF or another open, parseable format — reconversion adds no value
- An input file is known to be corrupted (e.g. the Glu-C file of IgG1-Human-LC or the trypsin file of WIgG1-Mouse-LC) — exclude it rather than attempting conversion

## Inputs

- Vendor-specific raw MS/MS data files (three antibody data sets: IgG1-Human, Herceptin, WIgG1-Mouse) downloaded from PRIDE and MassIVE

## Outputs

- MGF text files storing m/z and intensity pairs for multiple mass spectra (one .mgf per MS/MS experiment)

## How to apply

Retrieve the raw MS/MS data files from PRIDE and MassIVE repositories. Run ProteoWizard msConvert on each vendor-specific raw file to produce .mgf text files that store m/z and intensity pairs for multiple mass spectra. Exclude corrupted inputs — in the antibody study these were the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC — so downstream tools never receive unparseable spectra. Cross-check the converted set against the full experiment inventory (25 MS/MS experiments per Supplementary Figure S1) and verify each output file parses as m/z–intensity pairs before passing files to Novor or other de novo sequencing algorithms.

## Related tools

- **ProteoWizard** (msConvert performs the vendor-raw-to-MGF format conversion of each MS/MS experiment)
- **Novor** (Downstream consumer of the converted MGF files for de novo peptide sequencing)

## Examples

```
msconvert IgG1-Human.raw --mgf -o converted/
```

## Evaluation signals

- The converted set comprises MGF files for all 25 evaluated MS/MS experiments (per Supplementary Figure S1) minus the two excluded corrupted files
- Every output .mgf file is parseable as m/z–intensity spectrum text
- Files span all three antibody data sets: IgG1-Human, Herceptin, WIgG1-Mouse
- No corrupted source files (IgG1-Human-LC Glu-C, WIgG1-Mouse-LC trypsin) appear in the converted set

## Limitations

- Two input files reported in the study are corrupted (the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC) and must be excluded from the converted set
- The study does not state version numbers for the tools used, so a reproducible conversion pipeline must pin ProteoWizard versions independently

## Evidence

- [other] MGF conversion finding: [evidence span withheld — anchor status: not_found]
- [intro] Data retrieval: "The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36]."
