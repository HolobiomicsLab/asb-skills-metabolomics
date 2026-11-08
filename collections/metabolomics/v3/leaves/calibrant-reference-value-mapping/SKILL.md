---
name: calibrant-reference-value-mapping
description: Use when performing external CCS calibration for single-field DTIMS (e.g.,
  RapidFire) and you need to map calibrant feature-table ions to known reference CCS
  values from a calibrant file such as TuneMix-CCS.txt.
when_to_use_negative:
- Stepped-field DTIMS experiments, where CCS can be determined directly from multiple
  fields without external calibrant reference mapping (use --threshold_n_fields 5
  stepped-field workflow instead)
- Data lacking a calibrant run or a reference CCS list (e.g., no Agilent TuneMix or
  equivalent calibrant file) — external calibration cannot be anchored
- Traveling-wave platforms (Bruker timsTOF, Waters Synapt, SLIM) needing non-linear
  or linearized power calibration functions rather than single-field linear calibration
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Reads the calibrant reference CCS file (--calibrant_file TuneMix-CCS.txt)
    with --tunemix_sample_type AgilentTuneMix to map calibrant ions to known CCS values
    and fit the single-field calibration curve
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Runtime for autoCCS.py (Python 3.7+); pandas/numpy-style loading of calibrant
    feature tables and least-squares fitting of reference-mapped Ω'' vs corrected
    arrival time
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:d6c509470314b6a56d8c84f39ff575bf97e18f7b1601feb9691cd3331f733e93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# calibrant-reference-value-mapping

## Summary

Associates measured calibrant ion features (charge state, ion mass, drift/arrival time) with their known reference CCS values from a calibrant list (e.g., Agilent TuneMix CCS values) so that AutoCCS can build an external single-field DTIMS calibration curve.

## When to use

Use when performing external CCS calibration for single-field DTIMS (e.g., RapidFire) and you need to map calibrant feature-table ions to known reference CCS values from a calibrant file such as TuneMix-CCS.txt.

## When NOT to use

- Stepped-field DTIMS experiments, where CCS can be determined directly from multiple fields without external calibrant reference mapping (use --threshold_n_fields 5 stepped-field workflow instead)
- Data lacking a calibrant run or a reference CCS list (e.g., no Agilent TuneMix or equivalent calibrant file) — external calibration cannot be anchored
- Traveling-wave platforms (Bruker timsTOF, Waters Synapt, SLIM) needing non-linear or linearized power calibration functions rather than single-field linear calibration

## Inputs

- Calibrant feature tables (Features_csv/*.csv) from single-field RapidFire-DTIMS-MS runs
- Reference calibrant CCS list (TuneMix-CCS.txt, Agilent TuneMix known CCS values)
- IMS frame metadata files (ImsMetadata/*.txt) providing pressure/temperature and drift conditions
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns

## Outputs

- Calibrant ion-to-reference-CCS mappings used to fit the linear calibration function
- Fitted calibration parameters (slope, intercept, degree-1 polynomial) applied to unknown sample ions
- CCS values for unknown analyte ions from the calibrant-anchored calibration curve

## How to apply

Load the calibrant feature tables exported from the single-field RapidFire-DTIMS Features_csv directory together with the reference CCS list (TuneMix-CCS.txt) downloaded from MassIVE MSV000085979, using the --calibrant_file argument of autoCCS.py. Specify --tunemix_sample_type AgilentTuneMix so AutoCCS interprets the calibrant list format correctly, and supply the sample metadata (--sample_meta Datasets.csv with --colname_for_sample_type SampleType, --colname_for_filename RawFileName, --colname_for_ionization IonPolarity) to distinguish calibrant runs from sample runs. For each matched calibrant ion, compute the reduced CCS Ω' = (1/z)·sqrt(mI/(mI+mB))·Ω using the ion charge state, ion mass, and buffer gas mass, then apply the pressure/temperature adjustment Ω″ = (P/T)·Ω' to account for conditions differing between calibrant and sample runs. These reference-mapped Ω″ values are then regressed against corrected arrival times (linear fit, --degree 1) to yield the calibration slope and intercept. The mapping quality directly determines calibration accuracy, so each calibrant ion must be correctly matched to its known CCS entry before fitting.

## Related tools

- **AutoCCS** (Reads the calibrant reference CCS file (--calibrant_file TuneMix-CCS.txt) with --tunemix_sample_type AgilentTuneMix to map calibrant ions to known CCS values and fit the single-field calibration curve) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Runtime for autoCCS.py (Python 3.7+); pandas/numpy-style loading of calibrant feature tables and least-squares fitting of reference-mapped Ω'' vs corrected arrival time) — https://www.python.org/downloads/

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt" --feature_files "data/SingleField-RapidFire-DTIMS/Features_csv/*.csv" --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --output_dir data/SingleField-RapidFire-DTIMS/Results/ --tunemix_sample_type AgilentTuneMix --degree 1 --mode single
```

## Evaluation signals

- Every calibrant ion in the feature tables is matched to exactly one entry in TuneMix-CCS.txt (charge state, m/z, and ion polarity consistent with the AgilentTuneMix reference list)
- The reduced-CCS computation uses the correct ion charge state z and buffer gas mass, and the P/T-adjusted Ω'' is derived from frame metadata (pressure and temperature) of the calibrant run
- The degree-1 least-squares fit of Ω'' versus corrected arrival time converges and reports a high R^2 alongside slope and intercept
- Calibrated CCS values for known calibrant ions reproduce the reference TuneMix CCS values within expected agreement when back-predicted through the fitted curve
- Output log (single.log) records the calibrant file path and tunemix_sample_type used, matching the MassIVE MSV000085979 demo dataset configuration

## Limitations

- The paper reports no numeric R^2 threshold for accepting the calibration curve, so fit quality must be judged qualitatively
- Reference CCS values are platform/gas-specific; the Agilent TuneMix list must match the collision gas and ionization mode of the calibrant run
- The P/T correction assumes calibrant and sample runs may differ in pressure and temperature — if metadata are missing or wrong, the Ω'' = (P/T)·Ω' adjustment is invalid

## Evidence

- [methods] The enhanced single-field method computes a reduced CCS (Ω' = (1/z)·sqrt(mI/(mI+mB))·Ω) and a temperature/pressure-adjusted CCS (Ω″ = (P/T)·Ω'): [evidence span withheld — anchor status: not_found]
- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS), single-field methods for DTIMS (standard and enhanced calibration): [evidence span withheld — anchor status: not_found]
- [readme] --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt: "--calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --output_dir data/SingleField-RapidFire-DTIMS/Results/ --tunemix_sample_type AgilentTuneMix"
- [readme] Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979](https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000085979))"
