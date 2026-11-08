---
name: ms-peak-list-conversion-to-magma-format
description: Use when starting from flat CASMI-style MS/MS peak lists (precursor plus
  fragment m/z values at one or more collision energies) that must be loaded into
  MAGMa command-line or via the www.emetabolomics.org/magma web interface for candidate
  annotation.
when_to_use_negative:
- Data is already MSn (n>2) multistage fragmentation tree data that MAGMa natively
  ingests — this conversion targets flat MS/MS peak lists
- Peak lists are already in MAGMa hierarchical input format, in which case the next
  step is PubChem candidate retrieval rather than re-conversion
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MAGMa
  role: Target annotation software whose input files are produced by this conversion
    step; used in command-line mode or via web interface
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Not used in this conversion itself, but the local PubChem compound database
    (2 December 2013 snapshot) is queried downstream using precursor masses from the
    converted file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:d684f7c86f3807b7bf261bd1ca91023633c565515c5ba11f9bc93dbe6ac350e9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# MS peak-list conversion to MAGMa format

## Summary

Convert contest or instrument-provided MS/MS peak lists into MAGMa's hierarchical input format, where each precursor peak carries nested fragment peaks so candidates can be scored by substructure penalty. This is the mandatory first step before retrieving PubChem candidates and running in silico fragmentation.

## When to use

Use when starting from flat CASMI-style MS/MS peak lists (precursor plus fragment m/z values at one or more collision energies) that must be loaded into MAGMa command-line or via the www.emetabolomics.org/magma web interface for candidate annotation.

## When NOT to use

- Data is already MSn (n>2) multistage fragmentation tree data that MAGMa natively ingests — this conversion targets flat MS/MS peak lists
- Peak lists are already in MAGMa hierarchical input format, in which case the next step is PubChem candidate retrieval rather than re-conversion

## Inputs

- CASMI-style MS/MS peak lists (precursor peak and fragment m/z and intensity values)
- MS/MS spectra acquired at different collision energies to be merged into composite spectra

## Outputs

- MAGMa hierarchical input files with precursor peaks and nested fragment peaks (spectral trees)
- Composite spectra in which duplicate fragment ions across collision energies retain the maximum intensity

## How to apply

First take the peak lists supplied for each challenge (e.g. CASMI 2013 contest data) and convert them to MAGMa input files, whose format is an extension of commonly used peak lists: the precursor peak is written with fragment peaks nested in parentheses, forming a hierarchical spectral tree. When data contains spectra acquired at different collision energies, merge them into a composite spectrum before conversion, taking the highest intensity for ions present at both energies. Validate the resulting file by feeding it to MAGMa in command-line mode or through the web interface and checking that each precursor is associated with its correct fragment m/z values. The rationale is that MAGMa matches m/z values of the MS/MS peaks against in-silico generated substructures, so any precursor-fragment nesting lost during conversion corrupts the penalty scoring of downstream candidate ranking.

## Related tools

- **MAGMa** (Target annotation software whose input files are produced by this conversion step; used in command-line mode or via web interface) — https://github.com/NLeSC/MAGMa
- **PubChem** (Not used in this conversion itself, but the local PubChem compound database (2 December 2013 snapshot) is queried downstream using precursor masses from the converted file)

## Examples

```
magmaweb run --input challenge.magma  # after manually converting CASMI peak list (precursor with nested fragment peaks)
```

## Evaluation signals

- Each precursor in the converted MAGMa input file is annotated with its full set of fragment peaks from the original peak list
- Duplicate fragment ions across collision energies are represented once in the composite spectrum with the highest observed intensity
- MAGMa accepts the file without input-format errors and lists the expected number of fragment peaks per candidate scoring run
- Downstream candidate retrieval runs use the precursor masses from the converted file, e.g. within 10 ppm deviation (absolute minimum 0.002 Da)

## Limitations

- Conversion requires manual construction of the composite spectrum when multiple collision energies are present; the article does not document an automated merge tool
- The paper states peak lists 'were first converted to MAGMa input files' without specifying an exact parser, so edge cases in peak ordering or delimiter handling are not documented
- Too-restrictive mass tolerances downstream (e.g. 0.002 Da minimum) made challenge 11 peaks appear unexplained, so conversion quality alone does not guarantee correct annotation

## Evidence

- [results] The input data was constructed by generating composite spectra of the MS/MS spectra obtained at the different collision energies: [evidence span withheld — anchor status: not_found]
