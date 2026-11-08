---
name: aif-parent-fragment-deconvolution
description: Use when processing centroid-mode AIF LC–MS chromatograms whose parent–fragment
  relationships must be deconvoluted into pseudo-MS/MS spectra for metabolite annotation
  of isotopologues, adducts, and in-source fragments.
when_to_use_negative:
- Data were acquired with DIA schemes other than AIF (e.g. SWATH), which the package
  currently does not support.
- Input is already a deconvoluted pseudo-MS/MS spectrum or a feature list from another
  tool with the annotation step already complete.
- The sample matrix is dominated by nonlipid metabolites (e.g. urine MTBLS816 or amniotic
  fluid MTBLS666) and the nonlipid fragment library is small, as recall can drop to
  10–88%.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: R package that performs correlation-based parent–fragment linking and molecular
    fragment matching on AIF LC–MS data to produce pseudo-MS/MS spectra and ranked
    annotations.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment for installing and running MetaboAnnotatoR via devtools
    install_github; dependencies include mzR and Rcpp.
- name: MS-DIAL
  role: Benchmark deconvolution tool used for comparison; unlike MetaboAnnotatoR it
    determines parent–fragment relationships solely by co-elution inspection and performs
    untargeted annotation of all detected features.
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:39ff92f103dc9873f6159b13ac6599b283251bdc18e1bc844fa9eb1da16e6ad5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# aif-parent-fragment-deconvolution

## Summary

Reconstructs in-source parent–fragment ion relationships (pseudo-MS/MS spectra) from centroid-mode all-ion fragmentation LC–MS data by combining peak-shape correlation deconvolution with molecular fragment-library matching, using the MetaboAnnotatoR R package to annotate isotopologues, adducts, and in-source fragments.

## When to use

Use when processing centroid-mode AIF LC–MS chromatograms whose parent–fragment relationships must be deconvoluted into pseudo-MS/MS spectra for metabolite annotation of isotopologues, adducts, and in-source fragments.

## When NOT to use

- Data were acquired with DIA schemes other than AIF (e.g. SWATH), which the package currently does not support.
- Input is already a deconvoluted pseudo-MS/MS spectrum or a feature list from another tool with the annotation step already complete.
- The sample matrix is dominated by nonlipid metabolites (e.g. urine MTBLS816 or amniotic fluid MTBLS666) and the nonlipid fragment library is small, as recall can drop to 10–88%.

## Inputs

- centroid-mode AIF LC–MS chromatograms (mzML/mzXML readable by mzR)
- MetaboAnnotatoR fragment libraries (LipidPos, LipidNeg, MetabolitesPos, MetabolitesNeg)
- feature table from peak picking (e.g. xcms/RamClustR processed data)

## Outputs

- pseudo-MS/MS spectra linking parent ions to their fragments
- ranked annotation table assigning isotopologues, adducts, and in-source fragments per feature
- precision/recall metrics versus expert-manual annotation

## How to apply

Install the MetaboAnnotatoR package from GitHub and load its fragment libraries (LipidPos/LipidNeg for lipids, MetabolitesPos/MetabolitesNeg for nonlipid metabolites). Run the correlation-based parent–fragment linking, which pairs peak-shape matching (RT at maximum intensity and height at half-maximum) with peak-shape correlation to reconstruct pseudo-MS/MS spectra from AIF scans; this differs from tools like MS-DIAL, LipidMatch, or R-MetaboList that rely solely on co-elution inspection. Combine the resulting pseudo-MS/MS spectra with molecular fragment matching against the fragment libraries to assign candidate annotations, then rank them by score and keep the highest-rank scores (1–5). Because AIF datasets can contain tens of thousands of features, focus annotation on a statistically or biologically prioritized subset of features to save time, since in-source fragments and isotopologues arise directly from the MS spectrum during ionization. Validate by computing precision and recall against expert-annotated features; on the three human serum datasets with 599 manually annotated features, expect 82–92% precision and 82–85% recall for rank 1–5 annotations.

## Related tools

- **MetaboAnnotatoR** (R package that performs correlation-based parent–fragment linking and molecular fragment matching on AIF LC–MS data to produce pseudo-MS/MS spectra and ranked annotations.) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment for installing and running MetaboAnnotatoR via devtools install_github; dependencies include mzR and Rcpp.)
- **MS-DIAL** (Benchmark deconvolution tool used for comparison; unlike MetaboAnnotatoR it determines parent–fragment relationships solely by co-elution inspection and performs untargeted annotation of all detected features.)

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE)
```

## Evaluation signals

- Precision of 82–92% and recall of 82–85% for rank 1–5 annotations on the three human serum datasets containing 599 expert-annotated features.
- Pseudo-MS/MS spectra must correctly link in-source fragments and isotopologues to their parent ions, not only adducts.
- Results should equal or outperform MS-DIAL, the current state of the art for AIF data annotation.
- On external validation matrices (adipose tissue ESI+/ESI−, MTBLS666 amniotic fluid, MTBLS816 urine), expect precision 60–89% and recall 10–88%.
- Per-dataset precision/recall values should fall within the ranges reported in Table 2.

## Limitations

- Low recall on non-human serum matrices attributable to the small size of the nonlipid fragment library.
- Higher mass accuracy MS instruments (e.g. Orbitraps) were not tested but are stated as possible.
- Only AIF DIA is supported; other DIA schemes such as SWATH are future work.
- Peak shape matching and correlation deconvolution can produce incomplete pseudo-MS/MS spectra for co-eluting compounds with similar chemical groups producing nonspecific fragments, or for poorly detected fragment peaks.
- Annotation requires raw chromatograms acquired or transformed in centroid mode.
- Installation may fail with 'mzR has been built against a different Rcpp version'; fix with Sys.setenv(R_REMOTES_NO_ERRORS_FROM_WARNINGS="true") and INSTALL_opts c('--no-multiarch','--no-lock').

## Evidence

- [intro] Parent-fragment deconvolution rationale: "to get meaningful information from AIF datasets, special processing routines are needed to reconstruct the parent–fragment ion relationships and obtain a spectrum containing all fragments arising"
- [intro] Correlation + peak-shape approach: "These can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds containing similar types of chemical groups, which upon fragmentation produce nonspecific fragment ions"
- [abstract] Workflow combination: "combining correlation-based parent–fragment linking with molecular fragment matching"
- [abstract] Performance benchmark: "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)"
- [readme] Centroid-mode requirement: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [abstract] Fragment library scope: "Further validation for other biological matrices and different instrument types showed variable precision (60–89%) and recall (10–88%)"
- [intro] Subset-focus rationale: "focusing on a subset of features (e.g., those which reach statistical significance) might save time and simplify the process"
