---
name: proteomics-repository-dataset-retrieval
description: Use when a downstream analysis (e.g. de novo sequencing with Novor, Casanovo,
  PointNovo) needs raw antibody MS/MS data in an open text format and the data must
  be fetched from PRIDE or MassIVE.
when_to_use_negative:
- The data are already available in an open, parseable format (mzML or MGF) and only
  need to be downloaded, not reformatted
- The downstream tool requires vendor-native or mzML input rather than MGF (e.g. tools
  that consume mzML directly)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ProteoWizard
  role: msConvert is used to reformat vendor-specific raw MS/MS files into Mascot
    Generic Format (MGF) text files of m/z-intensity pairs
- name: PRIDE
  role: Proteomics MS data repository from which the antibody raw datasets were downloaded
- name: MassIVE
  role: Proteomics MS data repository from which the antibody raw datasets were downloaded
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:d0850e6445b68330b6c69543461e38a7a5bd3e36b0710cc8851451e0ba3fc1d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MassIVE
  - Novor
  - PRIDE
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# proteomics-repository-dataset-retrieval

## Summary

Retrieve raw MS/MS antibody datasets from the PRIDE and MassIVE proteomics repositories and reformat vendor-specific raw files into open Mascot Generic Format (MGF) via ProteoWizard, yielding parseable m/z-intensity text files that can serve as input for de novo peptide sequencing tools.

## When to use

Use when a downstream analysis (e.g. de novo sequencing with Novor, Casanovo, PointNovo) needs raw antibody MS/MS data in an open text format and the data must be fetched from PRIDE or MassIVE.

## When NOT to use

- The data are already available in an open, parseable format (mzML or MGF) and only need to be downloaded, not reformatted
- The downstream tool requires vendor-native or mzML input rather than MGF (e.g. tools that consume mzML directly)

## Inputs

- PRIDE or MassIVE repository accession/URLs for the antibody datasets (IgG1-Human, Herceptin, WIgG1-Mouse)
- Vendor-specific raw MS/MS data files downloaded from the repositories
- List of expected MS/MS experiments per dataset (e.g. from Supplementary Figure S1)

## Outputs

- Downloaded vendor raw MS/MS files for all repository datasets
- Converted .mgf text files storing m/z and intensity pairs of multiple mass spectra, one per non-corrupted experiment

## How to apply

Identify the target antibody datasets — in this study IgG1-Human, Herceptin and WIgG1-Mouse — and download the raw MS/MS files from the PRIDE and MassIVE repositories. Convert each vendor-specific raw file to Mascot Generic Format (MGF) using ProteoWizard msConvert, producing text files that store m/z and intensity pairs for multiple mass spectra. Inventory the converted files against the full experiment list (25 MS/MS experiments across the multi-enzymatic digests, as inventoried in Supplementary Figure S1) and exclude known corrupted inputs — the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC. Validate that every output .mgf file parses as m/z-intensity spectrum pairs before feeding the set to de novo sequencing algorithms.

## Related tools

- **ProteoWizard** (msConvert is used to reformat vendor-specific raw MS/MS files into Mascot Generic Format (MGF) text files of m/z-intensity pairs)
- **PRIDE** (Proteomics MS data repository from which the antibody raw datasets were downloaded)
- **MassIVE** (Proteomics MS data repository from which the antibody raw datasets were downloaded)

## Examples

```
msconvert IgG1-Human.raw --mgf -o converted/
```

## Evaluation signals

- The retrieved and converted set comprises MGF files for all 25 evaluated MS/MS experiments minus the two excluded corrupted files
- Each output .mgf file is parseable as m/z-intensity spectrum text
- All three antibody datasets (IgG1-Human, Herceptin, WIgG1-Mouse) are represented in the converted set
- The two known corrupted files (Glu-C file of IgG1-Human-LC, trypsin file of WIgG1-Mouse-LC) are explicitly excluded
- Conversion covers every enzymatic digest variant inventoried in Supplementary Figure S1

## Limitations

- Some repository files may be corrupted; the study reported two unusable inputs (the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC) that had to be excluded
- MGF conversion discards vendor-native metadata and yields plain m/z-intensity text, which may not suit tools requiring richer formats

## Evidence

- [intro] The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36].: "The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36]."
- [methods] The raw MS/MS data files were reformatted to Mascot Generic Format (MGF) using ProteoWizard; MGF stores the m/z and intensity pairs of multiple mass spectra in a single text format.: [evidence span withheld — anchor status: not_found]
- [supplementary] Exclude the two corrupted input files reported in the study (the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC) from the converted set.: [evidence span withheld — anchor status: not_found]
