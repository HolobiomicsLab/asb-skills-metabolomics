---
name: baseline-conditional-quantification-of-weak-signals
description: Use when processing full-scan AEMS data (e.g. precursor-ion or neutral-loss
  scans from 384-/1536-well plates) where a persistent high-intensity baseline signal
  obscures weak analyte peaks, so that frame boundaries must be derived from clear
  high-mass reference peaks rather than from the weak.
when_to_use_negative:
- The acquisition is MRM (<5 transitions per plate) rather than full-scan, since MRM
  transitions do not require reference-peak alignment of frame boundaries to handle
  persistent baseline signals.
- The analyte signal of interest is itself the well-defined high-mass peak that stands
  clearly above the baseline, in which case direct quantification of that peak is
  appropriate without borrowing frames from another signal.
- Input data lack a persistent high-intensity baseline signal, so weak signals are
  not obscured and per-signal direct peak detection and integration are sufficient.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: rtmsEcho
  role: R package that loads AEMS full-scan spectra per well and implements the reference-peak
    alignment used to define frames for quantifying weaker, baseline-obscured signals
  repo: https://github.com/nathanieltwarog/rtmsEcho
- name: rtms
  role: Underlying framework extended by rtmsEcho; provides the mass-spectrometry
    peak handling on which frame-based quantification of weak signals is built
  repo: https://github.com/nathanieltwarog/rtmsEcho
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03730
    title: 'rtmsEcho: An Open-Source R Package for Automated Analysis of Acoustic
      Ejection Mass Spectrometry Data'
  source_content_hash: sha256:57202be693cb0d6c0cdc21c5ee302956639079c88193c29e3fa9333811ca62ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nathanieltwarog/rtmsEcho
  tools:
  - R
  - rtms
  - rtmsEcho
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# baseline-conditional-quantification-of-weak-signals

## Summary

A peak-quantification strategy from the rtmsEcho AEMS pipeline in which well-defined high-mass peaks in each well's full-scan spectrum serve as reference anchors to define the acoustic shot's time frames; those frames are then applied to weaker, lower-intensity signals that would otherwise be obscured by the persistent baseline.

## When to use

Use when processing full-scan AEMS data (e.g. precursor-ion or neutral-loss scans from 384-/1536-well plates) where a persistent high-intensity baseline signal obscures weak analyte peaks, so that frame boundaries must be derived from clear high-mass reference peaks rather than from the weak signal itself.

## When NOT to use

- The acquisition is MRM (<5 transitions per plate) rather than full-scan, since MRM transitions do not require reference-peak alignment of frame boundaries to handle persistent baseline signals.
- The analyte signal of interest is itself the well-defined high-mass peak that stands clearly above the baseline, in which case direct quantification of that peak is appropriate without borrowing frames from another signal.
- Input data lack a persistent high-intensity baseline signal, so weak signals are not obscured and per-signal direct peak detection and integration are sufficient.

## Inputs

- Extracted full-scan AEMS spectra per well (precursor-ion or neutral-loss scan mode, e.g. up to a 60 Da window) from a 384-well or 1536-well plate
- Well/plate and compound metadata for the plate (associated with the raw AEMS data files)
- rtmsEcho plate object in R (result of loading the full-scan run with rtmsEcho's raw-data upload step)

## Outputs

- Structured peak table keyed by well and m/z, containing frame boundaries and integrated area per signal
- Per-well list of detected high-mass reference peaks used as frame anchors
- Flags for ambiguous frame assignments (signals where the reference-peak alignment could not be confidently applied) for expert review

## How to apply

In R with the rtmsEcho package, first load the extracted full-scan spectra for each well of the plate. Detect well-defined, high-intensity high-mass peaks that stand clearly above the persistent baseline signal in each well's spectrum; these serve as reference/anchor points. Use those high-mass reference peaks to define time frames bounding the acoustic shot events for each well. Then apply the defined frames to the weaker, less well-defined signals at other m/z values, performing baseline correction and area integration only within those frames to quantify them. Assemble the frame-based quantifications into a structured peak table keyed by well and m/z, including frame boundaries and integrated area. The rationale is that the weak analyte signal cannot be trusted to define its own window because the persistent baseline signal can obscure it; borrowing a clear high-mass reference peak's timing from the same shot event gives a reliable frame. Validate by confirming frame assignments are consistent with the detected reference peaks, flagging ambiguous assignments for expert review.

## Related tools

- **rtmsEcho** (R package that loads AEMS full-scan spectra per well and implements the reference-peak alignment used to define frames for quantifying weaker, baseline-obscured signals) — https://github.com/nathanieltwarog/rtmsEcho
- **rtms** (Underlying framework extended by rtmsEcho; provides the mass-spectrometry peak handling on which frame-based quantification of weak signals is built) — https://github.com/nathanieltwarog/rtmsEcho

## Examples

```
R: library(rtmsEcho); plate <- loadFullScan('well_spectra.mzML'); peaks <- alignReferencePeaks(plate, mzTargets=weak_analyte_mz, refHighMassPeaks=TRUE); quant <- quantifyWithinFrames(plate, peaks)
```

## Evaluation signals

- Confirm the output table exists with the required fields (well, m/z, frame boundaries, integrated area) for every well and m/z queried.
- Check that frame assignments are consistent with the detected high-mass reference peaks — i.e. each frame actually corresponds to the shot event anchored by a reference peak in the same well.
- Confirm ambiguous frame assignments are flagged (not silently filled) so they can be routed for expert review.
- Verify that each quantified weak signal's integration window lies entirely within a frame bounded by a clear high-mass reference peak from that well's own spectrum.
- Cross-check that processing a full 384-well plate in full-scan mode stays within the reported ~30 s runtime budget, indicating the frame-alignment step did not stall.

## Limitations

- Only signals that can be aligned to a well-defined high-mass reference peak in the same well's spectrum can be confidently quantified; weaker signals with no clear reference anchor are at risk of ambiguous frame assignment.
- Persistent high-intensity baseline signals in the full-scan spectra can obscure analyte peaks, so the method depends on the baseline being distinguishable from the high-mass reference peaks.
- Frame-assignment ambiguity requires expert review; rtmsEcho does not silently resolve it, so a fully automated run may leave some wells unquantified.
- rtmsEcho is an open-source R package licensed GPL-3.0; vendor-specific AEMS software remains the alternative, but the article notes vendor tools are inadequate for large-scale studies.

## Evidence

- [abstract] rtmsEcho can selectively use well-defined high-mass peaks as reference points for peak alignment, subsequently applying those time frames to quantify less well-defined, lower signals.: [evidence span withheld — anchor status: not_found]
- [abstract] This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes) while automating shot-to-peak association and spectral analysis.: "This specialized solution provides direct access to AEMS data, enabling customizable processing of both MRM and full-scan acquisitions (precursor ion and neutral loss modes)"
- [intro] Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window).: "Processing speed for a full 384-well plate takes less than 2 s for MRM runs (<5 transitions) and less than 30 s for full-scan mode runs (up to a 60 Da window)."
- [intro] yet the vendor-specific software solutions remain inadequate for large-scale studies due to four fundamental constraints: "yet the vendor-specific software solutions remain inadequate for large-scale studies due to four fundamental constraints"
- [discussion] By automating critical steps such as peak alignment, baseline correction, and area quantification: "By automating critical steps such as peak alignment, baseline correction, and area quantification"
