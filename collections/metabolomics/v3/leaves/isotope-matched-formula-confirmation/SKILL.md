---
name: isotope-matched-formula-confirmation
description: Use when assigning molecular formulas to ultrahigh-resolution mass spectra
  (FT-ICR/Orbitrap) where 13C and 34S polyisotopic peaks must be separated from monoisotopic
  masses before or after formula assignment.
when_to_use_negative:
- When the input spectrum has not been recalibrated and isotope-pair mass differences
  exceed the +/- 5 ppm window — run Recal() first
- When you need definitive proof of the presence or absence of 13C or 34S in a formula
  — the README states additional work would be needed and IsoFiltR should not be considered
  definitive
- For MS/MS or sparse spectra where the four-step KMD-series logic breaks down due
  to low continuity
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MFAssignR
  role: Package providing IsoFiltR isotope filtering and MFAssign isotope-mass matching
    within the formula assignment pipeline
  repo: https://github.com/skschum/MFAssignR
- name: IsoFiltR
  role: Identifies prospective 13C and 34S isotope ions via the four-step mass-difference,
    KMD, resolution-enhanced KMD, and abundance-ratio method
  repo: https://github.com/skschum/MFAssignR
- name: MFAssign
  role: Assigns monoisotopic MF and matches assigned monoisotopic masses to corresponding
    isotopic masses from the Iso output
  repo: https://github.com/skschum/MFAssignR
- name: KMDNoise
  role: Establishes the S/N threshold applied before isotope filtering and assignment
  repo: https://github.com/skschum/MFAssignR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.envres.2020.110114
    title: 'MFAssignR: Molecular formula assignment software for ultrahigh resolution
      mass spectrometry analysis of environmental complex mixtures'
  source_content_hash: sha256:d75fcbf35a5fe86fef5f3033c37326afde38bd454cf99fd6699f60350f291495
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skschum/MFAssignR
  tools:
  - IsoFiltR
  - KMDNoise
  - MFAssign
  - MFAssignCHO
  - MFAssignR
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# isotope-matched-formula-confirmation

## Summary

Match molecular formula assignments to their corresponding 13C and 34S isotope masses using MFAssignR's IsoFiltR and the Iso output from MFAssign, so that monoisotopic assignments are not made on isotope peaks and sulfur-containing seeds are isotope-supported.

## When to use

Use when assigning molecular formulas to ultrahigh-resolution mass spectra (FT-ICR/Orbitrap) where 13C and 34S polyisotopic peaks must be separated from monoisotopic masses before or after formula assignment.

## When NOT to use

- When the input spectrum has not been recalibrated and isotope-pair mass differences exceed the +/- 5 ppm window — run Recal() first
- When you need definitive proof of the presence or absence of 13C or 34S in a formula — the README states additional work would be needed and IsoFiltR should not be considered definitive
- For MS/MS or sparse spectra where the four-step KMD-series logic breaks down due to low continuity

## Inputs

- Two-column (or three-column) dataframe: first column ion mass, second column intensity (third column may be retention time)
- S/N noise threshold established by KMDNoise or HistNoise
- Two data frames output from IsoFiltR() (Mono and Iso)
- MFAssign/MFAssignCHO formula assignment output

## Outputs

- Mono dataframe of monoisotopic masses (unflagged masses returned to Mono)
- Iso dataframe of candidate 13C and 34S isotope masses
- Formula assignments annotated with matched isotopic masses for each assigned monoisotopic mass

## How to apply

Run IsoFiltR() on the two-column ion mass/intensity dataframe; it uses a four-step identification method (delta-mass pairing within +/- 5 ppm for 13C at 1.003355 or 34S at 1.995797, isotope-specific KMD homologous series, resolution-enhanced KMD separation using integers 21 for 13C and 12 for 34S with re-KMD differences of -0.496/0.503 for 13C and -0.291/0.709 for 34S, and loose abundance-ratio constraints) to split the mass list into Mono and Iso data frames. Feed the Mono output into MFAssignCHO/MFAssign with the KMDNoise-derived S/N threshold, then match assigned monoisotopic masses to their corresponding isotopic masses from the Iso output. Additionally require that sulfur-containing MF used as seeds in the multi-path extension are unambiguous and have a matching 34S peak. Treat the output as a confirmation and filtering device, not definitive proof of 13C or 34S presence in a formula.

## Related tools

- **MFAssignR** (Package providing IsoFiltR isotope filtering and MFAssign isotope-mass matching within the formula assignment pipeline) — https://github.com/skschum/MFAssignR
- **IsoFiltR** (Identifies prospective 13C and 34S isotope ions via the four-step mass-difference, KMD, resolution-enhanced KMD, and abundance-ratio method) — https://github.com/skschum/MFAssignR
- **MFAssign** (Assigns monoisotopic MF and matches assigned monoisotopic masses to corresponding isotopic masses from the Iso output) — https://github.com/skschum/MFAssignR
- **KMDNoise** (Establishes the S/N threshold applied before isotope filtering and assignment) — https://github.com/skschum/MFAssignR

## Examples

```
R: library(MFAssignR); filt <- IsoFiltR(df); assign <- MFAssign(filt$Mono, filt$Iso, SN = noise_threshold, Ambig = "off")
```

## Evaluation signals

- IsoFiltR returns exactly two data frames, Mono and Iso, and masses not flagged as possible mono/iso pairs are returned to the Mono output
- Candidate isotope pairs must satisfy delta mass within +/- 5 ppm of 1.003355 (13C) or 1.995797 (34S)
- Resolution-enhanced KMD differences for accepted pairs fall at -0.496/0.503 for 12/13C and -0.291/0.709 for 32/34S
- Sulfur-containing MF used as extension seeds must be unambiguous and have a matching 34S peak; ambiguous sulfur seeds should be absent from the final list
- Masses flagged as both monoisotopic and isotopic in complex mixtures appear in both Mono and Iso outputs and are resolved only after MF assignment

## Limitations

- IsoFiltR should not be considered definitive proof of the presence or absence of 13C or 34S in a MF; it identifies most MF with these isotopes but additional work is needed to use isotopes to reduce ambiguous assignments
- Abundance-ratio limits are loose because polyisotope abundance varies with analyte signal (isotope-dilution-like behavior) in ultrahigh resolution Orbitrap and FT-ICR measurements
- In complex mixtures, some masses can be flagged as both monoisotopic and isotopic and are included in both outputs, classified only after MF assignment

## Evidence
