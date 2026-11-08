---
name: aif-pseudo-msms-deconvolution
description: Use when processing centroid-mode AIF LC–MS/MS chromatograms where no
  targeted MS/MS scans exist, and the goal is to recover parent–fragment relationships
  and annotate adducts, isotopologues, and in-source fragments.
when_to_use_negative:
- When targeted MS/MS scans already exist for the features of interest — those spectra
  are higher quality than deconvoluted pseudo-MS/MS.
- When the dataset was acquired on a non-AIF DIA scheme (e.g. SWATH), which is not
  yet supported by MetaboAnnotatoR.
- When the user already has an expert-curated feature table with manual annotations
  — deconvolution would add redundant processing rather than new information.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnnotatoR
  role: R package that implements the correlation-based parent–fragment linking and
    fragment-library matching for pseudo-MS/MS annotation
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: Baseline AIF deconvolution workflow (MS2Dec) used for comparison against MetaboAnnotatoR's
    annotation results
- name: LipidBlast
  role: In-silico lipid spectral library searched with the deconvoluted pseudo-MS/MS
    spectra
- name: METLIN
  role: Public MS/MS spectral database used for comparison in the annotation evaluation
- name: MassBank
  role: Public MS/MS spectral database used for comparison in the annotation evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:4b150c3406079ba87acaa10aa62dcb2d0fd3b23aac1dc87a1730e57f69589243
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - GNPS
  - LipidBlast
  - METLIN
  - MS-DIAL
  - MassBank
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# aif-pseudo-msms-deconvolution

## Summary

Reconstruct pseudo-MS/MS spectra from all-ion fragmentation (AIF) LC–MS runs by linking co-eluting fragment peaks to their parent ions, then match those spectra against in-silico fragment libraries (e.g. LipidBlast, MetaboAnnotatoR fragment libraries) to annotate metabolite features.

## When to use

Use when processing centroid-mode AIF LC–MS/MS chromatograms where no targeted MS/MS scans exist, and the goal is to recover parent–fragment relationships and annotate adducts, isotopologues, and in-source fragments.

## When NOT to use

- When targeted MS/MS scans already exist for the features of interest — those spectra are higher quality than deconvoluted pseudo-MS/MS.
- When the dataset was acquired on a non-AIF DIA scheme (e.g. SWATH), which is not yet supported by MetaboAnnotatoR.
- When the user already has an expert-curated feature table with manual annotations — deconvolution would add redundant processing rather than new information.

## Inputs

- Raw centroid-mode AIF LC–MS/MS chromatograms (e.g. from xcms or RAMClustR processing)
- Feature tables containing m/z, retention time, and peak-shape information for detected features
- Statistically significant subset of features from the untargeted study

## Outputs

- Pseudo-MS/MS spectra linking parent ions to their co-eluting fragment peaks
- Ranked annotation lists assigning adducts, isotopologues, and in-source fragments to features
- Precision/recall statistics for features at high-rank scores (1–5)

## How to apply

First, pre-process raw centroid-mode AIF chromatograms (e.g. with xcms + RAMClustR) to obtain a feature table with retention times, m/z, and peak shapes. Then, within MetaboAnnotatoR, correlate the chromatographic peak shapes of co-eluting fragment peaks against parent ions to build pseudo-MS/MS spectra, as done in MS-DIAL's MS2Dec deconvolution. Match these pseudo-MS/MS spectra against in-silico fragment libraries (LipidBlast for lipids, custom .txt/.msp libraries for non-lipids) using the annotation scoring in MetaboAnnotatoR. Rationale: deconvolution is needed because AIF acquisition fragments all ions simultaneously, and peak shape matching/correlation recovers parent–fragment links when RTs coincide or peaks correlate; focusing on a statistically significant subset of features (e.g. those significant in the untargeted study) saves time and avoids annotating the entire dataset.

## Related tools

- **MetaboAnnotatoR** (R package that implements the correlation-based parent–fragment linking and fragment-library matching for pseudo-MS/MS annotation) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (Baseline AIF deconvolution workflow (MS2Dec) used for comparison against MetaboAnnotatoR's annotation results)
- **LipidBlast** (In-silico lipid spectral library searched with the deconvoluted pseudo-MS/MS spectra)
- **METLIN** (Public MS/MS spectral database used for comparison in the annotation evaluation)
- **MassBank** (Public MS/MS spectral database used for comparison in the annotation evaluation)

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE); results <- annotateAIF(features, chromatograms, libraries = list(lipid = LipidBlast))  # per the README vignette workflow
```

## Evaluation signals

- Manual-vs-MetaboAnnotatoR rank-1 overlap should match published figures (e.g. 121/161 for Lipid+, 37/50 for HILIC+), and should meet or exceed the MS-DIAL baseline (89/161, 31/50).
- Precision and recall for features at rank scores 1–5 should fall in the reported ranges of 82–92% and 82–85% for human serum datasets.
- The number of annotated features should be smaller than the total detected features, reflecting the focus on a statistically significant subset rather than the full dataset.
- Isotopologues and in-source fragments from isotopic series should be detected in the annotation output, as expected from the workflow design.
- If a feature is absent from the pseudo-MS/MS spectrum but present in the raw AIF scan, the workflow should still recover it via the AIF search fallback.

## Limitations

- Deconvolution strategies (peak shape matching / correlation) can miss co-eluting compounds with similar chemical groups producing nonspecific fragment ions, or poorly detected fragment peaks.
- Recall is low for datasets dominated by non-lipid metabolites (10–88%) due to the small size of the non-lipid fragment library.
- Only AIF acquisition is supported; other DIA schemes such as SWATH are not implemented.
- Higher-mass-accuracy instruments (e.g. Orbitraps) were not validated in the paper, though the authors note the workflow should work in principle.

## Evidence

- [abstract] combining correlation-based parent–fragment linking with molecular fragment matching: "combining correlation-based parent–fragment linking with molecular fragment matching"
- [intro] AIF acquisition fragments all ions: "All-ion fragmentation (AIF) LC–MS/MS acquisition provides fragmentation data at no additional experimental time cost"
- [intro] deconvolution reconstructs pseudo-MS/MS: "Several commercial and open-source/free software packages, such as MS-DIAL,7 DIA-Umpire,8 RAMClustR,5 and R-MetaboList,9 employ deconvolution techniques to link AIF parent ions to their corresponding"
- [intro] peak shape matching improves annotation: "Nonetheless, peak shape matching and peak shape correlation approaches when used together with fragment library matching could improve the annotation of AIF LC–MS features"
- [intro] focus on significant subset: "Since in most untargeted studies, statistical analysis is run prior to feature annotation, focusing on a subset of features (e.g., those which reach statistical significance) might save time and"
- [intro] co-eluting compounds produce incomplete spectra: "These can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds containing similar types of chemical groups, which upon fragmentation produce nonspecific fragment ions"
- [readme] README install + xcms/RamClustR pipeline: [evidence span withheld — anchor status: external_doc]
- [discussion] annotation of isotopologues and in-source fragments: "MetaboAnnotatoR is able to annotate the different isotopologues from isotopic series, as well as in-source fragments."
