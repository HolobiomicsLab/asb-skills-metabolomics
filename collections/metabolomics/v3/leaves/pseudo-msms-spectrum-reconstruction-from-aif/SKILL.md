---
name: pseudo-msms-spectrum-reconstruction-from-aif
description: Use when processing centroid-mode LC-MS AIF (DIA) chromatograms where
  the analyst needs a pseudo-MS/MS spectrum for a subset of features of interest,
  e.g. features reaching statistical significance in an untargeted metabolomics experiment.
when_to_use_negative:
- Input data were acquired with a DIA scheme other than plain AIF (e.g. SWATH-MS,
  SONAR) — MetaboAnnotatoR currently supports only AIF and the article flags this
  as future work.
- The analyst needs annotated spectra for all detected features in an untargeted manner
  — MS-DIAL performs untargeted annotation, whereas this workflow is intentionally
  restricted to a user-selected feature subset.
- The target compound's fragments are poorly detected or the feature co-elutes with
  compounds sharing similar chemical groups — the article warns this can yield incomplete
  pseudo-MS/MS spectra.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnnotatoR
  role: R package that implements the pseudo-MS/MS reconstruction step (getPseudoMSMS/RCspec/xcmsSpec)
    for AIF LC-MS data
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment; MetaboAnnotatoR is an open-source R package
- name: RAMClustR
  role: Downstream R dependency used for parent–fragment relationship deconvolution
    feeding the RC object required by RCspec/getPseudoMSMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:87a34a2a313f948e0d925ce5bb633a1dfa0700c84735d5827479e54010e827aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  - RAMClustR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pseudo-msms-spectrum-reconstruction-from-aif

## Summary

Reconstruct an in-source pseudo-MS/MS spectrum for a target LC-MS feature from all-ion fragmentation (AIF) data by correlating its MS1 extracted ion chromatogram (EIC) with high-collision-energy AIF EICs (MetaboAnnotatoR step 2, using xcmsSpec/RCspec/getPseudoMSMS), retaining co-eluting fragment peaks above a correlation threshold.

## When to use

Use when processing centroid-mode LC-MS AIF (DIA) chromatograms where the analyst needs a pseudo-MS/MS spectrum for a subset of features of interest, e.g. features reaching statistical significance in an untargeted metabolomics experiment.

## When NOT to use

- Input data were acquired with a DIA scheme other than plain AIF (e.g. SWATH-MS, SONAR) — MetaboAnnotatoR currently supports only AIF and the article flags this as future work.
- The analyst needs annotated spectra for all detected features in an untargeted manner — MS-DIAL performs untargeted annotation, whereas this workflow is intentionally restricted to a user-selected feature subset.
- The target compound's fragments are poorly detected or the feature co-elutes with compounds sharing similar chemical groups — the article warns this can yield incomplete pseudo-MS/MS spectra.

## Inputs

- Raw LC-MS AIF chromatograms in centroid mode
- xcmsSet object (xset) from xcms feature detection
- RAMClustR object (RC) from RamClustR processing
- Target feature (m/z and retention time) selected by prior statistical analysis

## Outputs

- Pseudo-MS/MS spectrum for the target feature (retained AIF fragment peaks and intensities)
- pseudo_msms_spectrum.rda file exported for downstream annotation steps

## How to apply

Start from an xcmsSet (xset) and RAMClustR (RC) object built from centroid-mode AIF chromatograms, select the target feature's MS1 EIC, and extract the high-collision-energy AIF EICs for every AIF scan spanning the run using xcmsSpec. Compute the Pearson correlation between the target's MS1 EIC and each candidate AIF fragment EIC (RCspec/getPseudoMSMS); peaks whose correlation exceeds the user-defined threshold θ (default 0.8; 0.7–0.8 typical) are assembled into the pseudo-MS/MS spectrum. If no AIF peak passes θ, fall back to the full AIF spectrum taken at the feature's retention time rather than emitting an empty spectrum. The rationale is that peak-shape correlation, not just RT matching, captures which fragments truly arise from the parent, which the article notes is more robust than plain peak-shape matching for co-eluting compounds. Export the resulting spectrum (e.g. to pseudo_msms_spectrum.rda) for downstream fragment-library matching in MetaboAnnotatoR steps 3–5.

## Related tools

- **MetaboAnnotatoR** (R package that implements the pseudo-MS/MS reconstruction step (getPseudoMSMS/RCspec/xcmsSpec) for AIF LC-MS data) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment; MetaboAnnotatoR is an open-source R package)
- **RAMClustR** (Downstream R dependency used for parent–fragment relationship deconvolution feeding the RC object required by RCspec/getPseudoMSMS)

## Examples

```
library(MetaboAnnotatoR); pseudo_msms <- getPseudoMSMS(xset, RC, feature_id, cor.threshold = 0.8)
```

## Evaluation signals

- Reconstructed pseudo-MS/MS must match the spectra produced by the paper's reference implementation (getPseudoMSMS/RCspec/xcmsSpec) on the shipped example xset/RC objects.
- Every AIF fragment peak in the exported spectrum must have a Pearson correlation with the target MS1 EIC greater than the chosen threshold θ (default 0.8).
- If no AIF peak passes θ, the exported spectrum must equal the full AIF scan at the feature's retention time, not an empty spectrum.
- The output file pseudo_msms_spectrum.rda must be produced and loadable in R for the subsequent MetaboAnnotatoR annotation steps.
- Peaks not captured in the pseudo-MS/MS can still be searched in raw AIF scans, so a spot check against the original AIF spectra should show no false retention of unrelated co-eluting ions.

## Limitations

- Only plain AIF acquisition is supported; other DIA schemes such as SWATH are not yet implemented.
- Peak-shape/correlation deconvolution can produce incomplete pseudo-MS/MS spectra for co-eluting compounds with similar chemical groups that yield nonspecific fragments, or when fragment peaks are poorly detected.
- Correlation threshold is user-tunable (default 0.8; 0.7–0.8 typical) — too high a θ discards genuine low-correlation fragments, too low admits nonspecific co-eluting ions.
- Requires raw chromatograms in centroid mode; profile-mode data must be transformed first.
- Annotation on higher mass-accuracy instruments (e.g. Orbitraps) is stated as possible but was not validated in the article.

## Evidence

- [methods] Pseudo-MS/MS generation computes Pearson correlations between the target feature's MS1 EIC and each AIF-scan EIC; AIF peaks with r above a user-set threshold θ (typically 0.7–0.8, default 0.8) are gathered into the pseudo-MS/MS spectrum: [evidence span withheld — anchor status: not_found]
- [methods] If no AIF peak passes θ, the full AIF spectrum at the feature RT is used instead.: [evidence span withheld — anchor status: not_found]
- [intro] Two R packages, CAMERA10 and RAMClustR,5 can also be used to reconstruct in-source parent-fragment pseudo-MS/MS spectra: "Two R packages, CAMERA10 and RAMClustR,5 can also be used to reconstruct in-source parent-fragment"
- [intro] to get meaningful information from AIF datasets, special processing routines are needed to reconstruct the parent–fragment ion relationships and obtain a spectrum containing all fragments arising from the same parent (pseudo-MS/MS spectrum): "special processing routines are needed to reconstruct the parent–fragment ion relationships and obtain a spectrum containing all fragments"
- [intro] These can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds containing similar types of chemical groups, which upon fragmentation produce nonspecific fragment ions: "These can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds containing similar types of chemical groups"
- [readme] It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode.: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [readme] Introductory vignette: an example of feature annotation using LC-MS AIF chromatograms processed using xcms and RamClustR packages, as described in the MetaboAnnotatoR paper.: "an example of feature annotation using LC-MS AIF chromatograms processed using xcms and RamClustR packages"
- [abstract] All-ion fragmentation (AIF) LC–MS/MS acquisition provides fragmentation data at no additional experimental time cost: "All-ion fragmentation (AIF) LC–MS/MS acquisition provides fragmentation data at no additional experimental time cost"
- [discussion] Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH.: "Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH"
- [discussion] The annotation of AIF datasets acquired on higher mass accuracy MS instruments (e.g., Orbitraps), although not tested here, is also possible using MetaboAnnotatoR.: "The annotation of AIF datasets acquired on higher mass accuracy MS instruments (e.g., Orbitraps), although not tested here"
