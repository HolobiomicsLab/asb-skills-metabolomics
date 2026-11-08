---
name: mass-spectrometry-mz-deviation-tracking
description: Use when tracking the long-term QC pool on a Waters Synapt instrument
  and you need to compute per-injection m/z deviations (ppm) from target m/z values
  and flag deviations > 20 ppm as serious analytical problems.
when_to_use_negative:
- Input is already a processed feature table with annotated m/z — module_TrackCmp
  expects raw mzML and a target list of m/z/RT pairs
- Goal is retention-time drift only (article reports RT deviations normally < 0.05
  min within short term); use RT tracking rather than m/z deviation monitoring
- Monitoring intensity drift or sensitivity loss (e.g. tryptophan intensity decreasing)
  — use the intensity/peak-shape tracking side of QC4Metabolomics instead of m/z deviation
  flagging
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Real-time and retrospective QC software whose module_TrackCmp performs the
    m/z deviation tracking
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web dashboard used to present the m/z deviation results
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment of QC4Metabolomics and the ProteoWizard mzML converter used to
    obtain the input files
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: module_TrackCmp
  role: Detects peaks for the user-defined m/z and RT pairs and extracts observed
    m/z, RT, intensity, peak shape
- name: module_Warner
  role: Sends email warnings when a quality parameter (e.g. m/z deviation) exceeds
    a user-defined threshold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:4b7ba4777dc3e74cda7e1e35afc3ed8840428eb66d892a3ca7720f3da7773005
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  - module_TrackCmp
  - module_Warner
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# mass-spectrometry-mz-deviation-tracking

## Summary

Tracks observed m/z for user-defined compounds across an untargeted metabolomics run and flags injections whose deviation from the target m/z exceeds 20 ppm — the QC4Metabolomics threshold indicating serious analytical problems (e.g. miscalibration) that should be addressed promptly.

## When to use

Use when tracking the long-term QC pool on a Waters Synapt instrument and you need to compute per-injection m/z deviations (ppm) from target m/z values and flag deviations > 20 ppm as serious analytical problems.

## When NOT to use

- Input is already a processed feature table with annotated m/z — module_TrackCmp expects raw mzML and a target list of m/z/RT pairs
- Goal is retention-time drift only (article reports RT deviations normally < 0.05 min within short term); use RT tracking rather than m/z deviation monitoring
- Monitoring intensity drift or sensitivity loss (e.g. tryptophan intensity decreasing) — use the intensity/peak-shape tracking side of QC4Metabolomics instead of m/z deviation flagging

## Inputs

- mzML files of the long-term QC pool (e.g. MetNexs) from a Waters Synapt instrument
- User-defined list of target m/z and retention time (RT) pairs
- QC4Metabolomics database populated by module_Files with newly acquired files

## Outputs

- Per-injection time-line of observed m/z, retention time, intensity and peak shape diagnostics per tracked compound
- Per-injection m/z deviation (ppm) time-line with entries flagged where deviation exceeds 20 ppm
- Dashboard plot/table (QC4Metabolomics Shiny web dashboard) showing m/z deviation over time

## How to apply

Load the mzML files for the MetNexs long-term QC pool acquired on the Waters Synapt instrument and run module_TrackCmp on a user-defined list of m/z and retention time pairs; the module detects peaks and extracts observed m/z, retention time, intensity, and peak-shape diagnostics for each injection. Compute m/z deviations in ppm between the observed m/z and the target m/z for each tracked compound across the time-line. Flag injections where the deviation exceeds 20 ppm — the threshold the article associates with serious analytical problems that should be addressed promptly — and, if desired, wire module_Warner to email when a quality parameter exceeds a user-defined threshold. Inspect the per-injection deviation time-line for systematic offsets (e.g. the May/June 2022 calibration offset) rather than isolated outliers, since instruments typically do not alert when calibration is inaccurate.

## Related tools

- **QC4Metabolomics** (Real-time and retrospective QC software whose module_TrackCmp performs the m/z deviation tracking) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web dashboard used to present the m/z deviation results) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment of QC4Metabolomics and the ProteoWizard mzML converter used to obtain the input files) — https://github.com/stanstrup/QC4Metabolomics
- **module_TrackCmp** (Detects peaks for the user-defined m/z and RT pairs and extracts observed m/z, RT, intensity, peak shape)
- **module_Warner** (Sends email warnings when a quality parameter (e.g. m/z deviation) exceeds a user-defined threshold)

## Examples

```
Rscript -e 'QC4Metabolomics::module_TrackCmp(qc4m_db, compound_list = metnexs_targets)'  # then flag injections where abs(mz_deviation_ppm) > 20 in the Shiny dashboard
```

## Evaluation signals

- Every tracked compound in the MetNexs QC pool has a per-injection m/z deviation value expressed in ppm
- Injections flagged in the output table have |deviation| > 20 ppm; no unflagged injection exceeds the threshold
- Observed m/z and RT values come from module_TrackCmp peak detection on the user-defined m/z/RT list, not from the raw mzML alone
- Deviation time-line reveals the documented systematic calibration offset pattern (e.g. May/June 2022) when replayed on the article's data
- Dashboard output appears in the QC4Metabolomics Shiny app and reflects the saved table file contents

## Limitations

- The 20 ppm threshold is instrument-specific guidance from the Waters Synapt example; other instruments may warrant different cutoffs
- Requires a user-defined list of m/z and RT pairs; untargeted, unknown features cannot be tracked
- Instrument will typically not alert or stop acquisition when calibration is inaccurate, so the monitoring must be run actively (real-time or retrospectively)
- RT deviations across batches can be at least twice the within-batch variation (< 0.05 min short-term), so m/z deviation tracking should not be conflated with RT drift assessment

## Evidence

- [results] Deviations > 20 ppm indicate serious analytical problems: [evidence span withheld — anchor status: not_found]
- [methods] module_TrackCmp extracts observed m/z and RT for a user-defined compound list: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [abstract] QC4Metabolomics tracks compounds and extracts diagnostics in real time or retrospectively: "QC4Metabolomics monitors files as they are acquired or retrospectively by tracking any user-defined compound(s)"
- [results] Instruments typically do not alert when calibration is inaccurate: "The instruments will typically not alert the user or stop the acquisition if the calibration is inaccurate"
- [methods] module_Warner emails warnings when a quality parameter exceeds a threshold: "module_Warner can send emails warning when a quality parameter exceeds a user defined threshold"
