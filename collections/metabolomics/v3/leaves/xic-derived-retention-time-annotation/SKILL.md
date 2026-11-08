---
name: xic-derived-retention-time-annotation
description: Use when you have Thermo .raw files containing spiked-in Biognosys iRT
  peptides and need observed retention times or an rt-vs-iRT-score calibration line
  for use-case style targeted analysis in R.
when_to_use_negative:
- The input is not a vendor .raw file (e.g. already-converted mzML or a precomputed
  chromatogram) — rawrr's readChromatogram wraps the vendor RawFileReader API and
  requires proprietary Thermo raw files.
- You need retention times from MS2-level fragment ion XICs rather than MS1 precursor
  XICs — then the filter string must be an MS2 filter (e.g. 'FTMS + c NSI Full ms2
  487.2567@hcd27.00 [100.0000-1015.0000]'), not filter = 'ms'.
- Your sample lacks spiked iRT peptide standards, so no iRT-score calibration reference
  exists.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rawrr
  role: Computes XICs for iRT peptide precursors from Thermo raw files via rawrr::readChromatogram
    with 10 ppm tolerance and MS1 filter
  repo: https://github.com/fgcz/rawrr
- name: RawFileReader
  role: Vendor-provided .NET API wrapped by rawrr to access spectral and chromatographic
    data in .raw files
  repo: https://github.com/thermofisherlsms/RawFileReader
- name: R (stats::lm)
  role: Fits the ordinary least squares rt ~ iRTscore calibration model and extracts
    intercept, slope, and R-squared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00866
    title: 'The rawrr R Package: Direct Access to Orbitrap Data and Beyond'
  source_content_hash: sha256:b3440c9f1f929b82883e6e72ff9f9c2174b45d287a0fb78bb704504c0d4d157a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fgcz/rawrr
  tools:
  - R
  - R (stats::lm)
  - RawFileReader
  - rawrr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# xic-derived-retention-time-annotation

## Summary

Derives observed retention times for iRT peptide precursors by computing extracted ion chromatograms (XICs) directly from Thermo Fisher Orbitrap raw files with the rawrr R package, then fits a linear rt-vs-iRT-score calibration model. It anchors spectral data to a shared retention-time scale for targeted proteomics workflows.

## When to use

Use when you have Thermo .raw files containing spiked-in Biognosys iRT peptides and need observed retention times or an rt-vs-iRT-score calibration line for use-case style targeted analysis in R.

## When NOT to use

- The input is not a vendor .raw file (e.g. already-converted mzML or a precomputed chromatogram) — rawrr's readChromatogram wraps the vendor RawFileReader API and requires proprietary Thermo raw files.
- You need retention times from MS2-level fragment ion XICs rather than MS1 precursor XICs — then the filter string must be an MS2 filter (e.g. 'FTMS + c NSI Full ms2 487.2567@hcd27.00 [100.0000-1015.0000]'), not filter = 'ms'.
- Your sample lacks spiked iRT peptide standards, so no iRT-score calibration reference exists.

## Inputs

- Thermo Fisher Scientific Orbitrap .raw file containing iRT peptide mix (Biognosys) spiked into a tryptic BSA digest (e.g. 20181113_010_autoQC01.raw from MSV000086542 or ExperimentHub EH4547)
- Vector of iRT peptide precursor m/z values (iRTmz)
- Biognosys iRT scores for the 11 iRT peptides

## Outputs

- Observed retention times for the 11 iRT peptides derived from MS1-level XICs
- Fitted linear calibration model (intercept, slope, R-squared), expected ~ rt = 17.85 + 0.11 * score, R2 = 0.99
- Calibration summary table, e.g. irt_calibration_summary.csv

## How to apply

Load the raw file with rawrr and compute XICs for the 11 iRT peptide precursor m/z values using rawrr::readChromatogram with tol = 10 (ppm mass tolerance), type = 'xic', and filter = 'ms' so only MS1-level scans are used. Pick, for each precursor, the chromatographic peak (apex) whose XIC signal corresponds to the eluting iRT peptide to obtain its observed retention time. Pair these observed times with the vendor's Biognosys iRT scores and fit an ordinary least squares model lm(rt ~ iRTscore) with R's stats::lm, extracting intercept, slope, and R-squared from summary.lm. Judge correctness by comparing against the reported calibration line rt = 17.85 + 0.11 * score with R-squared 0.99 — the high R-squared confirms the iRT retention times behave highly linearly; write the coefficients, R-squared, and verdict to a summary file (e.g. irt_calibration_summary.csv).

## Related tools

- **rawrr** (Computes XICs for iRT peptide precursors from Thermo raw files via rawrr::readChromatogram with 10 ppm tolerance and MS1 filter) — https://github.com/fgcz/rawrr
- **RawFileReader** (Vendor-provided .NET API wrapped by rawrr to access spectral and chromatographic data in .raw files) — https://github.com/thermofisherlsms/RawFileReader
- **R (stats::lm)** (Fits the ordinary least squares rt ~ iRTscore calibration model and extracts intercept, slope, and R-squared)

## Examples

```
C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms"); fit <- summary(lm(rt ~ iRTscore))
```

## Evaluation signals

- Fitted slope and intercept match the reported calibration line rt = 17.85 + 0.11 * score within tolerance
- R-squared of the lm(rt ~ iRTscore) fit is ~0.99, confirming highly linear retention-time behavior
- All 11 iRT peptides yield a detectable MS1 XIC peak at 10 ppm mass tolerance
- XIC extraction used only MS1-level scans (filter = 'ms'), verified by inspecting the readChromatogram call/filter string
- Output CSV contains fitted coefficients, R-squared, and a verification verdict

## Limitations

- Retention times come from XICs computed with a fixed 10 ppm mass tolerance on MS1-level scans; stricter/looser tolerance may change peak assignment.
- rawrr at the time of publication was not yet submitted to Bioconductor and lacked exchangeable backend and parallel computation support.
- Calibration constants (17.85, 0.11) are specific to the iRT/Biognosys system and LC gradient used (20 min linear C18 gradient at 300 nl/min); re-fit for other setups.

## Evidence

- [results] XIC extraction with 10 ppm mass tolerance using only MS1-level scans: "C <- rawrr::readChromatogram(rawfile, mass = iRTmz, tol = 10, type = "xic", filter = "ms")"
- [results] iRT precursor XICs as the mechanism for retention-time annotation: "this is achieved by computing an extracted ion chromatogram (XIC) for iRT peptide precursors, provided they were spiked into the sample matrix"
- [methods] Sample composition used for the iRT retention-time analysis: "The analyzed sample consists of the iRT peptide mix (Biognosys) in a tryptic BSA digest (NEB)"
- [readme] rawrr provides the raw-file access needed for direct XIC computation: "The package provides access to proprietary Thermo Fisher Scientific Orbitrap instrument data as a stand-alone R package"
