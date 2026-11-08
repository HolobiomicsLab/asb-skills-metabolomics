---
name: maldi-tof-spectral-ion-count-extraction
description: Use when MALDI-TOF spectra have been acquired over a library or array
  of targets (e.g., 749 yeast SSM colonies or nL droplet arrays) and per-target ion
  intensities at specific fatty acyl m/z values must be tabulated and ratio-ranked.
when_to_use_negative:
- Targets are smaller than 300 μm, where the article states the microscope-guided
  microMS suite is more effective
- The analysis goal is full spectrum identification or de novo peak annotation rather
  than targeted ion-count extraction at a few known m/z values
- Only a handful of samples are involved and building an optical image-guided coordinate
  file via macroMS would add overhead without benefit
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0622
tools:
- name: macroMS
  role: Web platform used to generate device coordinates from optical images and to
    extract ion counts and perform limited data analysis/visualization on the acquired
    MALDI-TOF spectra
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Alternative microscope-guided targeting suite for samples <300 μm; noted as
    more effective for small targets but not used for this macroscopic colony screen
- name: flexImaging
  role: Bruker instrument software cited as unsuitable for large numbers of samples,
    motivating macroMS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:78986d92b2dcddf6f150afe02d7fb8719f37c06b81823ef1cb346a2c7f9511c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - flexImaging
  - macroMS
  - microMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# maldi-tof-spectral-ion-count-extraction

## Summary

Extract ion counts at target m/z values (e.g., C16:1 and C18:1 fatty acyl fragments) from MALDI-TOF mass spectra of hundreds of screened samples using macroMS data-analysis tools, then compute per-sample intensity ratios for screening and ranking.

## When to use

Use when MALDI-TOF spectra have been acquired over a library or array of targets (e.g., 749 yeast SSM colonies or nL droplet arrays) and per-target ion intensities at specific fatty acyl m/z values must be tabulated and ratio-ranked.

## When NOT to use

- Targets are smaller than 300 μm, where the article states the microscope-guided microMS suite is more effective
- The analysis goal is full spectrum identification or de novo peak annotation rather than targeted ion-count extraction at a few known m/z values
- Only a handful of samples are involved and building an optical image-guided coordinate file via macroMS would add overhead without benefit

## Inputs

- MALDI-TOF mass spectra acquired at fixed target positions (custom geometry files from macroMS device coordinate generation)
- Target/colony position map linking spectra to library positions (e.g., 749 S. cerevisiae Gly1250 SSM colonies)
- List of target m/z values for fatty acyl fragment ions (C16:1 and C18:1)

## Outputs

- Per-colony ion-count ratio table (C16:1/C18:1) keyed by colony position, exported to CSV
- Population mean and standard deviation of ratios (expected 3.8 ± 2.9)
- Ranked top-colony statistics (top-five mean 18.1 ± 1.1) with genotype confirmation (G1250S)

## How to apply

Load the deposited MALDI-TOF spectra keyed by colony or target position into a data frame. For each spectrum, use macroMS's data analysis and visualization tools to extract the ion counts at the m/z values corresponding to the compounds of interest — in the yeast FASII Gly1250 SSM screen, the C16:1 and C18:1 fatty acyl fragment peaks. Compute per-colony ratios of the short-chain fragment to the canonical long-chain fragment (C16:1 over 18:1), rank colonies by ratio, and report population statistics (expect 3.8 ± 2.9) alongside the top-ranked subset (expect 18.1 ± 1.1 for the top five colonies, all confirmed as G1250S mutants by DNA sequencing). Write the per-colony ratio table and summary statistics to CSV; validation hinges on reproducing the reported ratios and confirming that high-ratio isolates carry the expected mutation. Spectra are acquired at ~1–2 s per sample, so throughput is sufficient for screens up to the ~10,000-target upper limit.

## Related tools

- **macroMS** (Web platform used to generate device coordinates from optical images and to extract ion counts and perform limited data analysis/visualization on the acquired MALDI-TOF spectra) — github.com/macroMS/macroMS_web
- **microMS** (Alternative microscope-guided targeting suite for samples <300 μm; noted as more effective for small targets but not used for this macroscopic colony screen)
- **flexImaging** (Bruker instrument software cited as unsuitable for large numbers of samples, motivating macroMS)

## Examples

```
# Rank MALDI-TOF spectra by C16:1/C18:1 fatty acyl ion-count ratio per colony
df['ratio'] = df['ion_count_C16:1'] / df['ion_count_C18:1']; ranked = df.sort_values('ratio', ascending=False)
```

## Evaluation signals

- Population C16:1/C18:1 ratio mean and SD reproduce 3.8 ± 2.9 across the 749 colonies
- Top-five colonies' ratio mean and SD reproduce 18.1 ± 1.1
- Top-ranked colony matches the known G1250S genotype when checked by DNA sequencing
- Every colony position in the source spectra map has a corresponding row in the output CSV (no dropped targets)
- Extracted ion counts are taken at the correct fatty acyl m/z values (C16:1 and C18:1 fragment peaks) rather than adjacent peaks

## Limitations

- macroMS supports only limited data analysis; deeper statistical treatment must be done outside the platform
- The upper limit for target number was found to be ~10,000, presented without detailed uncertainty measurement
- No version information for macroMS is stated in the source context
- Comparison with microMS is qualitative; no quantitative benchmark is provided

## Evidence

- [other] Extract the ion counts at the m/z values for C16:1 and C18:1 fatty acyl species from each spectrum using macroMS data analysis tools.: [evidence span withheld — anchor status: not_found]
- [other] The five colonies with the highest C16:1 over 18:1 ratio had an average value of 18.1±1.1, and DNA sequencing demonstrated that all five isolated colonies were G1250S mutants.: [evidence span withheld — anchor status: not_found]
