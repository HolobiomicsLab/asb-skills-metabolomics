---
name: mgf-spectrum-format-generation
description: Use when raw vendor MS/MS files (e.g. from PRIDE or MassIVE antibody
  data sets) must be reformatted into an open, text-based spectral format as input
  for de novo peptide sequencing or database-search tools.
when_to_use_negative:
- The spectra are already in an open format (mzML or MGF) that the downstream de novo
  sequencing tool accepts directly — re-conversion would only risk loss or corruption
- The analysis requires peak picking, deconvolution, or recalibration of the raw data
  — msConvert formatting alone does not perform those processing steps
- The downstream tool natively reads vendor raw formats and conversion would discard
  vendor-specific metadata needed later
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ProteoWizard
  role: msConvert converts each vendor-specific raw MS/MS file into Mascot Generic
    Format (MGF) text output of m/z–intensity spectrum pairs
- name: Novor
  role: Downstream consumer of the generated .mgf files for de novo peptide sequencing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:f398cf5968989603d7f5ae4328ba4a581cc35b38a50b4fbc202c25fe3b5d56c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Novor
  - ProteoWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mgf-spectrum-format-generation

## Summary

Converts vendor-specific raw MS/MS files of antibody proteomics data sets into Mascot Generic Format (MGF) text files using ProteoWizard msConvert, so that de novo peptide sequencing tools (e.g. Novor, Casanovo) can ingest the spectra. MGF stores the m/z and intensity pairs of multiple mass spectra in a single parseable text format.

## When to use

Use when raw vendor MS/MS files (e.g. from PRIDE or MassIVE antibody data sets) must be reformatted into an open, text-based spectral format as input for de novo peptide sequencing or database-search tools.

## When NOT to use

- The spectra are already in an open format (mzML or MGF) that the downstream de novo sequencing tool accepts directly — re-conversion would only risk loss or corruption
- The analysis requires peak picking, deconvolution, or recalibration of the raw data — msConvert formatting alone does not perform those processing steps
- The downstream tool natively reads vendor raw formats and conversion would discard vendor-specific metadata needed later

## Inputs

- Vendor-specific raw MS/MS data files downloaded from PRIDE and MassIVE (IgG1-Human, Herceptin, WIgG1-Mouse antibody data sets)
- List of corrupted/known-bad input files to exclude (e.g. Glu-C file of IgG1-Human-LC, trypsin file of WIgG1-Mouse-LC)

## Outputs

- One .mgf (Mascot Generic Format) text file per converted MS/MS experiment, containing m/z and intensity pairs for multiple mass spectra
- A converted-file inventory covering the 25 evaluated MS/MS experiments minus excluded corrupted files

## How to apply

Retrieve the raw MS/MS data files for the antibody data sets (IgG1-Human, Herceptin, WIgG1-Mouse) from the PRIDE and MassIVE proteomics repositories. Run ProteoWizard msConvert on each vendor-specific raw file to emit Mascot Generic Format (.mgf) output — text files that store m/z and intensity pairs for multiple mass spectra. Exclude corrupted input files identified upstream, in this study the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC. Verify conversion completeness against the study's experiment inventory (25 MS/MS experiments, as inventoried in Supplementary Figure S1): the converted set must comprise MGF files for all evaluated experiments minus the excluded corrupted files, and each output .mgf must be parseable as m/z–intensity spectrum text before feeding it to de novo sequencing tools.

## Related tools

- **ProteoWizard** (msConvert converts each vendor-specific raw MS/MS file into Mascot Generic Format (MGF) text output of m/z–intensity spectrum pairs)
- **Novor** (Downstream consumer of the generated .mgf files for de novo peptide sequencing)

## Examples

```
msconvert IgG1-Human.raw --mgf -o converted/
```

## Evaluation signals

- The converted set comprises MGF files for all 25 evaluated MS/MS experiments minus the two excluded corrupted files (Glu-C IgG1-Human-LC, trypsin WIgG1-Mouse-LC)
- Every output .mgf file parses as m/z–intensity spectrum text without errors
- Each of the three antibody data sets (IgG1-Human, Herceptin, WIgG1-Mouse) is represented in the converted set
- Downstream de novo sequencing tools (e.g. Novor, Casanovo) accept the .mgf inputs without format errors

## Limitations

- Two corrupted input files in the study (the Glu-C file of IgG1-Human-LC and the trypsin file of WIgG1-Mouse-LC) had to be excluded from the converted set — raw files should be checked for corruption before or after conversion
- Conversion is format reformatting only; MGF carries m/z and intensity pairs and does not preserve richer vendor metadata
- No version numbers for the tools are stated in the study, so conversion parameters are not fully reproducible from the article alone

## Evidence

- [methods] Raw-to-MGF conversion with ProteoWizard: [evidence span withheld — anchor status: not_found]
- [intro] Data provenance: PRIDE and MassIVE: "The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36]."
