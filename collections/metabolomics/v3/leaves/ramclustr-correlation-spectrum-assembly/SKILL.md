---
name: ramclustr-correlation-spectrum-assembly
description: Use when reconstructing an in-source fragment pseudo-MS/MS spectrum from
  centroided AIF LC–MS chromatograms processed with xcms/RAMClustR — e.g. when annotating
  isotopologues, adducts, or in-source fragments from AIF datasets where no targeted
  MS/MS acquisition exists.
when_to_use_negative:
- The input is already a feature table (m/z, RT, intensity) without raw chromatographic
  profiles — the correlation method requires EIC peak-shape profiles, not summary
  intensities
- The data were acquired with a non-AIF DIA scheme (e.g. SWATH) or on instruments
  where AIF support has not been validated (e.g. Orbitraps)
- A targeted MS/MS spectrum already exists for the feature — direct spectral matching
  is preferable to reconstructing an in-source pseudo-MS/MS
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: RAMClustR
  role: Provides the RCspec/getPseudoMSMS correlation framework that links AIF fragment
    peaks to the parent by EIC correlation
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MetaboAnnotatoR
  role: R package that wraps the correlation-based pseudo-MS/MS assembly and downstream
    fragment-library matching; the assembled spectrum is exported to pseudo_msms_spectrum.rda
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: xcms
  role: Generates the xcmsSet feature table and supplies the MS1 EIC extraction (xcmsSpec)
    used for the correlation calculation
- name: R
  role: Runtime environment for running the RAMClustR/MetaboAnnotatoR pipeline (install
    via install_github)
  repo: https://github.com/gggraca/MetaboAnnotatoR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:046f75344fc3257767ece0da198715561b969fe7b11e2e3bdb1c75bc4989fb1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  - RAMClustR
  - xcms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# RAMClustR correlation-based pseudo-MS/MS spectrum assembly

## Summary

Assembles a pseudo-MS/MS spectrum for a target LC–MS feature from all-ion-fragmentation (AIF) data by correlating the parent's MS1 extracted-ion chromatogram (EIC) against every AIF-scan EIC with RAMClustR/RSpec, retaining fragment peaks whose Pearson r exceeds a user-set threshold (default 0.8), and falling back to the full AIF spectrum at the feature's retention time when no fragment qualifies.

## When to use

Use when reconstructing an in-source fragment pseudo-MS/MS spectrum from centroided AIF LC–MS chromatograms processed with xcms/RAMClustR — e.g. when annotating isotopologues, adducts, or in-source fragments from AIF datasets where no targeted MS/MS acquisition exists.

## When NOT to use

- The input is already a feature table (m/z, RT, intensity) without raw chromatographic profiles — the correlation method requires EIC peak-shape profiles, not summary intensities
- The data were acquired with a non-AIF DIA scheme (e.g. SWATH) or on instruments where AIF support has not been validated (e.g. Orbitraps)
- A targeted MS/MS spectrum already exists for the feature — direct spectral matching is preferable to reconstructing an in-source pseudo-MS/MS

## Inputs

- Centroided raw AIF LC–MS chromatograms (mzML/NetCDF) or a pre-built xcmsSet (xcms) object plus a RAMClustR (RC) object from MetaboAnnotatoR
- Target feature identifier (m/z, retention time) whose pseudo-MS/MS spectrum is to be reconstructed
- User-set correlation threshold θ (default 0.8, range 0.7–0.8)

## Outputs

- Reconstructed pseudo-MS/MS spectrum of AIF fragment peaks co-eluting and correlating with the target parent
- Exported .rda file pseudo_msms_spectrum.rda for downstream fragment-library matching
- Fallback full AIF spectrum at the target feature's RT when no fragment peak passes the correlation threshold

## How to apply

1. Load the xcmsSet (xcms) and RAMClustR (RC) objects shipped with MetaboAnnotatoR (or generated from your own raw centroid-mode AIF mzML/NetCDF files) into R; the raw chromatograms must have been acquired/transformed in centroid mode. 2. For a chosen target feature, extract the MS1 EIC (e.g. via xcmsSpec) and the high-collision-energy AIF EICs via RCspec to obtain peak-shape profiles across the chromatographic run. 3. Compute the Pearson correlation between the target feature's MS1 EIC and each candidate AIF fragment EIC (RCspec/getPseudoMSMS). 4. Retain only AIF fragment peaks whose correlation exceeds the default threshold θ = 0.8 (typical range 0.7–0.8); peaks below θ are discarded, and if none qualify the full AIF spectrum at the feature's RT is used instead. 5. Export the assembled pseudo-MS/MS spectrum to an .rda file (pseudo_msms_spectrum.rda) for downstream fragment-library matching. 6. Validate by confirming the output matches the spectra produced by the paper's getPseudoMSMS/RCspec/xcmsSpec reference implementation on the example data.

## Related tools

- **RAMClustR** (Provides the RCspec/getPseudoMSMS correlation framework that links AIF fragment peaks to the parent by EIC correlation) — https://github.com/gggraca/MetaboAnnotatoR
- **MetaboAnnotatoR** (R package that wraps the correlation-based pseudo-MS/MS assembly and downstream fragment-library matching; the assembled spectrum is exported to pseudo_msms_spectrum.rda) — https://github.com/gggraca/MetaboAnnotatoR
- **xcms** (Generates the xcmsSet feature table and supplies the MS1 EIC extraction (xcmsSpec) used for the correlation calculation)
- **R** (Runtime environment for running the RAMClustR/MetaboAnnotatoR pipeline (install via install_github)) — https://github.com/gggraca/MetaboAnnotatoR

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE); pseudo <- getPseudoMSMS(xset, RC, feature_id, corr.threshold = 0.8)
```

## Evaluation signals

- Peaks retained in the pseudo-MS/MS spectrum must have Pearson r > θ (default 0.8) against the parent's MS1 EIC
- If no fragment passes θ, the output spectrum must be the full AIF spectrum at the target feature's RT, not an empty set
- The exported pseudo_msms_spectrum.rda must reproduce the spectra generated by getPseudoMSMS/RCspec/xcmsSpec on the paper's example xset/RC objects
- Fragment peaks in the assembled spectrum must co-elute with the parent (same RT region) and be present in the high-collision-energy AIF scans
- Precision/recall of downstream annotation for the three human serum datasets should fall within the reported 82–92% / 82–85% range for high-rank scores (1–5)

## Limitations

- Correlation-based deconvolution can produce incomplete pseudo-MS/MS spectra for co-eluting compounds with similar chemical groups that generate nonspecific fragment ions, or for poorly detected fragment peaks
- The default threshold θ = 0.8 may miss genuine fragments whose peak shapes are distorted; lowering θ risks including fragments from co-eluting compounds
- Peak shape matching / correlation strategies may miss information from co-eluting compounds (e.g. when isomers share fragment ions)
- AIF is the only DIA scheme currently supported; SWATH and other DIA acquisitions have not been validated
- Orbitrap-based (higher-mass-accuracy) AIF validation was not performed in the article; results on such instruments are untested
- Low recall was observed for non-lipid metabolites because the non-lipid fragment library is small

## Evidence

- [results] RAMClustR correlation threshold and fallback: [evidence span withheld — anchor status: not_found]
- [results] Correlation computation workflow: [evidence span withheld — anchor status: not_found]
- [intro] Incomplete pseudo-MS/MS caveat: "These can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds containing similar types of chemical groups, which upon fragmentation produce nonspecific fragment ions"
- [abstract] Correlation-based linking combined with fragment matching: "combining correlation-based parent–fragment linking with molecular fragment matching"
- [readme] Centroid-mode requirement: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [readme] MetaboAnnotatoR install instructions: [evidence span withheld — anchor status: external_doc]
- [abstract] Performance on human serum: "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)"
- [discussion] AIF support limitation: "Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH."
- [discussion] Orbitrap validation not performed: "The annotation of AIF datasets acquired on higher mass accuracy MS instruments (e.g., Orbitraps), although not tested here, is also possible using MetaboAnnotatoR."
