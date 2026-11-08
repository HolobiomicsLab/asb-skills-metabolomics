---
name: parent-fragment-ion-linking
description: Use when processing centroid-mode all-ion fragmentation (AIF) LC-MS chromatograms
  already peak-picked with xcms/RAMClustR and you need to assemble a pseudo-MS/MS
  spectrum for a specific target feature before fragment library matching.
when_to_use_negative:
- The data are already DIA-acquired with SWATH or other DIA schemes — MetaboAnnotatoR
  currently supports only AIF.
- Features are to be annotated in an untargeted manner across the entire dataset rather
  than a statistically prioritized subset.
- No centroid-mode AIF chromatograms exist; raw data must first be transformed to
  centroid mode.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnnotatoR
  role: R package implementing the pseudo-MS/MS reconstruction via RCspec/getPseudoMSMS
    and the correlation-threshold filtering step
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: xcms
  role: Provides xcmsSpec to extract the MS1 EIC and AIF-scan EICs (peak shape profiles)
    used in the correlation calculation
- name: RAMClustR
  role: Supplies the RC object with peak grouping; the correlation-based linking is
    performed with RCspec on the RC object
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:12a67871f40b245866e9ea1fd5906c081af3684c2f6161e86bfdec94d5454ba1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  - RAMClustR
  - xcms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# parent-fragment-ion-linking

## Summary

Reconstructs a pseudo-MS/MS spectrum for a target AIF feature by computing Pearson correlations between the MS1 EIC and candidate AIF fragment EICs, retaining fragments above a user-set threshold (default 0.8) so that only co-eluting, correlated fragments are linked to the parent ion.

## When to use

Use when processing centroid-mode all-ion fragmentation (AIF) LC-MS chromatograms already peak-picked with xcms/RAMClustR and you need to assemble a pseudo-MS/MS spectrum for a specific target feature before fragment library matching.

## When NOT to use

- The data are already DIA-acquired with SWATH or other DIA schemes — MetaboAnnotatoR currently supports only AIF.
- Features are to be annotated in an untargeted manner across the entire dataset rather than a statistically prioritized subset.
- No centroid-mode AIF chromatograms exist; raw data must first be transformed to centroid mode.

## Inputs

- xcmsSet (xset) object from centroid-mode AIF LC-MS chromatograms
- RAMClustR (RC) object from the same peak-picking run
- Target feature identifier and its retention time
- Correlation threshold θ (default 0.8, typical 0.7–0.8)

## Outputs

- pseudo-MS/MS spectrum (pseudo_msms_spectrum.rda) containing only AIF fragments correlated with the parent ion
- Fallback full AIF spectrum at the feature RT when no fragment exceeds the correlation threshold

## How to apply

Load the xcmsSet and RAMClustR (RC) objects produced by the upstream peak-picking step, then for a chosen target feature extract its MS1 EIC and every high-collision-energy AIF-scan EIC via xcmsSpec. Compute the Pearson correlation between the parent MS1 EIC and each candidate AIF fragment EIC using RCspec/getPseudoMSMS; only fragment peaks whose correlation exceeds the user-defined threshold θ (default 0.8, typically 0.7–0.8) are assembled into the pseudo-MS/MS spectrum. If no AIF peak passes the threshold, fall back to using the full AIF spectrum at the feature's retention time. The rationale is that peak-shape correlation deconvolution separates co-eluting parents and avoids including nonspecific fragments from compounds containing similar chemical groups. Finally, export the reconstructed spectrum to pseudo_msms_spectrum.rda so it can be fed into the downstream fragment-library matching stage of MetaboAnnotatoR.

## Related tools

- **MetaboAnnotatoR** (R package implementing the pseudo-MS/MS reconstruction via RCspec/getPseudoMSMS and the correlation-threshold filtering step) — https://github.com/gggraca/MetaboAnnotatoR
- **xcms** (Provides xcmsSpec to extract the MS1 EIC and AIF-scan EICs (peak shape profiles) used in the correlation calculation)
- **RAMClustR** (Supplies the RC object with peak grouping; the correlation-based linking is performed with RCspec on the RC object)

## Examples

```
library(MetaboAnnotatoR); pseudo <- getPseudoMSMS(xset, RC, feature_id, corr.threshold = 0.8)
```

## Evaluation signals

- The exported pseudo_msms_spectrum.rda reproduces the spectra produced by the paper's reference implementation of getPseudoMSMS/RCspec/xcmsSpec on the same example xset and RC objects.
- Every retained fragment peak has a Pearson correlation with the parent MS1 EIC above the configured threshold θ (e.g. > 0.8 when the default is used).
- When no AIF peak passes the threshold, the output spectrum equals the full AIF spectrum at the feature RT, confirming the fallback path was taken.
- The resulting pseudo-MS/MS spectrum, when fed into the downstream fragment-library matching, yields precision and recall in the 82–92% and 82–85% range reported for the highest-rank (1–5) scores on the human serum datasets.

## Limitations

- Peak shape correlation deconvolution can produce incomplete pseudo-MS/MS spectra for co-eluting compounds with similar chemical groups that produce nonspecific fragment ions, or for poorly detected fragment peaks.
- Only AIF acquisition is supported; other DIA schemes such as SWATH are not currently implemented or validated.
- Validation was performed on QTOF data; Orbitrap-based (higher mass accuracy) experiments are possible but were not tested.
- The nonlipid fragment library is small, which limits recall for datasets dominated by nonlipid metabolites.

## Evidence

- [full_text] Pseudo-MS/MS generation computes Pearson correlations between the target feature's MS1 EIC and each AIF-scan EIC; AIF peaks with r above a user-set threshold θ (typically 0.7–0.8, default 0.8) are gathered into the pseudo-MS/MS spectrum, and if none qualify, the full AIF spectrum at the feature RT is used instead.: [evidence span withheld — anchor status: not_found]
- [abstract] combining correlation-based parent–fragment linking with molecular fragment matching: "combining correlation-based parent–fragment linking with molecular fragment matching"
- [readme] This R package is designed to perform metabolite annotation of features from LC-MS All-ion fragmentation (AIF) datasets, using ion fragment databases. It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode.: "This R package is designed to perform metabolite annotation of features from LC-MS All-ion fragmentation (AIF) datasets, using ion fragment databases.
It requires raw LC-MS AIF chromatograms"
- [intro] These can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds containing similar types of chemical groups, which upon fragmentation produce nonspecific fragment ions: "These can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds containing similar types of chemical groups, which upon fragmentation produce nonspecific fragment ions"
- [results] export the reconstructed pseudo-MS/MS spectrum to pseudo_msms_spectrum.rda: [evidence span withheld — anchor status: not_found]
