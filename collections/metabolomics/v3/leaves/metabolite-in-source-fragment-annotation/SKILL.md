---
name: metabolite-in-source-fragment-annotation
description: Use when processing centroid-mode AIF LC–MS chromatograms (e.g., xcms
  + RAMClustR outputs) and needing to assign isotopologues, adducts, and in-source
  fragments to a statistically pre-selected subset of features rather than the full
  dataset.
when_to_use_negative:
- Input is already a fully annotated feature table (annotation step unnecessary)
- Data are acquired in DIA modes other than AIF, such as SWATH, since MetaboAnnotatoR
  currently only supports AIF
- High-mass-accuracy instruments like Orbitraps require validation the article did
  not perform, so results are untested
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MetaboAnnotatoR
  role: Primary R package for performing metabolite annotation of features from LC-MS
    AIF datasets using ion fragment databases
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: Benchmark comparison tool; MetaboAnnotatoR results equal or outperform MS-DIAL,
    the current state of the art for AIF data annotation
- name: RAMClustR
  role: Preprocessing R package to reconstruct in-source parent-fragment pseudo-MS/MS
    spectra before annotation
- name: xcms
  role: R package for feature detection/alignment whose output can be fed directly
    into MetaboAnnotatoR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:3fe3c4d32a43f233468389a70f50dd1526fec630d8b2a6247330ceffc6447f60
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
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# metabolite-in-source-fragment-annotation

## Summary

Automated annotation of isotopologues, adducts, and in-source fragments from all-ion fragmentation (AIF) LC–MS datasets by combining correlation-based parent–fragment linking with molecular fragment matching using the open-source R package MetaboAnnotatoR.

## When to use

Use when processing centroid-mode AIF LC–MS chromatograms (e.g., xcms + RAMClustR outputs) and needing to assign isotopologues, adducts, and in-source fragments to a statistically pre-selected subset of features rather than the full dataset.

## When NOT to use

- Input is already a fully annotated feature table (annotation step unnecessary)
- Data are acquired in DIA modes other than AIF, such as SWATH, since MetaboAnnotatoR currently only supports AIF
- High-mass-accuracy instruments like Orbitraps require validation the article did not perform, so results are untested

## Inputs

- Raw LC–MS AIF chromatograms acquired or transformed in centroid mode
- Feature tables from xcms (or similar) processing of AIF chromatograms
- Pseudo-MS/MS spectra from RAMClustR (optional reconstruction step)
- Fragment libraries: LipidPos, LipidNeg, MetabolitesPos, MetabolitesNeg (.txt or .msp format)

## Outputs

- Annotated feature table with isotopologues, adducts, and in-source fragment assignments per feature
- Ranked candidate annotations (ranks 1–5) per feature
- Precision and recall metrics when benchmarked against expert-annotated feature lists

## How to apply

First preprocess raw centroid-mode AIF chromatograms with xcms (feature detection/alignment) and optionally RAMClustR (pseudo-MS/MS spectrum reconstruction), as described in the introductory vignette. Select a subset of features (e.g., those reaching statistical significance) because MetaboAnnotatoR focuses on a subset rather than annotating the full dataset, saving time. Run annotateAIF on lipid-mode features with the LipidPos/LipidNeg fragment libraries, and annotateRC on nonlipid metabolite features with MetabolitesPos/MetabolitesNeg libraries, to assign isotopologues, adducts, and in-source fragments. Rank each feature's candidate annotations by score and keep ranks 1–5; compare against expert annotations to compute precision and recall. Default libraries can be customized by importing external fragment libraries as .txt and .msp files.

## Related tools

- **MetaboAnnotatoR** (Primary R package for performing metabolite annotation of features from LC-MS AIF datasets using ion fragment databases) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (Benchmark comparison tool; MetaboAnnotatoR results equal or outperform MS-DIAL, the current state of the art for AIF data annotation)
- **RAMClustR** (Preprocessing R package to reconstruct in-source parent-fragment pseudo-MS/MS spectra before annotation)
- **xcms** (R package for feature detection/alignment whose output can be fed directly into MetaboAnnotatoR)

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE)
```

## Evaluation signals

- Precision of automated annotations versus expert-annotated reference falls within the reported 82–92% range for rank 1–5 annotations on human serum datasets
- Recall versus expert annotations falls within the reported 82–85% range for rank 1–5 annotations on human serum datasets
- Each feature receives a ranked candidate annotation list, with only ranks 1–5 retained for benchmarking
- Isotopologues from isotopic series and in-source fragments are both present in the output annotations, not just parent ions
- Results equal or outperform MS-DIAL on the same AIF data

## Limitations

- Validation for other biological matrices and instrument types showed variable precision (60–89%) and recall (10–88%), particularly for datasets dominated by nonlipid metabolites, due to the small size of the nonlipid fragment library
- Low recall for non-human serum matrices attributable to small nonlipid fragment library size
- Only AIF is currently supported; other DIA schemes such as SWATH are not implemented or validated
- Annotation of AIF datasets acquired on higher mass accuracy MS instruments (e.g., Orbitraps) is possible but untested
- Feature selection prior to annotation is not possible in MS-DIAL, whereas MetaboAnnotatoR requires a pre-selected feature subset

## Evidence

- [abstract] automated annotation of isotopologues, adducts, and in-source fragments from AIF LC–MS datasets: "automated annotation of isotopologues, adducts, and in-source
fragments from AIF LC–MS datasets"
- [abstract] combination of correlation-based parent–fragment linking with molecular fragment matching: "combining correlation-based parent–fragment linking with molecular fragment matching"
- [abstract] The workflow is freely available as an open-source R package, MetaboAnnotatoR: "The workflow is freely available as an open-source
R package, MetaboAnnotatoR"
- [intro] statistical analysis is run prior to feature annotation, focusing on a subset of features: "Since
in most untargeted studies, statistical analysis is run prior to feature
annotation, focusing on a subset of features (e.g., those which reach
statistical significance) might save time and"
- [discussion] MetaboAnnotatoR is able to annotate the different isotopologues from isotopic series, as well as in-source fragments: "MetaboAnnotatoR is able to annotate the different isotopologues from isotopic series, as well as in-source fragments."
- [discussion] features obtained from xcms can be directly applied: "This enables the direct application of the workflow to features obtained from XCMS or similar software."
- [discussion] default libraries can be customized by importing external libraries as .txt and .msp files: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
- [abstract] precision (82–92%) and recall (82–85%) for features found in the highest-rank scores (1–5): "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found
in the highest-rank scores (1–5)"
- [readme] requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [readme] introductory vignette uses xcms and RamClustR: "an example of feature annotation using LC-MS AIF chromatograms processed using xcms and RamClustR packages"
