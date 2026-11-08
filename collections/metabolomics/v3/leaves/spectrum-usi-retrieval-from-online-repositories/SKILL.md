---
name: spectrum-usi-retrieval-from-online-repositories
description: Use when a task references a USI such as 'mzspec:MSV000079960:...' or
  asks to reproduce/compare a spectrum hosted in a public repository (e.g., GNPS task
  MSV000079960) without local raw data files.
when_to_use_negative:
- The spectrum data is already available locally in mzML/MGF form — use a local file
  reader instead of repository USI resolution (spectrum_utils itself does not read
  spectral data files and relies on external tools or the USI mechanism).
- The repository or dataset does not support USI resolution or no network access is
  available.
- The target scan is unknown — USI retrieval requires a precise dataset accession,
  filename, and scan number.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: spectrum_utils
  role: Provides MsmsSpectrum.from_usi to load spectra from online repositories via
    USI, plus downstream processing, annotation, and mirror plotting of the retrieved
    spectra
  repo: https://github.com/bittremieux/spectrum_utils
- name: GNPS
  role: Public MS/MS reference library infrastructure hosting the datasets (e.g. MSV000079960,
    MSV000080679) resolved by USI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:293f0048ad6837faf63ded03d69d44dbcb4084d29e2d58575e38c5f3d6d08b6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - GNPS
  - Python
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spectrum-usi-retrieval-from-online-repositories

## Summary

Retrieve MS/MS spectra directly from online proteomics/metabolomics data repositories (e.g., MassIVE/GNPS) by Universal Spectrum Identifier (USI) using spectrum_utils' MsmsSpectrum.from_usi, so that remote spectra can be fetched, processed, annotated, and visualized without downloading raw files.

## When to use

Use when a task references a USI such as 'mzspec:MSV000079960:...' or asks to reproduce/compare a spectrum hosted in a public repository (e.g., GNPS task MSV000079960) without local raw data files.

## When NOT to use

- The spectrum data is already available locally in mzML/MGF form — use a local file reader instead of repository USI resolution (spectrum_utils itself does not read spectral data files and relies on external tools or the USI mechanism).
- The repository or dataset does not support USI resolution or no network access is available.
- The target scan is unknown — USI retrieval requires a precise dataset accession, filename, and scan number.

## Inputs

- Universal Spectrum Identifier (USI) string, e.g. 'mzspec:MSV000079960:DY_HS_Exp7-Ad1:scan:30372'
- Optional ProForma 2.0 peptide string for annotation (e.g. 'DLTDYLM[Oxidation]K')

## Outputs

- spectrum_utils MsmsSpectrum object with peaks (m/z and intensity arrays) ready for processing, annotation, and plotting
- Annotated mirror plot figure comparing two retrieved spectra (e.g. mirror.png)

## How to apply

Call MsmsSpectrum.from_usi with a USI string that encodes the dataset accession, filename, and scan number, e.g. 'mzspec:MSV000079960:DY_HS_Exp7-Ad1:scan:30372' or 'mzspec:MSV000080679:j11962_C1orf144:scan:10671', which resolves the spectrum from the online repository. The returned MsmsSpectrum object supports the standard spectrum_utils processing pipeline — set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity — and annotation via annotate_proforma (e.g., peptide 'DLTDYLM[Oxidation]K' with fragment_tolerance=0.5 and ion_types='aby'). Retrieved spectra can be compared by loading two USIs and passing both to spectrum_utils.plot.mirror to render the top spectrum upright and the bottom comparison spectrum inverted. This relies on the PSI-USI mechanism for spectrum loading from online proteomics and metabolomics resources, so a network connection and a repository supporting USI resolution are required.

## Related tools

- **spectrum_utils** (Provides MsmsSpectrum.from_usi to load spectra from online repositories via USI, plus downstream processing, annotation, and mirror plotting of the retrieved spectra) — https://github.com/bittremieux/spectrum_utils
- **GNPS** (Public MS/MS reference library infrastructure hosting the datasets (e.g. MSV000079960, MSV000080679) resolved by USI)

## Examples

```
from spectrum_utils.spectrum import MsmsSpectrum; spec = MsmsSpectrum.from_usi('mzspec:MSV000079960:DY_HS_Exp7-Ad1:scan:30372'); spec.annotate_proforma('DLTDYLM[Oxidation]K', 0.5, 'aby')
```

## Evaluation signals

- MsmsSpectrum.from_usi returns a MsmsSpectrum object with non-empty, finite m/z and intensity arrays for the requested scan (e.g. scan 30372 of DY_HS_Exp7-Ad1).
- After annotate_proforma with fragment_tolerance=0.5 Da and ion_types='aby', matching a/b/y fragment peaks carry annotation labels (e.g. b- and y-ion series for DLTDYLM[Oxidation]K).
- In the mirror plot, peaks shared between the top and bottom spectra (e.g. the USI pair from MSV000079960 and MSV000080679) align at the same m/z positions, indicating spectral similarity.
- The rendered figure (mirror.png, dpi=300) shows the bottom spectrum inverted below the baseline as expected of a mirror plot.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; data access relies on external tools or the USI mechanism, so raw local file parsing is out of scope.
- USI retrieval requires the dataset (e.g. MSV000079960) to be publicly resolvable in an online proteomics/metabolomics resource.
- The interactive/static plotting split means a retrieved spectrum must be re-rendered if switching between web-interactive output and publication-quality static figures.

## Evidence

- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism."
- [other] mzspec:MSV000079960:DY_HS_Exp7-Ad1:scan:30372: [evidence span withheld — anchor status: not_found]
- [methods] annotate_proforma: "Annotate peaks corresponding to a, b, and y peptide fragments in the spectrum based on a ProForma 2.0 peptide string."
- [discussion] spectrum_utils does not provide functionality to read spectral data files: [evidence span withheld — anchor status: not_found]
