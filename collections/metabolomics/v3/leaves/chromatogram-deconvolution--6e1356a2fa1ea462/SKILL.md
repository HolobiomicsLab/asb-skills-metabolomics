---
name: chromatogram-deconvolution
description: Use when raw LC-HRMS2 chromatograms (converted from Thermo RAW via MSConvert)
  contain co-eluting or partially resolved peaks that the ADAP chromatogram builder
  grouped into a single chromatogram, and distinct peak boundaries must be recovered
  before MS2 export.
when_to_use_negative:
- The input is already a deconvoluted MZmine feature list or feature quantification
  table — no peak-boundary recovery is needed.
- The goal is feature alignment across batches/samples rather than per-sample peak
  resolution; the ENPKG framework deliberately skips RT-based alignment because it
  hinders cross-batch comparison.
- The data are acquired on an instrument whose chromatography or scan rate makes the
  ADAP wavelet defaults inappropriate without re-validation (e.g., very fast separations
  outside the 0.02–0.5 min peak duration range).
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine
  role: Executes the wavelets (ADAP) chromatogram deconvolution step and subsequent
    isotope grouping and feature filtering within the sample-centric ENPKG workflow.
  repo: http://mzmine.github.io/
- name: ProteoWizard
  role: Converts Thermo RAW LC-MS files to mzXML as the entry point for MZmine processing
    prior to deconvolution.
- name: GNPS
  role: Downstream consumer of the 'Export to GNPS' output (feature quantification
    table and .mgf MS2 spectra) produced after deconvolution.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:da16485985505b7392ec1cda3a231bf86e271825df27662c2094074c3c08a778
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MZmine
  - ProteoWizard
  - matchms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# chromatogram-deconvolution

## Summary

Deconvolute co-eluting chromatographic peaks in MZmine feature lists using the ADAP wavelets algorithm, as configured in the ENPKG sample-centric LC-HRMS2 workflow, to produce distinct peaks ready for MS2-linked feature filtering and quantification.

## When to use

Use when raw LC-HRMS2 chromatograms (converted from Thermo RAW via MSConvert) contain co-eluting or partially resolved peaks that the ADAP chromatogram builder grouped into a single chromatogram, and distinct peak boundaries must be recovered before MS2 export.

## When NOT to use

- The input is already a deconvoluted MZmine feature list or feature quantification table — no peak-boundary recovery is needed.
- The goal is feature alignment across batches/samples rather than per-sample peak resolution; the ENPKG framework deliberately skips RT-based alignment because it hinders cross-batch comparison.
- The data are acquired on an instrument whose chromatography or scan rate makes the ADAP wavelet defaults inappropriate without re-validation (e.g., very fast separations outside the 0.02–0.5 min peak duration range).

## Inputs

- Centroided MS1 chromatograms built by the ADAP chromatogram builder inside MZmine (from mzML/mzXML converted Thermo RAW files)
- MS2 spectra (DDA) associated with precursor ions detected in MS1

## Outputs

- Deconvoluted chromatographic peaks (wavelets/ADAP) forming per-sample MZmine feature lists
- Feature quantification table (m/z, RT, area/intensity) and per-feature MS2 .mgf files exported via the 'Export to GNPS' module

## How to apply

After building chromatograms with the ADAP chromatogram builder (minimum group size of 5 scans, minimum group intensity 1.0E4, minimum highest intensity 5.0E5, 12 ppm m/z tolerance), run MZmine's chromatogram deconvolution step with the wavelets (ADAP) algorithm. Set the S/N estimator to the intensity window method with a signal-to-noise ratio of 10, a minimum feature height of 5.0E5, a coefficient area threshold of 50, a peak duration range of 0.02–0.5 min, and an RT wavelet range of 0.01–0.03 min. The deconvoluted peaks are then grouped into isotopes (m/z tolerance 12 ppm, RT tolerance 0.01 min, max charge 2) and filtered to retain only features with an associated MS2 scan and RT between 0.5 and 8.0 min, ensuring that downstream 'Export to GNPS' produces a feature quantification table and per-feature MS2 .mgf spectra. Verify deconvolution succeeded by checking that peak counts are consistent with reported totals (~788,623 in PI, ~364,967 in NI before per-sample filtering) and that each retained feature carries an MS2 scan.

## Related tools

- **MZmine** (Executes the wavelets (ADAP) chromatogram deconvolution step and subsequent isotope grouping and feature filtering within the sample-centric ENPKG workflow.) — http://mzmine.github.io/
- **ProteoWizard** (Converts Thermo RAW LC-MS files to mzXML as the entry point for MZmine processing prior to deconvolution.)
- **GNPS** (Downstream consumer of the 'Export to GNPS' output (feature quantification table and .mgf MS2 spectra) produced after deconvolution.)

## Examples

```
sh workflow/00_workflow_all.sh  # runs the ENPKG pipeline, including MZmine deconvolution, per README
```

## Evaluation signals

- The deconvoluted feature list retains only peaks with an associated MS2 scan and RT between 0.5 and 8.0 min (the article's filter thresholds).
- Each retained feature's MS2 spectra are present in the exported .mgf files and each feature has a corresponding row in the feature quantification table.
- Total feature counts across the data set fall in the expected range (≈788,623 in PI and ≈364,967 in NI modes) before per-sample filtering.
- Isotope grouping is applied with m/z tolerance 12 ppm and RT tolerance 0.01 min, so peak isotopologues are linked rather than split into separate features.
- The minimum feature height (5.0E5) and S/N ratio (10) cutoffs are respected: no deconvoluted peak below the height threshold survives filtering.

## Limitations

- The wavelets (ADAP) parameters (0.02–0.5 min peak duration, 0.01–0.03 min RT wavelet range) were tuned to the ENPKG UHPLC-HRMS2 setup; other chromatographic methods may need re-tuning.
- Co-eluting isomers that produce overlapping MS2 spectra can still be grouped as a single feature if the deconvolution does not fully separate them.
- The article reports no benchmark against alternative deconvolution algorithms, so the chosen parameters lack external validation.

## Evidence

- [methods] Wavelets parameters: "For chromatogram deconvolution, the algorithm used was the wavelets (ADAP)."
- [methods] S/N and height thresholds: "The intensity window S/N was used as an S/N estimator with a signal-to-noise ratio set at 10, a minimum feature height at 5.0E5"
- [methods] Feature filtering rule: "Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min."
- [results] Export step context: "the data for each sample are processed to perform feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine."
- [readme] Per-sample README rationale: [evidence span withheld — anchor status: not_found]
