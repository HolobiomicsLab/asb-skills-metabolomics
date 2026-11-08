---
name: eic-peak-shape-correlation-deconvolution
description: Use when raw LC-MS all-ion fragmentation (AIF) centroid chromatograms
  are available and parent–fragment ion relationships must be deconvoluted from peak-shape
  correlation before fragment-library matching or feature annotation.
when_to_use_negative:
- The workflow expects raw AIF chromatograms; do not use this skill when the input
  is already a deconvoluted pseudo-MS/MS spectrum or a feature table from a DIA scheme
  such as SWATH, since only AIF is currently supported.
- In-source fragment correlation deconvolution requires fragment peaks detected in
  AIF scans; if the target feature has no AIF peaks above the threshold (e.g., very
  low intensity), the fallback full AIF spectrum should be inspected manually instead
  of forcing the threshold.
- This is not a targeted MS/MS spectral-matching skill; do not apply it to pre-acquired
  MS/MS spectra (e.g., DDA or library spectra in METLIN/MassBank format).
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MetaboAnnotatoR
  role: R package providing getPseudoMSMS, RCspec, and xcmsSpec functions that perform
    the EIC peak-shape correlation deconvolution and assemble the pseudo-MS/MS spectrum.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: RAMClustR
  role: Used alongside MetaboAnnotatoR to reconstruct in-source parent–fragment pseudo-MS/MS
    spectra; its RC object supplies the AIF EICs correlated against the target feature's
    MS1 EIC.
- name: xcms
  role: xcmsSet object holds the feature detection results from raw centroided AIF
    chromatograms; its MS1 EICs are the correlation reference for each fragment peak.
- name: MS-DIAL
  role: Benchmark tool for AIF parent-fragment deconvolution; results are compared
    against MetaboAnnotatoR's correlation-based approach.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:52f05eb59bebbe5322c5d2459330f33f73344d551aa6682b72b152ff16f6033f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  - RAMClustR
  - xcms
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# EIC peak-shape correlation deconvolution

## Summary

Reconstructs pseudo-MS/MS spectra from AIF LC–MS data by computing Pearson correlations between a target feature's MS1 EIC and each AIF-scan EIC, linking only co-eluting fragment peaks above a user-defined threshold (default 0.8) so that in-source fragments are assigned to the correct parent ion.

## When to use

Use when raw LC-MS all-ion fragmentation (AIF) centroid chromatograms are available and parent–fragment ion relationships must be deconvoluted from peak-shape correlation before fragment-library matching or feature annotation.

## When NOT to use

- The workflow expects raw AIF chromatograms; do not use this skill when the input is already a deconvoluted pseudo-MS/MS spectrum or a feature table from a DIA scheme such as SWATH, since only AIF is currently supported.
- In-source fragment correlation deconvolution requires fragment peaks detected in AIF scans; if the target feature has no AIF peaks above the threshold (e.g., very low intensity), the fallback full AIF spectrum should be inspected manually instead of forcing the threshold.
- This is not a targeted MS/MS spectral-matching skill; do not apply it to pre-acquired MS/MS spectra (e.g., DDA or library spectra in METLIN/MassBank format).

## Inputs

- Raw LC-MS AIF chromatograms in centroid mode (processed into an xcmsSet object)
- RAMClustR (RC) object from the same experiment
- Target feature (m/z and RT) selected for pseudo-MS/MS reconstruction
- User-defined Pearson correlation threshold θ (default 0.8)

## Outputs

- Reconstructed pseudo-MS/MS spectrum for the target feature
- Exported pseudo_msms_spectrum.rda object
- Assignment of AIF fragment peaks to the parent ion based on co-elution correlation

## How to apply

Load the xcmsSet (xset) and RAMClustR (RC) objects processed from raw centroided AIF chromatograms, choose a target feature, and extract its MS1 EIC with xcmsSpec plus the high-collision-energy AIF EICs with RCspec. Compute the Pearson correlation between the target feature's MS1 EIC and every candidate AIF fragment EIC; retain only AIF peaks whose r exceeds the user-set threshold θ (default 0.8) and assemble those fragments into the pseudo-MS/MS spectrum. If no AIF peak qualifies, fall back to the full AIF spectrum acquired at the feature's retention time. This correlation-based deconvolution is specifically designed for AIF data, where parent–fragment relationships are not known a priori and must be recovered by inspecting which fragment peaks co-elute with the parent ions.

## Related tools

- **MetaboAnnotatoR** (R package providing getPseudoMSMS, RCspec, and xcmsSpec functions that perform the EIC peak-shape correlation deconvolution and assemble the pseudo-MS/MS spectrum.) — https://github.com/gggraca/MetaboAnnotatoR
- **RAMClustR** (Used alongside MetaboAnnotatoR to reconstruct in-source parent–fragment pseudo-MS/MS spectra; its RC object supplies the AIF EICs correlated against the target feature's MS1 EIC.)
- **xcms** (xcmsSet object holds the feature detection results from raw centroided AIF chromatograms; its MS1 EICs are the correlation reference for each fragment peak.)
- **MS-DIAL** (Benchmark tool for AIF parent-fragment deconvolution; results are compared against MetaboAnnotatoR's correlation-based approach.)

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE); getPseudoMSMS(xset, RC, feature, corr.threshold = 0.8)
```

## Evaluation signals

- Confirm that only AIF fragment peaks with Pearson correlation above the chosen threshold (default 0.8) are present in the pseudo-MS/MS spectrum.
- Verify that if no AIF peak exceeds θ, the output is the full AIF spectrum at the feature RT rather than an empty spectrum.
- Compare the reconstructed spectrum against the reference implementation on the paper's example xset/RC objects — output should match the getPseudoMSMS/RCspec/xcmsSpec pipeline result.
- Check that parent–fragment linking improves when fragment peak shapes truly co-elute (similar RT at max intensity and peak height at half-maximum), not when peaks merely share a nominal RT window.
- Export the result as pseudo_msms_spectrum.rda and confirm it can be loaded back into R without error.

## Limitations

- Correlation-based deconvolution can produce incomplete pseudo-MS/MS spectra for co-eluting compounds with similar chemical groups that produce nonspecific fragment ions or for poorly detected fragment peaks.
- Low-intensity fragment peaks below the correlation threshold are excluded; peaks not captured by pseudo-MS/MS must be searched separately in the AIF scans.
- Only AIF acquisition is supported; other DIA schemes (SWATH) and higher-mass-accuracy instruments (Orbitraps) were not validated in the paper.
- Variable precision (60–89%) and recall (10–88%) were observed for non-serum matrices, partly because small nonlipid fragment libraries limit recall.

## Evidence

- [methods] Correlation-based pseudo-MS/MS reconstruction step: [evidence span withheld — anchor status: not_found]
- [methods] Fallback when no fragment passes threshold: [evidence span withheld — anchor status: not_found]
- [intro] Why special processing is needed for AIF data: "to get meaningful information from AIF datasets, special processing routines are needed to reconstruct the parent–fragment ion relationships and obtain a spectrum containing all fragments arising"
- [intro] Known failure mode of correlation deconvolution: "These can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds containing similar types of chemical groups, which upon fragmentation produce nonspecific fragment ions"
- [readme] Installation and vignette guidance from README: "This R package is designed to perform metabolite annotation of features from LC-MS All-ion fragmentation (AIF) datasets, using ion fragment databases. It requires raw LC-MS AIF chromatograms"
