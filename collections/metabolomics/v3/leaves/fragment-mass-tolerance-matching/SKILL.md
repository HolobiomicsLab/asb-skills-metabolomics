---
name: fragment-mass-tolerance-matching
description: Use when an MS/MS spectrum for a known peptide sequence needs fragment
  peak assignments — e.g. annotating b/y ion peaks for a peptidoform loaded via a
  Universal Spectrum Identifier before building an annotation table.
when_to_use_negative:
- Input is a raw spectral data file (mzML, MGF) and you need file parsing — spectrum_utils
  does not read spectral data files and relies on external tools for this task
- The peptide sequence is unknown and peaks must be assigned de novo rather than matched
  to a specified peptidoform
- You need annotation of fragment types outside the configured ion series (e.g. only
  neutral losses not covered by the chosen ion_types)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: spectrum_utils
  role: Provides MsmsSpectrum.from_usi for spectrum loading and annotate_proforma
    for tolerance-based fragment peak annotation from a ProForma 2.0 string
  repo: https://github.com/bittremieux/spectrum_utils
- name: Python
  role: Execution environment; spectrum_utils is available for Python 3.6+
- name: Pyteomics
  role: Proteomics data processing in Python that can accompany fragment annotation
    workflows
- name: pymzML
  role: External reader for mzML spectral data that supplies input to spectrum_utils,
    which cannot read spectral files itself
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:5848f47527a32a6422da650c72a516356ffc56033f3edb9dfce590bbeb86213b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - Pyteomics
  - Python
  - pymzML
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Fragment mass tolerance matching

## Summary

Match observed MS/MS peaks to theoretical peptide fragment ions (a, b, y) within a mass tolerance (e.g. 10 ppm) using spectrum_utils' annotate_proforma, which parses ProForma 2.0 peptidoform strings including position-specific modifications.

## When to use

Use when an MS/MS spectrum for a known peptide sequence needs fragment peak assignments — e.g. annotating b/y ion peaks for a peptidoform loaded via a Universal Spectrum Identifier before building an annotation table.

## When NOT to use

- Input is a raw spectral data file (mzML, MGF) and you need file parsing — spectrum_utils does not read spectral data files and relies on external tools for this task
- The peptide sequence is unknown and peaks must be assigned de novo rather than matched to a specified peptidoform
- You need annotation of fragment types outside the configured ion series (e.g. only neutral losses not covered by the chosen ion_types)

## Inputs

- MS/MS spectrum, loadable via a Universal Spectrum Identifier (USI) such as mzspec:PXD004732:...:scan:41840
- ProForma 2.0 peptidoform string (e.g. WNQLQAFWGTGK), optionally containing position-specific modifications
- Fragment mass tolerance (e.g. 10 ppm) and ion types to match (e.g. 'aby')

## Outputs

- Peak annotations attached to the spectrum object: fragment ion type, charge, and isotope for each matched m/z
- Tabular fragment annotation table (e.g. fragment_annotations.csv) of matched peak m/z values with fragment ion type, charge, and isotope

## How to apply

Load the MS/MS spectrum from an online proteomics resource by its Universal Spectrum Identifier (e.g. mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840) using MsmsSpectrum.from_usi in spectrum_utils.spectrum. Parse the ProForma 2.0 peptidoform string (e.g. WNQLQAFWGTGK) so that the amino acid sequence, theoretical fragment ion series, and any position-specific post-translational modifications are derived. Call annotate_proforma with a fragment tolerance of 10 ppm and ion_types='aby' so observed peaks are matched to a, b, and y fragments (b/y by default), collecting each match's fragment ion type, charge, and isotope annotation. The rationale is that spectrum_utils performs the mass matching internally and efficiently, but does not read spectral files itself — data must be provided by external tools such as pymzML or fetched via USI.

## Related tools

- **spectrum_utils** (Provides MsmsSpectrum.from_usi for spectrum loading and annotate_proforma for tolerance-based fragment peak annotation from a ProForma 2.0 string) — https://github.com/bittremieux/spectrum_utils
- **Python** (Execution environment; spectrum_utils is available for Python 3.6+)
- **Pyteomics** (Proteomics data processing in Python that can accompany fragment annotation workflows)
- **pymzML** (External reader for mzML spectral data that supplies input to spectrum_utils, which cannot read spectral files itself)

## Examples

```
from spectrum_utils.spectrum import MsmsSpectrum; spectrum = MsmsSpectrum.from_usi('mzspec:PXD004732:01650b_BC2-TUM_first_pool_53_01_01-3xHCD-1h-R2:scan:41840'); spectrum.annotate_proforma('WNQLQAFWGTGK', 10, 'ppm', ion_types='aby')
```

## Evaluation signals

- Every assigned fragment in the output table corresponds to an a, b, or y fragment match of the target peptidoform (e.g. WNQLQAFWGTGK)
- Each matched peak's |observed m/z − theoretical fragment m/z| is within the 10 ppm tolerance used for annotate_proforma
- Each annotation row carries a fragment ion type, charge, and isotope annotation
- Annotated b/y ion peaks for the peptide WNQLQAFWGTGK reproduce the pattern shown in Figure 1A
- All annotations are consistent with the ProForma 2.0 string including any position-specific modifications

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; data must be supplied by external tools or via USI
- Annotation is limited to the ion types explicitly requested (b/y by default; 'aby' for a, b, y) — other fragment types are not matched
- Matches are bounded by the specified tolerance (e.g. 10 ppm); noisy spectra or incorrect tolerance can produce missing or spurious assignments

## Evidence

- [methods] Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string.: [evidence span withheld — anchor status: not_found]
- [full_text] spectrum_utils annotates spectrum peaks with their peptide fragments, optionally accounting for post-translational modifications at specified amino acid positions: [evidence span withheld — anchor status: not_found]
- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism."
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
