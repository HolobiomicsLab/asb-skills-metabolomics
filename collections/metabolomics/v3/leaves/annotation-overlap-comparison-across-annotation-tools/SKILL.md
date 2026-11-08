---
name: annotation-overlap-comparison-across-annotation-tools
description: Use when evaluating a new or existing automated annotation workflow for
  AIF LC–MS data by benchmarking its annotations against a competing tool (e.g. MS-DIAL)
  and a manually curated expert list, to quantify agreement per dataset.
when_to_use_negative:
- Input is already a single-tool annotation output with no competing tool to compare
  — use a simple precision/recall evaluation against the expert list instead.
- The goal is de novo annotation quality assessment on raw spectra without a manually
  curated reference — use spectral-matching metrics (cosine similarity, ppm error)
  rather than intersection with an expert list.
- Only one dataset exists and no manual annotation reference is available — the overlap
  comparison cannot be computed.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnnotatoR
  role: Produces rank-1 annotations on AIF LC–MS features via correlation-based parent–fragment
    linking plus molecular fragment matching, one side of the comparison.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: State-of-the-art comparator; performs feature detection, MS2Dec deconvolution
    of AIF data, and library search against LipidBlast/METLIN/MassBank/GNPS to generate
    its annotation list.
- name: LipidBlast
  role: In-silico lipid spectral library searched by MS-DIAL during its annotation
    step; contributes to MS-DIAL's annotation output used in the overlap computation.
- name: METLIN
  role: Public MS/MS spectral database searched by MS-DIAL to obtain untargeted annotations
    for features.
- name: MassBank
  role: Public MS/MS spectral database searched by MS-DIAL to obtain untargeted annotations
    for features.
- name: GNPS
  role: Public MS/MS spectral database searched by MS-DIAL to obtain untargeted annotations
    for features.
- name: R
  role: Environment in which MetaboAnnotatoR is run and where the annotation lists
    are intersected and overlap statistics computed.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:43dc80cb7b4ce00da2540ab3579b31634af9251e62e3f453fc0601e55a2270da
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# annotation-overlap-comparison-across-annotation-tools

## Summary

Compares automated metabolite annotation outputs from different tools (e.g. MetaboAnnotatoR vs MS-DIAL) by intersecting their rank-1 or top-ranked annotation lists with a manually curated expert reference, reporting overlaps as counts and percentages per dataset.

## When to use

Use when evaluating a new or existing automated annotation workflow for AIF LC–MS data by benchmarking its annotations against a competing tool (e.g. MS-DIAL) and a manually curated expert list, to quantify agreement per dataset.

## When NOT to use

- Input is already a single-tool annotation output with no competing tool to compare — use a simple precision/recall evaluation against the expert list instead.
- The goal is de novo annotation quality assessment on raw spectra without a manually curated reference — use spectral-matching metrics (cosine similarity, ppm error) rather than intersection with an expert list.
- Only one dataset exists and no manual annotation reference is available — the overlap comparison cannot be computed.

## Inputs

- Raw centroided LC–MS AIF chromatograms (e.g. MESA Lipid+, Lipid−, HILIC+ QC datasets)
- Manually curated expert annotation lists (feature m/z, RT, compound identity) per dataset
- Annotation outputs from each tool (e.g. MS-DIAL annotated feature lists with in-source fragment/adduct assignments, MetaboAnnotatoR rank-1 annotations)
- Spectral/fragment libraries used by the tools (LipidBlast, METLIN, MassBank, GNPS)

## Outputs

- comparison_table.csv with per-dataset overlap counts and percentages (e.g. manual-vs-MetaboAnnotatoR rank-1 and manual-vs-MS-DIAL)
- Summary statistics of annotation agreement (e.g. 121/161 (75%) vs 89/161 (55%))
- Per-tool annotation feature lists ready for further statistical comparison

## How to apply

Run each annotation tool on the same raw AIF chromatograms — for MS-DIAL this means importing centroided LC–MS AIF files, running feature detection and MS2Dec deconvolution, then searching the deconvoluted pseudo-MS/MS spectra against LipidBlast and public libraries (METLIN, MassBank, GNPS). For MetaboAnnotatoR, extract the rank-1 annotation list from its output. Obtain a manually curated expert reference of annotated features (e.g. 161 features for Lipid+, 53 for Lipid−, 50 for HILIC+). Intersect each tool's annotations with the expert list to count matches, then compute overlap percentages. Tabulate counts and percentages for all datasets (e.g. comparison_table.csv) and validate against the reference figures (e.g. 121/161 vs 89/161 for Lipid+).

## Related tools

- **MetaboAnnotatoR** (Produces rank-1 annotations on AIF LC–MS features via correlation-based parent–fragment linking plus molecular fragment matching, one side of the comparison.) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (State-of-the-art comparator; performs feature detection, MS2Dec deconvolution of AIF data, and library search against LipidBlast/METLIN/MassBank/GNPS to generate its annotation list.)
- **LipidBlast** (In-silico lipid spectral library searched by MS-DIAL during its annotation step; contributes to MS-DIAL's annotation output used in the overlap computation.)
- **METLIN** (Public MS/MS spectral database searched by MS-DIAL to obtain untargeted annotations for features.)
- **MassBank** (Public MS/MS spectral database searched by MS-DIAL to obtain untargeted annotations for features.)
- **GNPS** (Public MS/MS spectral database searched by MS-DIAL to obtain untargeted annotations for features.)
- **R** (Environment in which MetaboAnnotatoR is run and where the annotation lists are intersected and overlap statistics computed.)

## Examples

```
install.packages('devtools', repos='http://cran.us.r-project.org', dependencies=TRUE); library(devtools); install_github('gggraca/MetaboAnnotatoR', dependencies=TRUE)
```

## Evaluation signals

- Overlap percentages match the published reference values exactly (e.g. manual-vs-MetaboAnnotatoR rank-1 = 121/161 for Lipid+, 37/50 for HILIC+, 37/53 for Lipid−; manual-vs-MS-DIAL = 89/161 for Lipid+, 31/50 for HILIC+, 39/53 for Lipid−).
- Denominator equals the manually curated expert list size per dataset (161 for Lipid+, 53 for Lipid−, 50 for HILIC+).
- Intersection is computed against rank-1 annotations for MetaboAnnotatoR (not the full ranked list) so both tools are compared at equivalent stringency.
- Output table includes all three datasets (Lipid+, Lipid−, HILIC+) with counts and percentages, not just one.
- Both tools are run on the identical raw AIF chromatograms and same feature detection settings so differences reflect annotation logic, not feature extraction.

## Limitations

- MS-DIAL annotations are performed in an untargeted manner; feature selection prior to annotation is not possible, so the comparison is against an untargeted tool rather than a subset-focused workflow.
- Comparison is limited to annotation agreement (rank-1 overlap with manual lists); it does not measure annotation correctness of each individual tool against spectral ground truth beyond the manual reference.
- Precision and recall of MetaboAnnotatoR itself vary by matrix (60–89% precision, 10–88% recall), so overlap with a competing tool can be high or low for reasons unrelated to either tool's absolute accuracy.
- Requires a manually curated expert annotation list of adequate size; without one, no denominator exists for the overlap metric.

## Evidence

- [results] MetaboAnnotatoR achieved a better overlap with manual annotations than MS-DIAL for Lipid+ [121/161 (75%) vs 89/161 (55%)] and HILIC+ [37/50 (74%) vs 31/50 (62%)], while for Lipid− MS-DIAL was slightly higher (39/53, 74% vs 37/53, 70%).: "MetaboAnnotatoR achieved a better overlap with manual annotations than MS-DIAL for Lipid+ [121/161 (75%) vs 89/161 (55%)] and HILIC+"
- [methods] Workflow step 1: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step 2: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step 3: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step 4: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step 5: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step 6: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step 7: [evidence span withheld — anchor status: not_found]
- [readme] README — package purpose: "This R package is designed to perform metabolite annotation of features from LC-MS All-ion fragmentation (AIF) datasets, using ion fragment databases."
- [readme] README — installation: [evidence span withheld — anchor status: external_doc]
