---
name: accuracy-rate-computation-annotation-vs-reference
description: Use when a new or revised annotation algorithm (e.
when_to_use_negative:
- When the samples contain only unknowns and no reference compound list with known
  sum formulas/molar masses exists — the comparison against a ground truth is then
  impossible.
- When the analysis goal is feature-grouping performance (e.g. how many pseudo-spectra
  are formed) rather than correctness of the proposed accurate molecular mass.
- When the reference compounds are absent from the analyzed matrix (e.g. not spiked
  or degraded), since a failed detection would be misinterpreted as an annotation
  error.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Produces the annotation (compound spectra, isotope/adduct labeling and proposed
    accurate masses) that is benchmarked against the reference annotation.
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Performs feature detection and peak picking to generate the peak lists that
    serve as input to the annotation being evaluated.
- name: getPeaklist
  role: Extracts the annotated peak list from the xsAnnotate object so the proposed
    accurate masses can be compared against the reference annotations.
- name: R
  role: Environment in which the comparison script and the accuracy-rate computation
    are carried out (CAMERA and XCMS are R packages).
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:b5fc64b8586287d499c7fa8ffe14195521887acecd490766063269d76b127323
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  - findAdducts
  - findIsotopes
  - getPeaklist
  - groupCorr
  - groupFWHM
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# Accuracy-rate computation for annotation vs. reference

## Summary

A method for validating computational annotation tools by comparing tool-derived molecular masses against manually defined reference annotations on compounds of known identity spiked into a complex matrix, then computing the percentage of correctly extracted accurate masses per ion mode (e.g. CAMERA achieved 89.7% in positive and 90.3% in negative mode on the MM39 recovery dataset).

## When to use

Use when a new or revised annotation algorithm (e.g. adduct/isotope labeling, compound-spectrum grouping) must be benchmarked for how successfully it extracts correct molecular masses, and a list of known reference compounds with sum formulas, molar masses and PubChem IDs spiked into a matrix is available in both ionization modes.

## When NOT to use

- When the samples contain only unknowns and no reference compound list with known sum formulas/molar masses exists — the comparison against a ground truth is then impossible.
- When the analysis goal is feature-grouping performance (e.g. how many pseudo-spectra are formed) rather than correctness of the proposed accurate molecular mass.
- When the reference compounds are absent from the analyzed matrix (e.g. not spiked or degraded), since a failed detection would be misinterpreted as an annotation error.

## Inputs

- LC/MS peak lists (from XCMS feature detection) for spiked recovery samples in positive and negative ion modes at multiple concentrations (e.g. 20, 5, 1, 0.2 µM) plus a pure-solution control
- Manually defined reference compound list with sum formulas, molar masses and PubChem IDs (e.g. the MM39 set of 39 known compounds)
- Annotation output table (e.g. from CAMERA's getPeaklist) containing proposed accurate compound masses per pseudo-spectrum

## Outputs

- Accuracy-rate metric per ion mode (e.g. 89.7% positive, 90.3% negative) and per concentration
- Annotated peak list table with the tool's proposed mass and the matching reference mass for each annotatable compound
- Comparison summary of correctly vs incorrectly extracted molecular masses (saved as a results annotation)

## How to apply

First, prepare a recovery experiment: a mixture of known compounds (e.g. the MM39 set of 39 compounds spanning m/z 161–822 Da) is dissolved both in pure solution and spiked into a complex matrix (e.g. Arabidopsis leaf extract) at multiple concentrations (20, 5, 1 and 0.2 µM) in positive and negative ionization modes. Run the peak-detection tool (XCMS) on each dataset, then apply the annotation algorithm under test (e.g. CAMERA's groupFWHM → findIsotopes → groupCorr → findAdducts) to generate compound spectra and proposed accurate masses. Next, load the manually defined reference annotation containing the true sum formulas, molar masses and PubChem IDs of the spiked compounds. For each annotatable compound, compare the tool-derived molecular mass against the reference molar mass; count a hit when the proposed accurate mass matches the reference within the tool's mass-accuracy tolerance. Finally, compute the accuracy rate as the number of correctly extracted masses divided by the number of annotatable compounds, reported separately for each ion mode and concentration, and save the annotation table with the comparison summary as a results annotation. Rationale: this measures the annotation method's ability to recover the correct quasi-molecular ion rather than merely the number of grouped features.

## Related tools

- **CAMERA** (Produces the annotation (compound spectra, isotope/adduct labeling and proposed accurate masses) that is benchmarked against the reference annotation.) — https://github.com/sneumann/CAMERA
- **XCMS** (Performs feature detection and peak picking to generate the peak lists that serve as input to the annotation being evaluated.)
- **getPeaklist** (Extracts the annotated peak list from the xsAnnotate object so the proposed accurate masses can be compared against the reference annotations.)
- **R** (Environment in which the comparison script and the accuracy-rate computation are carried out (CAMERA and XCMS are R packages).)

## Examples

```
Rscript annotate_and_score.R --peaklist=xcms_peaks.RData --reference=mm39_reference.csv --mode=positive --tolerance_ppm=5
```

## Evaluation signals

- Accuracy-rate value should approach the expected benchmark (e.g. ~90% in both ion modes) when the same MM39 recovery dataset and CAMERA parameters are used.
- Every correctly annotated compound's proposed mass must match a reference molar mass within the tool's mass-accuracy tolerance; mismatches should be explainable by the documented failure modes (too few explained features, tied precedence scores, or a compound spectrum containing only molecular ion and fragment ions with no further adducts).
- The comparison summary must state the number of annotatable compounds and the denominator used, since compounds without enough features in the pseudo-spectrum are excluded from the accuracy calculation.
- Results should be reported separately for positive and negative ion modes, because CAMERA's paper reports distinct accuracy rates for each mode.
- The annotated peak list must include the PubChem ID and sum formula from the reference list for each correctly annotated compound, allowing a manual spot-check of the mapping.

## Limitations

- Accuracy is defined only over 'annotatable compounds'; compounds whose pseudo-spectra contain too few explained features or only a molecular ion plus fragment ions (with no further adducts) cannot be annotated and are excluded, potentially inflating the reported rate.
- When several hypotheses have identical precedence scores, the correct mass may be missed even though a plausible spectrum exists.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- Sample concentration must avoid MS detector saturation, which is known to cause shifts of m/z and retention time centroids and truncated peak profiles, biasing the annotation.
- The reference accuracy is tied to the specific recovery experiment (MM39 spiked into Arabidopsis extract); it does not generalise to datasets without known reference compounds.

## Evidence

- [abstract] Accuracy benchmark on MM39 recovery data: "CAMERA successfully extracted accurate masses for 89.7% and 90.3% of the annotatable compounds in positive and negative ion modes, respectively."
- [intro] Annotation challenge addressed by the skill: "two additional processing steps are desired for LC/MS data analysis: 1) grouping all features which are derived from the same analyte, and 2) annotation of the type of ion species."
- [results] Reference dataset used for the accuracy comparison: "For the evaluation we used a mixture of 39 known compounds (short: MM39), covering a broad mass range between 161 and 822 Da"
- [results] Documented failure mode reducing the achievable accuracy: "they occurred either because the compound spectrum did not contain enough explained features, or in other cases several hypotheses had the same precedence scores"
- [results] Compounds whose spectra contain only molecular ion and fragment ions cannot be annotated without extra rules: "In this case the compound cannot be annotated directly, unless the neutral loss is added to the rule set."
- [intro] Pre-experimental constraint on sample concentration for valid annotations: "We made sure that the concentration of the samples do not lead to saturation of the MS detector system"
- [discussion] Where to obtain the tool used for the annotation being benchmarked: "The CAMERA packages for Windows, Mac OS and Linux, manuals and tutorials are freely available from the Bioconductor repository"
