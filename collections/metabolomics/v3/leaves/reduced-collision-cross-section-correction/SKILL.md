---
name: reduced-collision-cross-section-correction
description: Use when calibrating single-field DTIMS-MS arrival times to CCS and calibrant
  and sample runs differ in pressure/temperature, requiring charge-state, reduced-mass,
  and P/T correction before linear fitting.
when_to_use_negative:
- Data are stepped-field DTIMS measurements — use AutoCCS's stepped-field method (Stow
  et al., 2017, --mode multi) rather than the enhanced single-field calibration
- Traveling-wave IMS data (Bruker timsTOF, Waters Synapt, SLIM) — these require linear
  timsTOF calibration, polynomial functions, or the linearized power function (Ruotolo
  et al., 2008), not the single-field P/T correction
- Input values are already reduced/P-T-adjusted CCS values rather than raw arrival-time
  feature tables needing calibration
tools:
- name: AutoCCS
  role: 'Open-source tool that implements the enhanced single-field calibration: reduced-CCS
    and P/T correction (Eq. 3) and the degree-1 linear calibration function (Eq. 4)
    for RapidFire-DTIMS data'
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Runtime language (Python 3.7+) in which the Ω'/Ω″ corrections and least-squares
    calibration fit (e.g. numpy.polyfit degree 1) are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:885409f7ec841c863f55105e5b41349dff320da46cb887c96869c3604f4ed452
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# reduced-collision-cross-section-correction

## Summary

Applies AutoCCS's enhanced single-field DTIMS calibration, which converts calibrant arrival times to a reduced CCS (Ω' = (1/z)·sqrt(mI/(mI+mB))·Ω) and a pressure/temperature-adjusted CCS (Ω″ = (P/T)·Ω'), then fits a degree-1 linear calibration function to assign CCS to unknown analyte ions.

## When to use

Use when calibrating single-field DTIMS-MS arrival times to CCS and calibrant and sample runs differ in pressure/temperature, requiring charge-state, reduced-mass, and P/T correction before linear fitting.

## When NOT to use

- Data are stepped-field DTIMS measurements — use AutoCCS's stepped-field method (Stow et al., 2017, --mode multi) rather than the enhanced single-field calibration
- Traveling-wave IMS data (Bruker timsTOF, Waters Synapt, SLIM) — these require linear timsTOF calibration, polynomial functions, or the linearized power function (Ruotolo et al., 2008), not the single-field P/T correction
- Input values are already reduced/P-T-adjusted CCS values rather than raw arrival-time feature tables needing calibration

## Inputs

- Single-field RapidFire-DTIMS calibrant feature tables (Features_csv/*.csv)
- Reference calibrant CCS list (TuneMix-CCS.txt, Agilent tune-mix)
- Frame metadata files (ImsMetadata/*.txt)
- AutoCCS configuration file (autoCCS_config.xml)
- Sample metadata table (Datasets.csv with SampleType, RawFileName, IonPolarity columns)

## Outputs

- Fitted linear calibration coefficients (slope, intercept) relating P/T-adjusted reduced CCS (Ω″) to corrected arrival time
- Coefficient of determination (R²) of the linear calibration fit
- Calibrated CCS values for unknown analyte ions in the sample feature tables

## How to apply

Load the single-field RapidFire-DTIMS calibrant feature tables (Features_csv/*.csv) and the Agilent tune-mix reference CCS list (TuneMix-CCS.txt), plus frame metadata (ImsMetadata/*.txt) and autoCCS_config.xml. For each calibrant ion, compute the reduced CCS Ω' = (1/z)·sqrt(mI/(mI+mB))·Ω using the ion charge state z, ion mass mI, buffer gas mass mB, and known CCS value (Eq. 3). Apply the pressure/temperature correction Ω″ = (P/T)·Ω' to account for condition differences between calibrant and sample runs. Fit Ω″ versus the corrected arrival time with a degree-1 least-squares regression (numpy.polyfit), matching AutoCCS's --degree 1 setting for single-field DTIMS. Record the fitted slope, intercept, and coefficient of determination (R²), then use the calibration function to assign CCS to unknown analyte ions via AutoCCS in --single_mode batch --mode single.

## Related tools

- **AutoCCS** (Open-source tool that implements the enhanced single-field calibration: reduced-CCS and P/T correction (Eq. 3) and the degree-1 linear calibration function (Eq. 4) for RapidFire-DTIMS data) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Runtime language (Python 3.7+) in which the Ω'/Ω″ corrections and least-squares calibration fit (e.g. numpy.polyfit degree 1) are executed)

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --tunemix_sample_type AgilentTuneMix --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- The degree-1 least-squares fit converges and yields finite slope and intercept values for the Ω″ versus corrected-arrival-time relation
- R² is computed and reported alongside slope and intercept in the output (the paper reports no numeric R² threshold, so a qualitative high-fit check applies)
- Each calibrant ion's reduced CCS uses the correct charge state z and reduced-mass factor sqrt(mI/(mI+mB)) per Eq. 3
- The (P/T) scaling is applied between calibrant and sample run conditions, not omitted, when the runs differ in pressure or temperature
- Output calibration parameters are written to calibration_parameters.csv in the --output_dir Results directory

## Limitations

- The paper reports no numeric R² threshold, so calibration acceptance is only a qualitative high-fit judgment
- Discussion section content is absent from the provided document, so no documented limitations or reproducibility gaps can be cited
- Demonstration is limited to Agilent tune-mix calibrant on three platforms (stepped-field DTIMS-MS, single-field RapidFire-DTIMS-MS, SLIM-based IMS); behavior for other calibrant mixes is not documented in the provided context

## Evidence

- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS), single-field methods for DTIMS (standard and enhanced calibration): [evidence span withheld — anchor status: not_found]
- [abstract] Demo datasets are publicly available at MassIVE (Dataset ID: MSV000085979).: "Demo datasets are publicly available at MassIVE (Dataset ID: MSV000085979)."
- [readme] Calibration functions for single field and traveling wave-based methods - Linear function: "Calibration functions for single field and traveling wave-based methods
  - Linear function"
- [readme] python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt": "python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt""
