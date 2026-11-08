---
name: isotopologue-adduct-annotation
description: Use when processing centroid-mode AIF LC–MS metabolomics chromatograms
  and you need to assign isotopologues, adducts, and in-source fragments across features,
  especially when focusing on a statistically significant subset rather than the full
  dataset.
when_to_use_negative:
- Input is DIA data other than AIF (e.g., SWATH) — MetaboAnnotatoR currently supports
  only AIF.
- Data were acquired on high-resolution instruments without centroid conversion —
  raw profile-mode chromatograms are not supported directly.
- You need full-dataset untargeted annotation with no statistical pre-selection —
  MetaboAnnotatoR is designed for a subset of features, not exhaustive annotation.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MetaboAnnotatoR
  role: Primary R package performing correlation-based parent–fragment linking and
    fragment matching to annotate isotopologues, adducts, and in-source fragments
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: xcms
  role: Used upstream to process raw AIF chromatograms into feature tables that MetaboAnnotatoR
    ingests directly
- name: RAMClustR
  role: Reconstructs in-source parent-fragment pseudo-MS/MS spectra prior to MetaboAnnotatoR
    annotation
- name: MS-DIAL
  role: State-of-the-art comparator for AIF annotation performance; used in benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:6206e804e148ec1ec063ce6ee9022efc6f4cf32afebdf45f547a7ae573bf10ec
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# isotopologue-adduct-annotation

## Summary

Annotates isotopologues, adducts, and in-source fragments in untargeted all-ion fragmentation (AIF) LC–MS datasets using MetaboAnnotatoR, a correlation-based parent–fragment linking approach combined with molecular fragment matching. Enables automated, rank-scored annotation of AIF features to replace manual MS spectral inspection.

## When to use

Use when processing centroid-mode AIF LC–MS metabolomics chromatograms and you need to assign isotopologues, adducts, and in-source fragments across features, especially when focusing on a statistically significant subset rather than the full dataset.

## When NOT to use

- Input is DIA data other than AIF (e.g., SWATH) — MetaboAnnotatoR currently supports only AIF.
- Data were acquired on high-resolution instruments without centroid conversion — raw profile-mode chromatograms are not supported directly.
- You need full-dataset untargeted annotation with no statistical pre-selection — MetaboAnnotatoR is designed for a subset of features, not exhaustive annotation.

## Inputs

- Centroid-mode AIF LC–MS chromatograms (e.g., lipid-mode and nonlipid-mode acquisitions)
- Feature tables from xcms (RT, m/z, intensity)
- Pseudo-MS/MS spectra from RAMClustR or equivalent deconvolution
- External fragment libraries in .txt or .msp format (optional, to customize defaults)

## Outputs

- Ranked annotation table per feature (isotopologues, adducts, in-source fragments with scores)
- Subset of annotated features suitable for downstream statistical/biological interpretation
- Precision and recall metrics when compared against expert annotations

## How to apply

Acquire or convert raw AIF chromatograms to centroid mode, process with xcms and RAMClustR to obtain feature RT/intensity tables and pseudo-MS/MS spectra, then feed these to MetaboAnnotatoR's annotateAIF (lipid mode, using LipidPos/LipidNeg libraries) and annotateRC (nonlipid mode, using MetabolitesPos/MetabolitesNeg libraries). The workflow combines correlation-based parent–fragment linking with molecular fragment matching to assign isotopologues, adducts, and in-source fragments; run statistical analysis first so annotation targets only a subset of features (e.g., those reaching significance) to save time. Combine lipid and nonlipid outputs and rank candidate annotations by score, keeping ranks 1–5 for downstream validation. Validate by comparing automated annotations to expert-manually annotated features, computing precision and recall.

## Related tools

- **MetaboAnnotatoR** (Primary R package performing correlation-based parent–fragment linking and fragment matching to annotate isotopologues, adducts, and in-source fragments) — https://github.com/gggraca/MetaboAnnotatoR
- **xcms** (Used upstream to process raw AIF chromatograms into feature tables that MetaboAnnotatoR ingests directly)
- **RAMClustR** (Reconstructs in-source parent-fragment pseudo-MS/MS spectra prior to MetaboAnnotatoR annotation)
- **MS-DIAL** (State-of-the-art comparator for AIF annotation performance; used in benchmarking)

## Examples

```
library(devtools); install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE)
```

## Evaluation signals

- Precision should fall within 82–92% and recall within 82–85% on human serum datasets for rank 1–5 annotations.
- Annotated features should include different isotopologues from isotopic series as well as in-source fragments.
- Peak-list format must be compatible with xcms/RAMClustR outputs (direct feature import).
- Custom fragment libraries should be imported as .txt or .msp files and produce consistent annotation outputs.
- Manual comparison against expert-annotated features (e.g., 599-feature reference list) should show no systematic mismatch in rank 1–5 assignments.

## Limitations

- AIF only — other DIA schemes (e.g., SWATH) are not currently supported.
- Higher-mass-accuracy instruments (e.g., Orbitraps) are possible but were not tested in the publication.
- Performance varies for non-human serum matrices, especially when dominated by nonlipid metabolites, due to the small nonlipid fragment library size.
- Installation may fail due to mzR/Rcpp version mismatch; workaround documented in README.

## Evidence

- [abstract] Core function description: "automated annotation of isotopologues, adducts, and in-source
fragments from AIF LC–MS datasets"
- [abstract] Workflow rationale: "combining correlation-based
parent–fragment linking with molecular fragment matching"
- [abstract] Performance metric: "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found
in the highest-rank scores (1–5)"
- [intro] Upstream processing: "Two R packages, CAMERA10 and RAMClustR,5 can also be used to reconstruct in-source parent-fragment
pseudo-MS/MS spectra"
- [readme] Install command: [evidence span withheld — anchor status: external_doc]
- [discussion] Fragment library customization: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
- [discussion] Isotopologue annotation: "MetaboAnnotatoR is able to annotate the different isotopologues from isotopic series, as well as in-source fragments."
