---
name: savitzky-golay-peak-smoothing
description: Use when you need to smooth noisy extracted ion chromatograms from mzXML/mzML
  LC-MS metabolomics data before peak detection and per-peak quality metrics (MCQ,
  asymmetry factor, FWHM) are computed.
when_to_use_negative:
- Raw chromatograms that have not undergone IRLS baseline correction — the workflow
  smooths the baseline-corrected EIC, not the raw signal
- Data analysis aimed at quality-control filtering of already-scored peaks (e.g. applying
  Std/Blk > r, RSD < c, intensity > I thresholds) rather than peak-profile smoothing
tools:
- name: MetaMOPE
  role: Containerized web workflow that generates EICs from mzXML/mzML files and applies
    IRLS baseline correction plus Savitzky-Golay smoothing before computing the eight
    per-peak quality metrics
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Language of the source-code functions implementing the smoothing and multiple
    peak-quality metrics, distributed with MetaMOPE
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Container runtime used to deploy the MetaMOPE web tool that performs the smoothing
    step (docker-compose up --build)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:fa1d3f9d13fb1e6eee9de1f68c5f809c1537598e45003738075f7e2852ec00bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# savitzky-golay-peak-smoothing

## Summary

Smoothing of extracted ion chromatograms (EICs) with a Savitzky-Golay filter after IRLS baseline correction, as implemented in the MetaMOPE workflow for fast LC-MS chromatographic peak evaluation in metabolomics.

## When to use

Use when you need to smooth noisy extracted ion chromatograms from mzXML/mzML LC-MS metabolomics data before peak detection and per-peak quality metrics (MCQ, asymmetry factor, FWHM) are computed.

## When NOT to use

- Raw chromatograms that have not undergone IRLS baseline correction — the workflow smooths the baseline-corrected EIC, not the raw signal
- Data analysis aimed at quality-control filtering of already-scored peaks (e.g. applying Std/Blk > r, RSD < c, intensity > I thresholds) rather than peak-profile smoothing

## Inputs

- Extracted ion chromatograms (EICs) of standard analytes derived from an input mzXML or mzML LC-MS file
- Baseline-corrected EICs (after IRLS baseline correction) to be smoothed

## Outputs

- Smoothed EIC peak profiles
- Smoothed profiles serving as input for per-peak metrics: MCQ, maximum intensity, asymmetry factor, FWHM, jaggedness, modality

## How to apply

First read the sample's mzXML/mzML file and generate the standard analytes' EICs over the specified m/z window. Apply IRLS (Iteratively Reweighted Least Squares) baseline correction to each EIC before smoothing, since the Savitzky-Golay filter is applied to the baseline-corrected EIC. Then apply the Savitzky-Golay filter to smooth the peak profile; MetaMOPE ships the smoothing functions as R source code. Smoothed profiles feed downstream peak detection, from which the maximum intensity, mass chromatographic quality (MCQ) index, asymmetry factor, FWHM, jaggedness, and modality are computed. Validation with 20 chemical standards showed the pipeline can build a 571-metabolite reference library for a HILIC LC-MS platform.

## Related tools

- **MetaMOPE** (Containerized web workflow that generates EICs from mzXML/mzML files and applies IRLS baseline correction plus Savitzky-Golay smoothing before computing the eight per-peak quality metrics) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Language of the source-code functions implementing the smoothing and multiple peak-quality metrics, distributed with MetaMOPE) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Container runtime used to deploy the MetaMOPE web tool that performs the smoothing step (docker-compose up --build))

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && docker-compose up --build  # deploy MetaMOPE web tool, whose R functions perform IRLS + Savitzky-Golay EIC smoothing
```

## Evaluation signals

- Baseline correction (IRLS) precedes Savitzky-Golay smoothing in the executed order; applying smoothing before baseline correction deviates from the workflow
- Smoothed EICs should yield a detectable local maximum intensity and a computable MCQ and asymmetry factor for each analyte peak
- Number of peaks and peak retention times are determinable from the mass chromatographic quality value, asymmetric factor, and local maximum intensity of the smoothed EIC
- The per-analyte metrics table (MCQ, max intensity, RT RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) is fully populated and written to CSV
- On validated standards (20 chemical standards; HILIC LC-MS), the workflow constructed a reference library of 571 metabolites, giving a reference point for expected coverage

## Limitations

- The article's discussion section contains no content ('No changelog found'), so limitations, missing artifacts, or reproducibility gaps cannot be extracted from that section
- The smoothing step is described as part of the MetaMOPE workflow without explicit filter window or polynomial-order parameters in the provided context

## Evidence

- [abstract] The extracted ion chromatograms of each chemical standard were retrieved (Supplementary Fig. S2), followed by baseline correction and smoothing of EIC using the IRLS algorithm and Savitzky-Golay: "The extracted ion chromatograms of each chemical standard were retrieved (Supplementary Fig. S2), followed by baseline correction and smoothing of EIC using the IRLS algorithm and Savitzky-Golay"
- [abstract] We calculated the maximum intensity, mass chromatographic quality index (MCQ) and asymmetry factor after IRLS (Iteratively reweighted least squares) baseline correction and Savitzky-Golay smoothing: "We calculated the maximum intensity, mass chromatographic quality index (MCQ) and asymmetry factor after IRLS (Iteratively reweighted least squares) baseline correction and Savitzky-Golay smoothing"
- [abstract] After reading a sample's input mzXML/mzML file, we generated standard analytes' EIC.: [evidence span withheld — anchor status: not_found]
- [readme] A mass chromatographic quality value, an asymmetric factor, and the local maximum intensity of the extracted ion chromatogram were calculated to determine the number of peaks and peak retention time.: "A mass chromatographic quality value, an asymmetric factor, and the local maximum intensity of the extracted ion chromatogram were calculated to determine the number of peaks and peak retention time."
