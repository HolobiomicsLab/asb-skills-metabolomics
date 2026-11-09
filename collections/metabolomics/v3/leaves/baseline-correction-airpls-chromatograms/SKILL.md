---
name: baseline-correction-airpls-chromatograms
description: Use when raw extracted-ion chromatograms of targeted MS metabolites (or
  their isotopologues) contain drifting baselines that must be removed prior to peak
  detection and peak-area quantification.
when_to_use_negative:
- Input is already a peak/feature table with baseline correction already applied —
  airPLS would be redundant or destructive.
- The goal is untargeted, whole-run peak detection rather than targeted quantification
  of a specific metabolite and its isotopologues.
- Data are MS/MS or data-independent-acquisition spectra — the README lists 'Support
  data-independent acquisition' as a planned future feature, so this pipeline does
  not yet cover it.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: airPLS
  role: Removes the baselines of all the EICs before peak detection
- name: TarMet
  role: Shiny application whose extraction pipeline invokes airPLS baseline correction,
    then MassSpecWavelet peak detection and isotopic-similarity selection
  repo: https://github.com/hcji/TarMet
- name: MassSpecWavelet
  role: Consumes the airPLS-corrected EICs to detect peak positions (bounds estimated
    via Haar wavelet derivative)
- name: enviPat
  role: Provides theoretical isotope patterns used downstream to score peaks against
    detected isotope patterns
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-018-1363-7
    title: 'TarMet: a reactive GUI tool for efficient and confident quantification
      of MS based targeted metabolic and stable isotope tracer analysis'
  source_content_hash: sha256:39ddf13af954ef4e36b3baed9bb4fbf8269132fbc7f94144dcd7f08a7494bbc1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/TarMet
  tools:
  - MassSpecWavelet
  - TarMet
  - airPLS
  - enviPat
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# baseline-correction-airpls-chromatograms

## Summary

Baseline-removal step of TarMet's targeted isotopologue extraction: every extracted-ion chromatogram (EIC) for a target metabolite and its isotopologues is detrended with the airPLS method before MassSpecWavelet peak detection and isotopic-similarity peak selection.

## When to use

Use when raw extracted-ion chromatograms of targeted MS metabolites (or their isotopologues) contain drifting baselines that must be removed prior to peak detection and peak-area quantification.

## When NOT to use

- Input is already a peak/feature table with baseline correction already applied — airPLS would be redundant or destructive.
- The goal is untargeted, whole-run peak detection rather than targeted quantification of a specific metabolite and its isotopologues.
- Data are MS/MS or data-independent-acquisition spectra — the README lists 'Support data-independent acquisition' as a planned future feature, so this pipeline does not yet cover it.

## Inputs

- Extracted-ion chromatograms (EICs) for the target metabolite and its isotopologues
- Theoretical isotope patterns computed by enviPat at the MS resolution (used downstream, not by airPLS itself)

## Outputs

- Baseline-corrected EIC traces ready for MassSpecWavelet peak-position detection
- Downstream: peak-position/bounds/area table annotated with per-peak isotopic similarity values

## How to apply

Load the EICs extracted for the target metabolite (and any co-eluting or isotopologue traces) as the first stage of TarMet's extraction pipeline, alongside the theoretical isotope patterns computed by enviPat at the instrument resolution. Apply the airPLS (asymmetrically reweighted penalized least squares) algorithm to each EIC in turn so that the slowly varying baseline is removed while sharp chromatographic peaks are preserved, using the airPLS implementation that ships with the TarMet R package. Feed the baseline-corrected traces downstream without smoothing decisions of your own: TarMet next detects peak positions with the MassSpecWavelet package, finds peak bounds via a Haar-wavelet-derived valley, and integrates peak areas. Rationale: removing the baseline first improves the signal-to-noise ratio of the wavelet-based peak detection and bounds estimation, which is what enables TarMet to detect and quantify more isotopologues than state-of-the-art methods.

## Related tools

- **airPLS** (Removes the baselines of all the EICs before peak detection)
- **TarMet** (Shiny application whose extraction pipeline invokes airPLS baseline correction, then MassSpecWavelet peak detection and isotopic-similarity selection) — https://github.com/hcji/TarMet
- **MassSpecWavelet** (Consumes the airPLS-corrected EICs to detect peak positions (bounds estimated via Haar wavelet derivative))
- **enviPat** (Provides theoretical isotope patterns used downstream to score peaks against detected isotope patterns)

## Examples

```
install.packages(c("BiocManager","enviPat","shiny","plotly")); BiocManager::install("MassSpecWavelet"); install_github("hcji/TarMet"); library(TarMet); runTarMet()
```

## Evaluation signals

- After correction, the EIC baseline should be flat in non-peak regions while sharp chromatographic peak shapes are preserved.
- Downstream MassSpecWavelet detection should yield peak positions whose bounds are resolvable via the Haar-wavelet valley, indicating sufficient post-correction SNR.
- The peak finally selected as the target must show the maximum isotopic similarity between theoretical (enviPat) and detected isotope patterns among all candidate peaks.
- Resulting quantification should reproduce the reported behavior that TarMet detects more isotopologues and quantifies them better than state-of-the-art methods on validation datasets (e.g. the MM14 and AssayR cellular metabolites datasets).

## Limitations

- airPLS baseline correction is described only as applied to EICs within a targeted workflow; its behavior on untargeted full-scan chromatograms is not characterized in this source.
- The pipeline's confidence depends on the downstream isotopic-similarity selection, so a poor baseline correction can cascade into wrong peak-bounds estimation and mis-selection of the target peak.
- MS/MS library verification and data-independent acquisition support are only planned future features, limiting applicability to DIA/MS/MS data.

## Evidence

- [abstract] airPLS removes EIC baselines before wavelet peak detection: "baselines of all the EICs are removed by airPLS method"
- [abstract] TarMet's full extraction pipeline context: baseline removal feeds MassSpecWavelet peak detection: "Peak positions are detected by MassSpecWavelet package"
- [readme] TarMet is an R/shiny application; airPLS correction is performed within its targeted MS analysis pipeline: "TarMet is a shiny application for targeted metabolic analyses based on mass spectrometry."
- [readme] MassSpecWavelet is installed via Bioconductor as a dependency of the TarMet pipeline: "BiocManager::install("MassSpecWavelet")"
- [abstract] enviPat provides the theoretical isotope patterns used alongside the corrected EICs: "TarMet will calculate the theoretical isotope patterns of the compounds based on the resolution of the MS via enviPat package"
