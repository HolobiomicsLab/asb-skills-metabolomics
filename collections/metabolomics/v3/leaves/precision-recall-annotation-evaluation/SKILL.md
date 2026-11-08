---
name: precision-recall-annotation-evaluation
description: Use when you have run an automated annotation workflow on an AIF LC–MS
  dataset and possess a manually curated reference list of annotated features, and
  need to quantify annotation accuracy as precision and recall, optionally restricted
  to high-rank candidates (ranks 1–5).
when_to_use_negative:
- No expert-manually annotated reference set exists — precision and recall cannot
  be computed without ground-truth annotations; report only raw annotation counts
  instead.
- The goal is to evaluate annotation of DIA schemes other than AIF (e.g., SWATH) or
  of higher mass-accuracy instruments (e.g., Orbitrap) — the paper states these were
  not implemented or tested with this workflow.
- The dataset is dominated by nonlipid metabolites and the nonlipid fragment library
  is small; expect low recall (as low as 10–88%) and interpret the metric as a library-coverage
  limitation rather than a workflow failure.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: Produces the automated candidate annotations that are scored against the manual
    reference list; its annotateAIF and annotateRC functions are the prediction source
    for the precision/recall computation.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: Benchmark tool whose annotation performance is compared against MetaboAnnotatoR
    in the precision/recall evaluation.
- name: R
  role: Runtime environment for MetaboAnnotatoR and for computing precision/recall
    metrics on the annotation results.
  repo: https://github.com/gggraca/MetaboAnnotatoR
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:636e4b9a24f02d67297b00cbee2616a50e271f5dd77ec48b865fa66c85604fd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# precision-recall-annotation-evaluation

## Summary

Quantify the annotation quality of an automated LC–MS metabolite annotation workflow by computing precision and recall against a set of expert-manually annotated reference features, as done in MetaboAnnotatoR validation on 599 human serum features (precision 82–92%, recall 82–85% for rank 1–5).

## When to use

Use when you have run an automated annotation workflow on an AIF LC–MS dataset and possess a manually curated reference list of annotated features, and need to quantify annotation accuracy as precision and recall, optionally restricted to high-rank candidates (ranks 1–5).

## When NOT to use

- No expert-manually annotated reference set exists — precision and recall cannot be computed without ground-truth annotations; report only raw annotation counts instead.
- The goal is to evaluate annotation of DIA schemes other than AIF (e.g., SWATH) or of higher mass-accuracy instruments (e.g., Orbitrap) — the paper states these were not implemented or tested with this workflow.
- The dataset is dominated by nonlipid metabolites and the nonlipid fragment library is small; expect low recall (as low as 10–88%) and interpret the metric as a library-coverage limitation rather than a workflow failure.

## Inputs

- Annotated feature table produced by the automated workflow (MetaboAnnotatoR or similar), with each feature carrying candidate annotations ranked by score
- Expert-manually annotated reference feature list (e.g., 599 features across the three human serum QC datasets: MESA Lipid+, Lipid−, HILIC+)
- Optionally: rank-score threshold (e.g., keep only ranks 1–5) to restrict which annotations are counted as predictions

## Outputs

- Per-dataset precision and recall values for the automated annotations
- Aggregated metrics table (e.g., metabolite_annotation_results.csv) comparing automated vs. manual annotation
- Diagnostic stratification showing performance separately for lipid vs. nonlipid metabolite classes

## How to apply

First run the automated annotation (e.g., MetaboAnnotatoR's annotateAIF on lipid-mode data using LipidPos/LipidNeg libraries, and annotateRC on nonlipid data using MetabolitesPos/MetabolitesNeg libraries) so every feature receives candidate annotations that are ranked by score; restrict evaluation to the top-ranked candidates (ranks 1–5) as the paper does. Then intersect the automated annotations with the expert-manual reference list: precision = correctly annotated features / total annotated features, recall = correctly annotated features / total manually annotated features. Compare results across matrices and instrument types — the paper reports 82–92% precision and 82–85% recall on three human serum AIF datasets, but 60–89% precision and 10–88% recall on adipose tissue, MTBLS666 amniotic fluid and MTBLS816 urine, where low recall stems from the small nonlipid fragment library. Aggregate per-dataset metrics into a single results file (e.g., metabolite_annotation_results.csv) for reporting.

## Related tools

- **MetaboAnnotatoR** (Produces the automated candidate annotations that are scored against the manual reference list; its annotateAIF and annotateRC functions are the prediction source for the precision/recall computation.) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (Benchmark tool whose annotation performance is compared against MetaboAnnotatoR in the precision/recall evaluation.)
- **R** (Runtime environment for MetaboAnnotatoR and for computing precision/recall metrics on the annotation results.) — https://github.com/gggraca/MetaboAnnotatoR

## Examples

```
Rscript -e 'library(MetaboAnnotatoR); ann <- annotateAIF(xcms_peaktable, "LipidPos"); metrics <- precision_recall(ann, manual_annotated, ranks=1:5); write.csv(metrics, "metabolite_annotation_results.csv")'
```

## Evaluation signals

- Precision on the three human serum AIF datasets falls within the reported 82–92% range for rank 1–5 annotations.
- Recall on the same datasets falls within the reported 82–85% range for rank 1–5 annotations.
- Precision and recall on the four external validation datasets (adipose ESI+/ESI−, MTBLS666, MTBLS816) fall within 60–89% and 10–88%, respectively.
- Per-dataset metrics in the aggregated CSV match the values reported in Table 2 of the paper.
- Annotation comparison is restricted to features present in the manually annotated reference list; features outside it are excluded from the recall denominator.

## Limitations

- Low recall on urine (MTBLS816) and amniotic fluid (MTBLS666) datasets is attributable to the small size of the nonlipid fragment library, not necessarily to a flaw in the scoring algorithm.
- The workflow was demonstrated on AIF LC–MS data acquired at unit-mass resolution; performance on higher mass accuracy instruments (e.g., Orbitraps) is untested.
- Only AIF is currently supported; DIA schemes such as SWATH are not handled by the workflow.
- MetaboAnnotatoR is designed to focus on a subset of features (e.g., those reaching statistical significance) rather than annotate the full dataset, so recall estimates depend on which feature subset was submitted to annotation.

## Evidence

- [abstract] Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5): "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)"
- [abstract] Further validation for other biological matrices and different instrument types showed variable precision (60–89%) and recall (10–88%) particularly for datasets dominated by nonlipid metabolites: "Further validation for other biological matrices and different instrument types showed variable precision (60–89%) and recall (10–88%) particularly for datasets dominated by nonlipid metabolites"
- [discussion] Running MetaboAnnotatoR on the three human serum datasets containing 599 expert-annotated features yields precision of 82–92% and recall of 82–85% for rank 1–5 annotations.: [evidence span withheld — anchor status: not_found]
- [discussion] These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library.: "These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library."
- [other] together with the fragment libraries from Github (https://github.com/gggraca/MetaboAnnotatoR): "together with the fragment libraries from Github (https://github.com/gggraca/MetaboAnnotatoR)"
