---
name: molecular-formula-assignment-mass-spectrometry
description: Use when a two-column (ion mass, intensity) peak list from UHR-MS (Orbitrap/FT-ICR)
  of natural organic matter or aerosol needs chemically reasonable, low-ambiguity
  molecular formula assignments with C, H, O and optional heteroatoms.
when_to_use_negative:
- Spectra at unit or low resolution where KMD sorting, re-KMD isotope pairing (+/-
  5 ppm), and exact-mass moiety assignment are not meaningful
- Highly discontinuous spectra (e.g., raw MS/MS fragment lists) without enabling the
  MSMS parameter, since standard mode pre-filters ions below the de novo threshold
- Datasets already carrying vetted molecular formulas — the skill is for de novo formula
  assignment, not re-annotation
edam_operation: http://edamontology.org/operation_3627
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MFAssignR
  role: R package providing the whole assignment pipeline (MFAssign, MFAssignCHO,
    IsoFiltR, KMDNoise, SNplot, Recal, RecalList)
  repo: https://github.com/skschum/MFAssignR
- name: MFAssignCHO
  role: Assigns C, H, O-only formulas via the CHOFIT low-mass-moiety strategy; faster,
    used for preliminary assignment and recalibrant selection
  repo: https://github.com/skschum/MFAssignR
- name: MFAssign
  role: Full multi-element assignment with KMD/z* homologous-series seeding, multi-path
    CH2/O/H2/H2O/CH2O extension, and path-frequency prioritization
  repo: https://github.com/skschum/MFAssignR
- name: IsoFiltR
  role: Four-step 13C and 34S isotope-pair identification producing the Mono and Iso
    data frames consumed by MFAssign
  repo: https://github.com/skschum/MFAssignR
- name: KMDNoise
  role: Kendrick-mass-defect-based noise estimation to set the S/N threshold applied
    before assignment
  repo: https://github.com/skschum/MFAssignR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.envres.2020.110114
    title: 'MFAssignR: Molecular formula assignment software for ultrahigh resolution
      mass spectrometry analysis of environmental complex mixtures'
  source_content_hash: sha256:9fb71aa3ec88bc0dbe3d7169d07989c0007d53e31bdad2b1e1b99f0fda069e7b
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
  edam_operation: http://edamontology.org/operation_3627
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# molecular-formula-assignment-mass-spectrometry

## Summary

Assign molecular formulas to ultrahigh-resolution mass spectra of complex organic mixtures using MFAssignR's CHOFIT-derived CHO core assignment plus multi-path CH2/O/H2/H2O/CH2O formula extensions, prioritizing formulas by extension-path frequency to minimize ambiguity.

## When to use

Use when a two-column (ion mass, intensity) peak list from UHR-MS (Orbitrap/FT-ICR) of natural organic matter or aerosol needs chemically reasonable, low-ambiguity molecular formula assignments with C, H, O and optional heteroatoms.

## When NOT to use

- Spectra at unit or low resolution where KMD sorting, re-KMD isotope pairing (+/- 5 ppm), and exact-mass moiety assignment are not meaningful
- Highly discontinuous spectra (e.g., raw MS/MS fragment lists) without enabling the MSMS parameter, since standard mode pre-filters ions below the de novo threshold
- Datasets already carrying vetted molecular formulas — the skill is for de novo formula assignment, not re-annotation

## Inputs

- Two-column dataframe of ion mass and intensity (three-column with retention time supported for LC-MS) from IsoFiltR 'Mono' output
- IsoFiltR 'Iso' output dataframe of candidate 13C/34S isotopic masses
- S/N noise threshold determined by KMDNoise (or HistNoise) and verified with SNplot
- User-set parameters: de novo cutoff (e.g., m/z 300), O/C and H/C ratio limits, DBE-O limits, minimum O count

## Outputs

- Unambiguous and ambiguous molecular formula assignment lists (separate output data frames) with 12C/1H/16O plus optional heteroatoms (13C, 14N/15N, 31P, 32S/34S, 35Cl/37Cl, 19F, 79Br/81Br, 126I) and Na+ adducts
- Path-frequency annotations tracking how many extension paths intercept each assigned formula
- Annotated formula list saved as mf_assignment_output.csv
- Isotope-matched assignment table linking monoisotopic masses to corresponding 13C/34S isotopic masses

## How to apply

First filter isotopes with IsoFiltR, which returns Mono and Iso data frames, and establish a noise S/N threshold (e.g., via KMDNoise and SNplot). Run MFAssignCHO on the Mono list to assign C, H, O formulas via the low-mass-moiety CHOFIT strategy. MFAssign then computes CH2-based Kendrick mass defects (KMD) and z* values to sort masses into CH2 homologous series, attempts assignment on 1–3 seed members below the user-defined de novo cutoff (e.g., m/z 300), and extends unambiguous seeds by CH2, O, H2, H2O, and CH2O bases. For each mass, keep the formula with the largest number of intercepting extension paths (Ambig='off'); where conflicts remain, HetCut selects the formula with the fewest heteroatoms. Finally match assigned monoisotopic masses to their IsoFiltR isotopic counterparts and save the annotated formula list (e.g., mf_assignment_output.csv).

## Related tools

- **MFAssignR** (R package providing the whole assignment pipeline (MFAssign, MFAssignCHO, IsoFiltR, KMDNoise, SNplot, Recal, RecalList)) — https://github.com/skschum/MFAssignR
- **MFAssignCHO** (Assigns C, H, O-only formulas via the CHOFIT low-mass-moiety strategy; faster, used for preliminary assignment and recalibrant selection) — https://github.com/skschum/MFAssignR
- **MFAssign** (Full multi-element assignment with KMD/z* homologous-series seeding, multi-path CH2/O/H2/H2O/CH2O extension, and path-frequency prioritization) — https://github.com/skschum/MFAssignR
- **IsoFiltR** (Four-step 13C and 34S isotope-pair identification producing the Mono and Iso data frames consumed by MFAssign) — https://github.com/skschum/MFAssignR
- **KMDNoise** (Kendrick-mass-defect-based noise estimation to set the S/N threshold applied before assignment) — https://github.com/skschum/MFAssignR

## Examples

```
R: mf <- MFAssignCHO(Mono = iso$Mono, Iso = iso$Iso, SN = 6); formulas <- MFAssign(Mono = iso$Mono, Iso = iso$Iso, SN = 6, Ambig = "off", HetCut = "on")
```

## Evaluation signals

- The number of masses flagged ambiguous is small relative to the unambiguous set — the multi-path path-prioritization step should greatly reduce ambiguous assignments versus the unfiltered chemically reasonable candidate list
- Each retained formula for a mass has the largest number of intercepting extension paths among candidates, tracing back to unambiguous seeds below the de novo cutoff (e.g., m/z 300)
- All assigned formulas pass the non-adjustable QA rules: nitrogen rule, large atom rule, maximum H rule, maximum DBE rule, and Senior rules, plus user O/C, H/C, DBE-O and minimum-O limits
- Sulfur-containing seeds are unambiguous and have a matching 34S isotope peak in the IsoFiltR Iso output
- Recalibration quality: RecalList-selected CH2 homologous series (up to ten) cover the spectrum ('Number Observed', 'Mass Range', 'Tall Peak') and Recal reduces mass error before assignment

## Limitations

- IsoFiltR is not definitive proof of the presence or absence of 13C or 34S; some masses can be flagged as both monoisotopic and isotopic in complex mixtures and are only classified after MF assignment
- Increasing element count and molecular weight inflate chemically reasonable candidate formulas, so MFAssign returns a list of ambiguous and unambiguous MF rather than a single answer per mass
- Ambiguous MF on seed members below the de novo cutoff are returned to the unassigned list, limiting coverage; the MSMS parameter relaxes this at the cost of runtime
- Default settings are relatively lenient; user caution with parameter settings and output evaluation is explicitly required
- Prior versions (<= v.0.0.3) used MFAssignAll/MFAssignMSMS functions now replaced by the Ambig and MSMS parameters

## Evidence
