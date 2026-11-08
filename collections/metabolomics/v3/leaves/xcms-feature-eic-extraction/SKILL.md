---
name: xcms-feature-eic-extraction
description: Use when you have centroid-mode AIF LC-MS chromatograms already processed
  with xcms and RAMClustR (xcmsSet/RC objects) and need per-feature MS1 and high-collision-energy
  EIC peak shapes to build pseudo-MS/MS spectra.
when_to_use_negative:
- The input is a feature intensity table or already-assembled MS/MS spectra rather
  than raw centroid-mode AIF chromatograms processed with xcms/RAMClustR
- The data were acquired with a DIA scheme other than AIF (e.g., SWATH), since MetaboAnnotatoR
  currently supports only AIF
- The chromatograms are profile mode rather than centroid mode, which MetaboAnnotatoR
  does not accept
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: Supplies xcmsSpec, RCspec, and getPseudoMSMS functions to extract EICs from
    xcms/RAMClustR objects and build pseudo-MS/MS spectra
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: xcms
  role: Produces the xcmsSet feature list whose per-feature EICs are extracted by
    xcmsSpec
- name: RAMClustR
  role: Supplies the RC object used together with the xcmsSet for EIC extraction and
    correlation-based parent-fragment linking
- name: R
  role: Runtime environment in which the MetaboAnnotatoR package and its EIC-extraction
    functions are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:898479485b5b9a4f6e3c368caf4f6137ed158d5de3b46580ef64a3fda3bb229b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  - RAMClustR
  - xcms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# xcms-feature-eic-extraction

## Summary

Extract the MS1 and AIF EICs for each detected feature from xcms/RAMClustR outputs using MetaboAnnotatoR's xcmsSpec, so peak-shape correlation can link parent and fragment ions and reconstruct pseudo-MS/MS spectra from all-ion fragmentation LC-MS data.

## When to use

Use when you have centroid-mode AIF LC-MS chromatograms already processed with xcms and RAMClustR (xcmsSet/RC objects) and need per-feature MS1 and high-collision-energy EIC peak shapes to build pseudo-MS/MS spectra.

## When NOT to use

- The input is a feature intensity table or already-assembled MS/MS spectra rather than raw centroid-mode AIF chromatograms processed with xcms/RAMClustR
- The data were acquired with a DIA scheme other than AIF (e.g., SWATH), since MetaboAnnotatoR currently supports only AIF
- The chromatograms are profile mode rather than centroid mode, which MetaboAnnotatoR does not accept

## Inputs

- xcmsSet object (xcms feature list) from centroid-mode LC-MS AIF chromatograms
- RAMClustR (RC) object with feature-to-fragment grouping
- Target feature(s) selected from the xcmsSet for EIC extraction

## Outputs

- MS1 EIC peak-shape profile for the target feature
- High-collision-energy AIF EIC peak-shape profiles for candidate fragments
- Reconstructed pseudo-MS/MS spectrum (assembled from AIF peaks with Pearson r above the 0.8 threshold), exported as pseudo_msms_spectrum.rda

## How to apply

Load the xcmsSet and RC (RAMClustR) objects shipped with MetaboAnnotatoR into R, then call xcmsSpec on a chosen target feature to pull the MS1 EIC and the high-collision-energy AIF EICs from the centroid-mode LC-MS AIF chromatograms. These peak-shape profiles are the inputs to RCspec/getPseudoMSMS, which computes Pearson correlations between the target MS1 EIC and each AIF fragment EIC; only AIF peaks whose correlation exceeds the default threshold of 0.8 are retained and assembled into the pseudo-MS/MS spectrum, which is exported as pseudo_msms_spectrum.rda. The rationale, from the MetaboAnnotatoR paper, is that EIC-based correlation deconvolution reconstructs the parent-fragment ion relationships needed to give AIF features meaningful fragment information, and that using EICs extracted directly from xcms/RAMClustR outputs enables the workflow to be applied to features obtained from XCMS or similar software without reprocessing the raw data.

## Related tools

- **MetaboAnnotatoR** (Supplies xcmsSpec, RCspec, and getPseudoMSMS functions to extract EICs from xcms/RAMClustR objects and build pseudo-MS/MS spectra) — https://github.com/gggraca/MetaboAnnotatoR
- **xcms** (Produces the xcmsSet feature list whose per-feature EICs are extracted by xcmsSpec)
- **RAMClustR** (Supplies the RC object used together with the xcmsSet for EIC extraction and correlation-based parent-fragment linking)
- **R** (Runtime environment in which the MetaboAnnotatoR package and its EIC-extraction functions are executed)

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE, INSTALL_opts = c('--no-multiarch', '--no-lock'))
```

## Evaluation signals

- The extracted MS1 and AIF EIC peak shapes reproduce the spectra produced by the paper's getPseudoMSMS/RCspec/xcmsSpec reference implementation on the example xset and RC objects
- Only AIF fragment peaks with Pearson r above the default 0.8 threshold are included in the pseudo-MS/MS spectrum
- If no AIF EIC exceeds the 0.8 threshold, the full AIF spectrum at the feature RT is used instead of an empty spectrum
- The reconstructed spectrum is exported as a valid pseudo_msms_spectrum.rda object that can be loaded back into R
- Annotation of the reconstructed pseudo-MS/MS spectra yields precision in the 82-92% and recall in the 82-85% range on the three human serum datasets (high-rank scores 1-5)

## Limitations

- Only works on centroid-mode raw LC-MS AIF chromatograms processed with xcms and RAMClustR; profile-mode data or other feature-finding tools require conversion or preprocessing first
- Peak shape correlation deconvolution can yield incomplete pseudo-MS/MS spectra for co-eluting compounds with similar chemical groups (nonspecific fragment ions) or poorly detected fragment peaks
- Installation can fail with 'mzR has been built against a different Rcpp version'; set Sys.setenv(R_REMOTES_NO_ERRORS_FROM_WARNINGS="true") or use --no-multiarch/--no-lock

## Evidence

- [methods] Pseudo-MS/MS generation computes Pearson correlations between the target feature's MS1 EIC and each AIF-scan EIC: [evidence span withheld — anchor status: not_found]
- [methods] AIF peaks with r above a user-set threshold θ (typically 0.7-0.8, default 0.8): [evidence span withheld — anchor status: not_found]
- [abstract] Combining correlation-based parent-fragment linking with molecular fragment matching: "combining correlation-based
parent–fragment linking with molecular fragment matching"
- [intro] Special processing routines are needed to reconstruct the parent-fragment ion relationships: "to get meaningful information from AIF datasets, special
processing routines are needed to reconstruct the parent–fragment
ion relationships and obtain a spectrum containing all fragments"
- [discussion] This enables the direct application of the workflow to features obtained from XCMS or similar software.: "This enables the direct application of the workflow to features obtained from XCMS or similar software."
- [readme] It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode.: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
