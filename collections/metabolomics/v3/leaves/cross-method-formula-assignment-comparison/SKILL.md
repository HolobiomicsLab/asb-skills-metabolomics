---
name: cross-method-formula-assignment-comparison
description: Use when a two-column (m/z, intensity) ultrahigh-resolution FT-ICR or
  Orbitrap spectrum of an environmental complex mixture (e.
when_to_use_negative:
- The data set is MS/MS or otherwise discontinuous with no continuous CH2 homologous
  series below the de novo cutoff — the standard MFAssign de novo path relies on continuity
  and the article notes the 'MSMS' parameter changes behavior for sparse data, so
  a standard cross-method comparison on continuous-spectra assumptions is not appropriate.
- Only a single method's formula assignment output is needed without a comparison
  to other molecular formula assignment or noise estimation methods — in that case
  use MFAssignR's pipeline directly rather than this cross-method benchmarking skill.
- The spectrum is not from an environmental complex mixture or lacks ultrahigh resolution
  (e.g., low-resolution unit-mass spectra), where the KMD/isotope-pair logic (e.g.,
  13C at +/-5 ppm, re-KMD integer 21) is not applicable.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: MFAssignR
  role: Core R package providing the full pipeline (KMDNoise, IsoFiltR, MFAssignCHO,
    RecalList, Recal, MFAssign) whose outputs are benchmarked against other molecular
    formula assignment and noise estimation methods
  repo: github.com/skschum/MFAssignR
- name: MFAssign
  role: Assigns molecular formulas (12C, 1H, 16O plus heteroatoms, isotopes, Na+ adducts)
    using CH2 Kendrick mass defects, z* sorting, and CH2/O/H2/H2O/CH2O formula extensions
    with path-frequency prioritization; its formula list is the primary output compared
    across methods
- name: KMDNoise
  role: Custom Kendrick-mass-defect-based noise estimator that determines the noise
    level from a mass-dependent CH2 KMD slice containing only noise; its noise estimate
    is one side of the noise-method comparison
- name: HistNoise
  role: Alternative noise estimation method (Zhurov et al. 2014 histogram of natural
    log intensities) available in MFAssignR for cross-checking the noise estimate
- name: IsoFiltR
  role: Four-step 13C/34S isotope-pair identifier that returns Mono and Iso data frames,
    removing isotope masses from the monoisotopic list before formula assignment in
    the comparison run
- name: MFAssignCHO
  role: Simplified CHO-only assignment used for preliminary mass-accuracy assessment
    prior to recalibrant selection in the pipeline being compared
- name: RecalList
  role: Generates potential recalibrant CH2 homologous series (with 'Number Observed',
    'Mass Range', 'Tall Peak' metrics) for choosing up to ten recalibrant series
- name: Recal
  role: Recalibrates the spectrum using a polynomial central moving average walking
    recalibration from the chosen recalibrant series before final formula assignment
- name: SNplot
  role: Plots the spectrum with masses colored red (below S/N threshold) to blue (above)
    to visually verify the noise-threshold effectiveness during the comparison run
- name: R
  role: Runtime environment in which the MFAssignR pipeline is executed for the cross-method
    comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.envres.2020.110114
    title: 'MFAssignR: Molecular formula assignment software for ultrahigh resolution
      mass spectrometry analysis of environmental complex mixtures'
  source_content_hash: sha256:65c508ad2a0e83c1860ceb05d1cb02bc70841028d6b30db1917b7ddc118e411f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HistNoise
  - IsoFiltR
  - KMDNoise
  - MFAssign
  - MFAssignCHO
  - MFAssignR
  - R
  - Recal
  - RecalList
  - SNplot
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-method-formula-assignment-comparison

## Summary

A benchmarking practice for evaluating MFAssignR's full pipeline (KMDNoise noise estimation, IsoFiltR isotope filtering, Recal recalibration, MFAssign formula assignment) against other available molecular formula assignment and noise estimation methods on the same ultrahigh-resolution mass spectrum of an environmental complex mixture.

## When to use

Use when a two-column (m/z, intensity) ultrahigh-resolution FT-ICR or Orbitrap spectrum of an environmental complex mixture (e.g., biomass burning organic aerosol) needs to be processed with MFAssignR AND the resulting molecular formula assignments and noise estimate must be compared head-to-head with alternative molecular formula assignment and noise estimation methods.

## When NOT to use

- The data set is MS/MS or otherwise discontinuous with no continuous CH2 homologous series below the de novo cutoff — the standard MFAssign de novo path relies on continuity and the article notes the 'MSMS' parameter changes behavior for sparse data, so a standard cross-method comparison on continuous-spectra assumptions is not appropriate.
- Only a single method's formula assignment output is needed without a comparison to other molecular formula assignment or noise estimation methods — in that case use MFAssignR's pipeline directly rather than this cross-method benchmarking skill.
- The spectrum is not from an environmental complex mixture or lacks ultrahigh resolution (e.g., low-resolution unit-mass spectra), where the KMD/isotope-pair logic (e.g., 13C at +/-5 ppm, re-KMD integer 21) is not applicable.

## Inputs

- Two-column (mass, intensity) or three-column (mass, intensity, retention time) ultrahigh-resolution mass spectrum data frame from an environmental complex mixture (e.g., biomass burning influenced organic aerosol, LC-MS capable)
- Optional: results from other molecular formula assignment and noise estimation methods for the same spectrum, to build the comparison table

## Outputs

- Assigned molecular formula list (Mono and Iso data frames from IsoFiltR; ambiguous/unambiguous MF table from MFAssign) for the MFAssignR run
- Noise estimate (from KMDNoise or HistNoise) with S/N threshold used
- comparison_table.csv summarizing MFAssignR formula assignments and noise estimate versus other available molecular formula assignment and noise estimation methods

## How to apply

Run the full MFAssignR recommended order of operations on the target spectrum: KMDNoise() for noise estimation (a mass-dependent CH2-based Kendrick mass defect slice containing only noise), SNplot() to verify the S/N threshold, IsoFiltR() to flag 13C/34S isotope pairs (delta masses 1.003355 and 1.995797 at +/-5 ppm, with resolution-enhanced KMD integers 21 and 12), MFAssignCHO() for preliminary mass-accuracy assessment, RecalList()/Recal() for polynomial central moving average walking recalibration (mzRange = 30), and MFAssign() for final formula assignment using CH2 Kendrick base de novo assignment below a user-defined cutoff (e.g., m/z 300) with CH2/O/H2/H2O/CH2O formula extensions and path-frequency prioritization. Collect the assigned formula list and noise estimate into a comparison table alongside the outputs of other available molecular formula assignment and noise estimation methods, then judge the comparison on assignment coverage, ambiguity rates, and the quality of chemically reasonable CHO and heteroatom formulas.

## Related tools

- **MFAssignR** (Core R package providing the full pipeline (KMDNoise, IsoFiltR, MFAssignCHO, RecalList, Recal, MFAssign) whose outputs are benchmarked against other molecular formula assignment and noise estimation methods) — github.com/skschum/MFAssignR
- **MFAssign** (Assigns molecular formulas (12C, 1H, 16O plus heteroatoms, isotopes, Na+ adducts) using CH2 Kendrick mass defects, z* sorting, and CH2/O/H2/H2O/CH2O formula extensions with path-frequency prioritization; its formula list is the primary output compared across methods)
- **KMDNoise** (Custom Kendrick-mass-defect-based noise estimator that determines the noise level from a mass-dependent CH2 KMD slice containing only noise; its noise estimate is one side of the noise-method comparison)
- **HistNoise** (Alternative noise estimation method (Zhurov et al. 2014 histogram of natural log intensities) available in MFAssignR for cross-checking the noise estimate)
- **IsoFiltR** (Four-step 13C/34S isotope-pair identifier that returns Mono and Iso data frames, removing isotope masses from the monoisotopic list before formula assignment in the comparison run)
- **MFAssignCHO** (Simplified CHO-only assignment used for preliminary mass-accuracy assessment prior to recalibrant selection in the pipeline being compared)
- **RecalList** (Generates potential recalibrant CH2 homologous series (with 'Number Observed', 'Mass Range', 'Tall Peak' metrics) for choosing up to ten recalibrant series)
- **Recal** (Recalibrates the spectrum using a polynomial central moving average walking recalibration from the chosen recalibrant series before final formula assignment)
- **SNplot** (Plots the spectrum with masses colored red (below S/N threshold) to blue (above) to visually verify the noise-threshold effectiveness during the comparison run)
- **R** (Runtime environment in which the MFAssignR pipeline is executed for the cross-method comparison)

## Examples

```
R> library(MFAssignR); noise <- KMDNoise(spectrum); SNplot(spectrum, noise); iso <- IsoFiltR(spectrum); mf <- MFAssign(iso$Mono, DeNovo = 300)
```

## Evaluation signals

- The full recommended order of operations (KMDNoise -> SNplot -> IsoFiltR -> MFAssignCHO -> RecalList -> Recal -> MFAssign) is executed in sequence, not a single-step run
- Recalibrant series chosen from RecalList must show 'Tall Peak' coverage at least every 100 m/z across the spectrum
- Formula assignment coverage and ambiguity rate (single mass with multiple MF) can be compared between MFAssignR's output and the other methods' outputs on identical input masses
- Mass accuracy after Recal should improve: MFAssignCHO is run explicitly 'to assess the mass accuracy' before recalibration, so a recalibrated vs. pre-recalibration mass error comparison is a check
- Isotope filtering efficacy: peaks flagged as 13C/34S (delta mass 1.003355 or 1.995797, +/-5 ppm) should not be assigned monoisotopic formulas in the final Mono list

## Limitations

- IsoFiltR 'should not be considered as definitive proof of the presence or absence of 13C or 34S in a MF' — some isotope flags may be false positives or false negatives
- Ambiguous formula assignments can persist when 'Ambig' is turned on; the default path-frequency prioritization reduces but does not eliminate multi-MF masses
- Sulfur-containing MF used as seeds must be unambiguous and have a matching 34S peak, so some sulfur assignments may be missed
- MFAssignR was designed for ultrahigh-resolution spectra of environmental complex mixtures; prior versions (<= v.0.0.3) used different functions (MFAssignAll, MFAssignMSMS), so version consistency matters when comparing across methods
- RecalList allows up to ten recalibrant series; choosing fewer or poorly distributed series can leave parts of the mass range without recalibration

## Evidence

- [abstract] MFAssignR was evaluated on biomass burning influenced organic aerosol and compared to other molecular formula assignment and noise estimation methods: "it was used to characterize a sample of biomass burning influenced organic aerosol and the results were compared to those from other available methods of molecular formula assignment and noise"
- [methods] MFAssignR pipeline order of operations: "1. Run KMDNoise() to determine the noise level for the data."
- [methods] MFAssign pipeline steps 2 and 3: "2. Check effectiveness of S/N threshold using SNplot()."
- [methods] IsoFiltR step in pipeline: "3. Use IsoFiltR() to identify potential 13C and 34S isotope masses."
- [methods] MFAssignCHO preliminary step: "run MFAssignCHO() to assign MF with C, H, and O to assess the mass accuracy."
- [methods] RecalList role: "Use RecalList() to generate a list of the potential recalibrant series."
- [methods] Recal role: "After choosing recalibrant series, use Recal() to recalibrate the mass lists."
- [methods] Four-step isotope identification: "IsoFiltR() identifies potential isotope masses using a four-step identification method."
- [abstract] KMDNoise novelty: "Novel applications of mass defect analysis were used in the functions for noise estimation and isotope pair identification."
- [abstract] Comparison finding: [evidence span withheld — anchor status: not_found]
- [abstract] Formula extension mechanism: "Using formula extensions, exact mass measurements are converted to unambiguous molecular formulas via data dependent pathways, reducing a priori decisions."
- [readme] IsoFiltR caution: "Thus IsoFiltR() should not be considered as definitive proof of the presence or absence of 13C or 34S in a MF"
- [methods] S/N threshold visualization: "The SNplot function is used to show the mass spectrum with the masses below and above the threshold shown in the output plot with red to blue colors"
- [readme] De novo concept and CH2 KMD sorting: "First, Kendrick mass defects (KMD) and z* values are calculated with a CH2 Kendrick base to sort the measured masses into CH2 homologous series (Stenson et al., 2003)."
