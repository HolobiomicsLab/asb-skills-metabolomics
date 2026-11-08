---
name: mass-accuracy-shift-rule-implementation
description: Reference for encoding Rapid QC-MS's per-IS warn thresholds (Δm/z > 80%
  of the library m/z-shift cutoff; ΔRT > 66% of library RT cutoff; in-run ΔRT > 80%
  of in-run average cutoff; missing-IS → Fail) into a decision function.
when_to_use_negative:
- Data is already aggregated to a whole-run or dashboard-level score, not a single
  IS target's status
- You need to compute the raw ΔRT / Δm/z metrics from mzML — use MSConvert + MS-DIAL
  peak picking first, then apply these rules
- You are using a QC scheme whose warn fractions are not 0.66 (library RT) / 0.80
  (in-run RT) / 0.80 (m/z) — re-derive the fractions from your own SOP rather than
  hard-coding these
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Rapid QC-MS
  role: Source of the warn/fail rule fractions and the alert statuses ('Pass'/'Warn'/'Fail');
    the rule module mirrors its specimen-level IS QC logic
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL
  role: Upstream peak picking and identification that supplies the RT / m/z measurements
    and library thresholds consumed by the warn rules
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
- name: MSConvert
  role: Converts vendor raw files to mzML before MS-DIAL processing, enabling the
    vendor-agnostic RT/m/z inputs used by the rules
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: Python
  role: Language for the pure decision function and pytest boundary tests (versions
    3.8–3.11)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:bb66cc7050ac9ac339971391b4f622efc62acdeed9794caf6e995b6f6d5cc65f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  repo_url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - MS-DIAL
  - MSConvert
  - Python
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# mass-accuracy-shift-rule-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements Rapid QC-MS's per-internal-standard 'Warning' rule for mass accuracy drift: flag Δm/z when it exceeds 80% of the specimen's library m/z-shift threshold, alongside the parallel RT-shift warn rules (66% library RT / 80% in-run average RT) and the missing-IS 'Fail' rule.

## When to use

Reference for encoding Rapid QC-MS's per-IS warn thresholds (Δm/z > 80% of the library m/z-shift cutoff; ΔRT > 66% of library RT cutoff; in-run ΔRT > 80% of in-run average cutoff; missing-IS → Fail) into a decision function.

## When NOT to use

- Data is already aggregated to a whole-run or dashboard-level score, not a single IS target's status
- You need to compute the raw ΔRT / Δm/z metrics from mzML — use MSConvert + MS-DIAL peak picking first, then apply these rules
- You are using a QC scheme whose warn fractions are not 0.66 (library RT) / 0.80 (in-run RT) / 0.80 (m/z) — re-derive the fractions from your own SOP rather than hard-coding these

## Inputs

- Measured retention time of an IS peak in the current specimen
- In-run average retention time across specimens
- Measured m/z of the IS peak
- Library / sequence RT and m/z shift thresholds from the .msp library entry
- Detected peak presence flag for each IS target

## Outputs

- Per-IS status string: 'Pass', 'Warn', or 'Fail'
- Suggested pytest cases covering at/below/above each 0.66 and 0.80 fraction and the missing-IS case

## How to apply

Encode the measured RT, in-run RT, m/z, and library thresholds as pure-function arguments. Apply the paper's three warn fractions: individual-IS ΔRT > 66% of the RT shift-from-library threshold, in-run ΔRT > 80% of the in-run average threshold, and Δm/z > 80% of the m/z shift-from-library threshold. A 'Fail' fires only when an IS target has no detected peak in the specimen. Boundary-test each fraction with pytest at, just-below, and just-above 0.66 and 0.80 to confirm no tolerance drift, and always return 'Fail' for missing IS.

## Related tools

- **Rapid QC-MS** (Source of the warn/fail rule fractions and the alert statuses ('Pass'/'Warn'/'Fail'); the rule module mirrors its specimen-level IS QC logic) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL** (Upstream peak picking and identification that supplies the RT / m/z measurements and library thresholds consumed by the warn rules) — http://prime.psc.riken.jp/compms/msdial/main.html
- **MSConvert** (Converts vendor raw files to mzML before MS-DIAL processing, enabling the vendor-agnostic RT/m/z inputs used by the rules) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **Python** (Language for the pure decision function and pytest boundary tests (versions 3.8–3.11))

## Examples

```
pytest test_is_rules.py --tb=short; assert classify_is(delta_rt=0.66*rt_thr, delta_rt_inrun=0.80*avg_thr, delta_mz=0.80*mz_thr, detected=True) == 'Warn'
```

## Evaluation signals

- Boundary test at ΔRT/RT_threshold = 0.66 exactly, just-below, and just-above yields no tolerance drift
- Boundary test at Δm/z/mz_threshold = 0.80 exactly, just-below, and just-above yields no tolerance drift
- Missing-IS case always returns 'Fail' regardless of ΔRT or Δm/z values
- In-run ΔRT/in-run_average_RT_threshold = 0.80 boundary behaves identically to the library-RT boundary
- Rule module is pure (no I/O side effects) so it can be unit-tested with synthetic thresholds

## Limitations

- Warn fractions (0.66 / 0.80 / 0.80) are hard-coded from the paper's stated thresholds — re-tune them if your library thresholds differ from the Rapid QC-MS defaults
- Rapid QC-MS is only extensively tested on Thermo RAW files and Thermo acquisition sequences; other vendor formats may carry processing bugs upstream
- Rule depends on MS-DIAL correctly detecting each IS peak; missed detections collapse the decision to 'Fail' regardless of true mass accuracy

## Evidence

- [methods] In Rapid QC-MS, missing IS are marked 'Fail'; an individual IS receives 'Warning' if ΔRT is greater than 66% of the RT shift-from-library threshold, in-run ΔRT is greater than 80% of the in-run average threshold, or Δm/z is greater than 80% of the m/z shift-from-library threshold.: [evidence span withheld — anchor status: not_found]
- [readme] Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert for vendor format data conversion and MS-DIAL for data processing and identification: [evidence span withheld — anchor status: not_found]
