---
name: lipid-fragment-library-matching
description: Use when annotating lipid features from all-ion fragmentation (AIF) LC–MS
  lipidomics datasets (e.g., human serum Lipid+ or Lipid− modes) that require parent–fragment
  matching against curated lipid fragment libraries rather than generic MS/MS spectral
  search.
when_to_use_negative:
- When the dataset is dominated by nonlipid metabolites — recall was low (down to
  10%) on urine/amniotic fluid because the nonlipid fragment library is small.
- When input is a pseudo-MS/MS spectrum from a DIA scheme other than AIF (e.g., SWATH),
  which is not supported.
- When features come from an instrument with higher mass accuracy (e.g., Orbitrap)
  without prior validation — the article notes this case was not tested.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MetaboAnnotatoR
  role: Core R package that implements `annotateAIF` for correlation-based parent–fragment
    linking and lipid fragment library matching on AIF datasets.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: Used to process raw AIF chromatograms into feature tables that are subsequently
    annotated by MetaboAnnotatoR.
- name: xcms
  role: Alternative preprocessing tool whose exported feature lists can be directly
    fed into MetaboAnnotatoR.
- name: RAMClustR
  role: Used in the introductory vignette to reconstruct pseudo-MS/MS spectra from
    AIF data before library matching.
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:ef5a18f38e60067a6ddd12923e072bfd6931957fe25171c84c53dd3099c27ac7
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# lipid-fragment-library-matching

## Summary

Matches metabolite features from AIF LC–MS lipidomics data against curated in-source fragment libraries (LipidPos/LipidNeg) to assign lipid identities, isotopologues, adducts, and in-source fragments via the MetaboAnnotatoR R package.

## When to use

Use when annotating lipid features from all-ion fragmentation (AIF) LC–MS lipidomics datasets (e.g., human serum Lipid+ or Lipid− modes) that require parent–fragment matching against curated lipid fragment libraries rather than generic MS/MS spectral search.

## When NOT to use

- When the dataset is dominated by nonlipid metabolites — recall was low (down to 10%) on urine/amniotic fluid because the nonlipid fragment library is small.
- When input is a pseudo-MS/MS spectrum from a DIA scheme other than AIF (e.g., SWATH), which is not supported.
- When features come from an instrument with higher mass accuracy (e.g., Orbitrap) without prior validation — the article notes this case was not tested.

## Inputs

- LC–MS AIF chromatograms (centroid mode) for lipidomics runs, e.g., Lipid+ and Lipid− acquisitions
- Feature tables exported from MS-DIAL or XCMS/RAMClustR
- LipidPos and LipidNeg fragment libraries supplied with the MetaboAnnotatoR package
- Optional: expert-annotated reference feature list for precision/recall evaluation

## Outputs

- Annotated feature table with assigned lipid names, adducts, isotopologues, and in-source fragments
- Ranked annotation list per feature (score ranks 1–5)
- Precision and recall metrics when compared against a manual annotation list

## How to apply

Run the MetaboAnnotatoR `annotateAIF` function on lipid-mode AIF chromatograms (centroid-mode, processed via MS-DIAL or XCMS) using the LipidPos and LipidNeg fragment libraries supplied with the package. The workflow combines correlation-based parent–fragment linking with molecular fragment matching: fragment peaks that co-elute with a parent ion are linked by peak-shape/correlation, and the resulting pseudo-MS/MS spectrum is searched against the fragment library to assign the lipid, its adduct, isotopologues from the isotopic series, and in-source fragments. Rank candidate annotations by MetaboAnnotatoR score and keep the top ranks (1–5) for validation against an expert-annotated reference list. Precision and recall can then be computed against the 599 manually annotated human serum features.

## Related tools

- **MetaboAnnotatoR** (Core R package that implements `annotateAIF` for correlation-based parent–fragment linking and lipid fragment library matching on AIF datasets.) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (Used to process raw AIF chromatograms into feature tables that are subsequently annotated by MetaboAnnotatoR.)
- **xcms** (Alternative preprocessing tool whose exported feature lists can be directly fed into MetaboAnnotatoR.)
- **RAMClustR** (Used in the introductory vignette to reconstruct pseudo-MS/MS spectra from AIF data before library matching.)

## Examples

```
library(MetaboAnnotatoR); results <- annotateAIF(lipidpos_data, fragment_lib = "LipidPos", msnlib = "LipidNeg")
```

## Evaluation signals

- Precision and recall against the 599 expert-annotated human serum features should fall within the reported 82–92% (precision) and 82–85% (recall) ranges for rank 1–5 annotations.
- On the external validation datasets (adipose tissue ESI+/ESI−, MTBLS666, MTBLS816) precision should be 60–89% and recall 10–88%; low recall on urine/amniotic fluid is expected due to small nonlipid library.
- All annotated isotopologues from isotopic series should be reported, not just the monoisotopic peak.
- Results should equal or outperform MS-DIAL annotations on the same AIF lipidomics data.

## Limitations

- Only AIF is currently supported; other DIA schemes such as SWATH are not implemented.
- The nonlipid fragment library is small, causing low recall (as low as 10%) on urine and amniotic fluid matrices.
- Orbitrap or other higher mass accuracy instruments were not tested in the paper.
- The workflow focuses on a subset of features (e.g., statistically significant ones) rather than annotating the full dataset, which may miss low-intensity features.

## Evidence

- [abstract] Combining correlation-based parent-fragment linking with molecular fragment matching: "combining correlation-based parent–fragment linking with molecular fragment matching"
- [results] annotateAIF function: [evidence span withheld — anchor status: not_found]
- [results] Human serum precision/recall: [evidence span withheld — anchor status: not_found]
- [results] Validation on other matrices: [evidence span withheld — anchor status: not_found]
- [discussion] Isotopologues and in-source fragments: "MetaboAnnotatoR is able to annotate the different isotopologues from isotopic series, as well as in-source fragments."
- [discussion] Direct XCMS compatibility: "This enables the direct application of the workflow to features obtained from XCMS or similar software."
- [discussion] AIF-only support: "Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH."
- [readme] Centroid-mode requirement: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [abstract] MS-DIAL comparison: "These results equal or outperform those obtained using MS-DIAL software, the current state of the art for AIF data annotation"
