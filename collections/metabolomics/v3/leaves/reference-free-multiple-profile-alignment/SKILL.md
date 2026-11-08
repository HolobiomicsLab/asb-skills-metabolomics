---
name: reference-free-multiple-profile-alignment
description: Use when XCMS peak grouping has flagged feature groups as misaligned
  (features whose RT drift differs across m/z bins or with run-order-dependent drift)
  and the per-sample corrected RTs must be replaced with compound-specific warping
  functions before re-grouping or peak-filling.
when_to_use_negative:
- Features are already well-aligned by XCMS's single per-sample warping function and
  were not flagged by ncGTW's misalignment detection module
- You need a global whole-dataset re-alignment rather than targeted realignment of
  specific detected feature groups — ncGTW is a targeted post-XCMS plug-in, not a
  full preprocessing pipeline
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ncGTW
  role: Performs the two-stage reference-free realignment (loadProfile, ncGTWalign,
    adjustRT) on misaligned feature profiles
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Upstream peak detection/alignment whose corrected RTs are replaced by ncGTW
    warping functions; downstream grouping and peak-filling consume the realigned
    data
- name: R
  role: Runtime environment; ncGTW is an R/Bioconductor package installed via BiocManager
    or devtools
  repo: https://github.com/ChiungTingWu/ncGTW
- name: Bioconductor
  role: Distribution channel for the ncGTW R package
  repo: https://github.com/ChiungTingWu/ncGTW
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:cafb45447f302a83723aa8df41b60ff605ba02aa127f41dd8c13d19334239e8a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - Bioconductor
  - R
  - XCMS
  - ncGTW
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# reference-free-multiple-profile-alignment

## Summary

Realign misaligned LC-MS feature profiles using ncGTW's two-stage, reference-free algorithm: combinatorial pairwise warpings between all sample pairs (with neighboring-sample structural constraints solved as maximum-flow problems) are coordinately aggregated onto a pseudo-reference to yield sample-specific RT warping functions.

## When to use

Use when XCMS peak grouping has flagged feature groups as misaligned (features whose RT drift differs across m/z bins or with run-order-dependent drift) and the per-sample corrected RTs must be replaced with compound-specific warping functions before re-grouping or peak-filling.

## When NOT to use

- Features are already well-aligned by XCMS's single per-sample warping function and were not flagged by ncGTW's misalignment detection module
- You need a global whole-dataset re-alignment rather than targeted realignment of specific detected feature groups — ncGTW is a targeted post-XCMS plug-in, not a full preprocessing pipeline

## Inputs

- XCMS object (e.g., xcmsLargeWin) with detected feature groups and corrected retention times
- Raw LC-MS profile data (extracted-ion profiles) for feature groups flagged as misaligned by ncGTW's detection step
- ncGTW misalignment detection output identifying candidate misaligned feature groups

## Outputs

- Sample-specific RT warping functions for each realigned feature (from adjustRT/rtncGTW)
- XCMS object with corrected retention times replaced for realigned features (xcmsLargeWin@rt$corrected)
- Stored realigned features and warping functions (e.g., ncGTW_aligned_features.rda) for downstream XCMS grouping and peak-filling

## How to apply

First load the raw extracted-ion profiles for each flagged feature group with ncGTW's loadProfile(), applied to the XCMS object and the raw files. Run ncGTWalign() on each flagged feature's profiles, using parSamp=5 to split features into sub-groups; Stage 1 performs all pairwise alignments between samples with neighboring-sample structural constraints, and Stage 2 estimates warping functions for all samples to a common coordinate using the pairwise alignments as constraints, avoiding a fixed reference. Convert the alignment into sample-specific RT warping functions via adjustRT()/rtncGTW and use them to replace xcmsLargeWin@rt$corrected for the realigned features. Store the realigned features and new warping functions (e.g., ncGTW_aligned_features.rda) so XCMS can re-run grouping and peak-filling on the corrected data.

## Related tools

- **ncGTW** (Performs the two-stage reference-free realignment (loadProfile, ncGTWalign, adjustRT) on misaligned feature profiles) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Upstream peak detection/alignment whose corrected RTs are replaced by ncGTW warping functions; downstream grouping and peak-filling consume the realigned data)
- **R** (Runtime environment; ncGTW is an R/Bioconductor package installed via BiocManager or devtools) — https://github.com/ChiungTingWu/ncGTW
- **Bioconductor** (Distribution channel for the ncGTW R package) — https://github.com/ChiungTingWu/ncGTW

## Examples

```
devtools::install_github("ChiungTingWu/ncGTW"); profiles <- loadProfile(xcmsLargeWin, ncGTWres); ncGTWres <- ncGTWalign(profiles, xcmsLargeWin, parSamp=5); newRT <- adjustRT(profiles, xcmsLargeWin, ncGTWres)
```

## Evaluation signals

- Average pairwise correlation coefficient of realigned profiles is higher than the initial XCMS alignment on true-positive misaligned features
- Average pairwise total overlapping area of realigned profiles is higher than the initial XCMS alignment
- Post-alignment screening with these two scores successfully separates true positives from detection-stage false positives (e.g., the 57 features detected in the article's analysis)
- Realigned features and warping functions are stored and XCMS peak-filling retrieves previously missing peaks on the corrected data
- No severe peak shape distortion introduced by the warping functions (distortion correctable by ncGTW's post-processing module)

## Limitations

- ncGTW can introduce peak shape distortion; parameter settings may need adjustment to reduce the likelihood of distortion, and a post-processing module corrects distortion
- The misalignment detection step yields a false positive rate higher than the theoretical 0.05 threshold in some datasets (e.g., Rotterdam), so detected candidates must be screened post-alignment
- ncGTW has higher computational cost; accepting false positives adds burden, though screening them out mitigates this
- Exhaustive parameter settings and version numbers for ncGTW/XCMS are not specified in the article; consult the package defaults

## Evidence

- [methods] Peak shape distortion is then efficiently corrected by the post-processing module of ncGTW package.: [evidence span withheld — anchor status: not_found]
