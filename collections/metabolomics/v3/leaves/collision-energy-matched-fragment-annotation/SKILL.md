---
name: collision-energy-matched-fragment-annotation
description: Use when both CFM-ID-predicted fragment lists and experimental HRMS/MS
  spectra (multi collision energy, e.g. 20/40/60 eV) exist for DNA adduct standards,
  and annotation confidence from fragment overlap is needed.
when_to_use_negative:
- The compound class is not DNA adducts and no [M+H-dR]+ deoxyribose-loss diagnostic
  is expected, so the deoxyribose-loss anchor fragment check is meaningless.
- Only experimental MS/MS (or only CFM-ID predictions) are available for an adduct,
  so no predicted-vs-experimental overlap can be computed.
- Spectra were acquired at a single collision energy, preventing energy-resolved (20/40/60
  eV) matching.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID
  role: generates in silico MS/MS fragments for all database adducts (fragments >5%
    relative intensity at three energy levels) that are matched against experimental
    spectra
- name: tandem HRMS/MS
  role: provides the experimental MS/MS spectra of the 15 reference standards against
    which predicted fragments are matched
- name: rstudio/DT
  role: renders the experimental and predicted fragment tables as interactive web
    datatables for inspection
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:2d23f21ede6a6b2e218e03dd67dc4ab929696a89409eeb21714077e6cb72d5e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - rstudio/DT
  - tandem HRMS/MS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# collision-energy-matched-fragment-annotation

## Summary

Validates DNA adduct annotations by matching CFM-ID in silico MS/MS fragments against experimental MS/MS spectra of reference standards acquired at 20, 40, and 60 eV on a Vion-IMS-qTOF, computing per-adduct fragment overlap to judge whether predicted fragments can support annotation.

## When to use

Use when both CFM-ID-predicted fragment lists and experimental HRMS/MS spectra (multi collision energy, e.g. 20/40/60 eV) exist for DNA adduct standards, and annotation confidence from fragment overlap is needed.

## When NOT to use

- The compound class is not DNA adducts and no [M+H-dR]+ deoxyribose-loss diagnostic is expected, so the deoxyribose-loss anchor fragment check is meaningless.
- Only experimental MS/MS (or only CFM-ID predictions) are available for an adduct, so no predicted-vs-experimental overlap can be computed.
- Spectra were acquired at a single collision energy, preventing energy-resolved (20/40/60 eV) matching.

## Inputs

- Experimental fragment datatable for 15 DNA adduct reference standards (MS/MS at 20, 40, 60 eV on Vion-IMS-qTOF)
- CFM-ID in silico MS/MS fragment datatable (fragments >5% relative intensity, three energy levels)
- Adduct metadata (structures/formulas) from the 582-entry DNA adduct database

## Outputs

- fragment_overlap_summary.csv (per-adduct matched/total predicted and experimental fragment counts per collision energy)
- 15 overlaid predicted-vs-experimental spectra plots (one panel per standard, mirroring Supplementary Figures S1-S15)

## How to apply

Load the experimental fragment datatable for the 15 reference standards (MS/MS acquired at 20, 40, and 60 eV on the Vion-IMS-qTOF) alongside the CFM-ID predicted fragment datatable, which retains only fragments above 5% relative intensity generated at three energy levels. For each adduct, match predicted fragment m/z values against experimental m/z values at each collision energy using a high-resolution mass tolerance consistent with qTOF accuracy, and compute per-adduct overlap counts (matched fragments relative to total predicted and total experimental fragments), explicitly noting the characteristic [M+H-dR]+ deoxyribose-loss fragment. Generate overlaid predicted-vs-experimental spectra plots, one panel per standard (15 panels total), mirroring Supplementary Figures S1-S15, and write the overlap statistics to fragment_overlap_summary.csv. The rationale is that the paper found 'considerable overlap' between predicted and experimental spectra, so CFM-ID fragments can support annotation of the 582-entry DNA adduct database even where no standard is available.

## Related tools

- **CFM-ID** (generates in silico MS/MS fragments for all database adducts (fragments >5% relative intensity at three energy levels) that are matched against experimental spectra)
- **tandem HRMS/MS** (provides the experimental MS/MS spectra of the 15 reference standards against which predicted fragments are matched)
- **rstudio/DT** (renders the experimental and predicted fragment tables as interactive web datatables for inspection) — https://github.com/rstudio/DT

## Examples

```
Rscript -e 'DT::datatable(read.csv("fragment_overlap_summary.csv"))'
```

## Evaluation signals

- The overlap summary qualitatively reproduces the reported result: experimental and CFM-ID in silico spectra show considerable overlap across the 15 standards.
- Every one of the 15 standards has a comparison panel, mirroring Supplementary Figures S1-S15.
- The characteristic [M+H-dR]+ deoxyribose-loss fragment is present/annotated in each adduct's matched fragment set.
- Matching was performed per collision energy (20, 40, 60 eV), not pooled, and the mass tolerance is consistent with qTOF accuracy.
- Predicted fragment lists are filtered to fragments above 5% relative intensity before matching.

## Limitations

- Only 15 reference standards were measured, so the spectral library is preliminary and overlap-based validation covers just that subset of the 582-entry database.
- Overlap is qualitative ('considerable overlap'); the workflow does not establish a quantitative similarity threshold for confident annotation.
- CFM-ID predictions depend on correct input structures/SMILES from the harmonized database; errors in harmonization propagate to fragment prediction.
- The rstudio/DT package used for interactive datatables is in maintenance-only mode, with no guarantee that new fixes or pull requests will be merged.

## Evidence

- [abstract] Preliminary spectral library from experimental MS/MS of 15 standards: "In the fourth step, a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [abstract] CFM-ID used for in silico fragments: "generating in silico MS/MS fragments for all the adducts"
- [abstract] Experimental and predicted fragments reported in interactive datatables: "reporting both experimental and predicted fragments into interactive web datatables"
- [readme] DT package provides datatable() used for the interactive fragment tables: [evidence span withheld — anchor status: external_doc]
