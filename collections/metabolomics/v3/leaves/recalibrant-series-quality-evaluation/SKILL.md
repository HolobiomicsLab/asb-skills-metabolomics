---
name: recalibrant-series-quality-evaluation
description: Use when a CHO-assigned molecular formula dataframe (from MFAssignCHO
  or MFAssign) is available and the mass spectrum shows systematic mass error that
  must be corrected via RecalList/Recal internal recalibration.
when_to_use_negative:
- The peak list has no prior molecular formula assignment — RecalList must be run
  on the output of MFAssign or MFAssignCHO, not on raw unassigned masses
- The spectrum is already well calibrated with negligible systematic mass error —
  preliminary MFAssignCHO assignment is used specifically to assess mass accuracy
  before deciding recalibration is needed
- The goal is isotope identification rather than recalibration — IsoFiltR, not RecalList/Recal,
  handles 13C and 34S polyisotopic mass filtering
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MFAssignR::RecalList
  role: Generates the data frame of potential recalibrant CH2 homologous series from
    MFAssign/MFAssignCHO output, with the quality metrics used for series selection
  repo: github.com/skschum/MFAssignR
- name: MFAssignR::Recal
  role: Recalibrates the spectrum using up to ten user-chosen series, extending formulas
    via H2/O homologous steps and applying per-segment polynomial moving-average correction
  repo: github.com/skschum/MFAssignR
- name: MFAssignR::MFAssignCHO
  role: Provides the preliminary CHO-only MF assignments whose output feeds RecalList
    for recalibrant series generation
  repo: github.com/skschum/MFAssignR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.envres.2020.110114
    title: 'MFAssignR: Molecular formula assignment software for ultrahigh resolution
      mass spectrometry analysis of environmental complex mixtures'
  source_content_hash: sha256:0169036f37cd41e084a482ba1b9620c0f60dff39dc1eee7c248776696cb25429
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MFAssignR
  - MFAssignR::MFAssignCHO
  - MFAssignR::Recal
  - MFAssignR::RecalList
  - R
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# recalibrant-series-quality-evaluation

## Summary

A method for scoring and selecting internal mass-recalibrant CH2 homologous series from MFAssignR::RecalList output, using the 'Number Observed', 'Mass Range', 'Tall Peak', 'Series Score', 'Peak Distance', and 'Abundance Score' metrics to choose up to ten series that feed MFAssignR::Recal for walking recalibration of ultrahigh-resolution MS peak lists.

## When to use

Use when a CHO-assigned molecular formula dataframe (from MFAssignCHO or MFAssign) is available and the mass spectrum shows systematic mass error that must be corrected via RecalList/Recal internal recalibration.

## When NOT to use

- The peak list has no prior molecular formula assignment — RecalList must be run on the output of MFAssign or MFAssignCHO, not on raw unassigned masses
- The spectrum is already well calibrated with negligible systematic mass error — preliminary MFAssignCHO assignment is used specifically to assess mass accuracy before deciding recalibration is needed
- The goal is isotope identification rather than recalibration — IsoFiltR, not RecalList/Recal, handles 13C and 34S polyisotopic mass filtering

## Inputs

- CHO-assigned molecular formula dataframe produced by MFAssignCHO() or MFAssign()
- S/N-thresholded two-column (mass, intensity) peak list
- RecalList output data frame of potential recalibrant CH2 homologous series with metrics 'Number Observed', 'Mass Range', 'Tall Peak', 'Series Score', 'Peak Distance', 'Abundance Score'

## Outputs

- Recalibrant series table (recal_list.csv) with quality metrics per CH2 homologous series
- Selected set of up to ten recalibrant series used by Recal()
- Recalibrated peak list (recalibrated_mass_list.csv) with reduced systematic mass error

## How to apply

Run MFAssignR::RecalList() on the CHO-assigned formula dataframe (output of MFAssignCHO or MFAssign) to generate a data frame of potential recalibrant CH2 homologous series. Judge each series on its 'Number Observed', 'Mass Range', 'Tall Peak', 'Series Score', 'Peak Distance', and 'Abundance Score' output metrics, which indicate coverage and quality. Select up to ten long series that jointly span the full mass spectral range, ensuring a 'Tall Peak' appears at least every 100 m/z for complete coverage. Feed the selected series to MFAssignR::Recal() with the default mzRange of 30; the function extends the series via plus/minus H2 and O homologous steps to build a recalibrant ion pool, adds any recalibrant ion with a matching 13C peak, and applies a polynomial central moving average mass error correction per spectral segment (Kozhinov et al. 2013 / Savory et al. 2011 walking recalibration). If Recal prints a console warning about mzRange, increase the segment length and re-run. The rationale is that recalibrant ions derived from unambiguous assigned formulas anchor the correction across the whole spectrum, avoiding a priori reliance on external calibrants.

## Related tools

- **MFAssignR::RecalList** (Generates the data frame of potential recalibrant CH2 homologous series from MFAssign/MFAssignCHO output, with the quality metrics used for series selection) — github.com/skschum/MFAssignR
- **MFAssignR::Recal** (Recalibrates the spectrum using up to ten user-chosen series, extending formulas via H2/O homologous steps and applying per-segment polynomial moving-average correction) — github.com/skschum/MFAssignR
- **MFAssignR::MFAssignCHO** (Provides the preliminary CHO-only MF assignments whose output feeds RecalList for recalibrant series generation) — github.com/skschum/MFAssignR

## Examples

```
recal_list <- MFAssignR::RecalList(MFAssignCHO_output)  # inspect 'Number Observed', 'Mass Range', 'Tall Peak' before choosing series for Recal()
```

## Evaluation signals

- recal_list.csv exists and its rows carry the documented series metrics ('Number Observed', 'Mass Range', 'Tall Peak', 'Series Score', 'Peak Distance', 'Abundance Score')
- The selected series jointly span the full mass spectral range with a 'Tall Peak' at least every 100 m/z
- recalibrated_mass_list.csv exists and recalibrated masses show reduced systematic mass error across spectral segments compared to pre-recalibration
- Recal completes without printing a console warning about mzRange (default mzRange is 30; a warning means segment length must be increased and recalibration re-run)
- At most ten recalibrant series were chosen, consistent with the Recal() function's limit

## Limitations

- Only up to ten recalibrant series can be used by Recal(), so selection is a trade-off among series length, score, and spectral coverage
- The default mzRange of 30 may be insufficient for some data; Recal emits a console warning and the segment length must be increased and recalibration re-run
- RecalList quality depends on the preceding CHO assignment quality; MFAssignCHO is a simplified CHO-only assignment intended for preliminary assignments before recalibrant selection
- User caution with function parameter settings and output evaluation is required; the authors note this explicitly and provide several function outputs to assist evaluation

## Evidence
