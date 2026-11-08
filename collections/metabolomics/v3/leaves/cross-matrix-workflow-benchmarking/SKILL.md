---
name: cross-matrix-workflow-benchmarking
description: Use when an annotation workflow validated on one matrix (e.g. human serum)
  must be tested on other biological matrices or instrument types, with precision/recall
  computed against expert manual annotations.
when_to_use_negative:
- The data are the original human serum development datasets — use the primary workflow
  evaluation, not the cross-matrix benchmark
- The goal is validating on higher mass accuracy instruments (e.g. Orbitrap) or non-AIF
  DIA schemes such as SWATH — explicitly untested/unsupported per the paper
- Input is MS/MS spectra for spectral-library matching rather than AIF pseudo-MS/MS
  feature annotation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: R package under evaluation; runs correlation-based parent–fragment linking
    plus molecular fragment matching on each external dataset
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: State-of-the-art baseline for AIF annotation used as the comparison benchmark
    for the cross-matrix precision/recall results
- name: R
  role: Execution environment; MetaboAnnotatoR is installed and run as an open-source
    R package
  repo: https://github.com/gggraca/MetaboAnnotatoR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:56cf339878f5a7c7830251498ab77d551cd1200635643b051c4d3aaa056ff897
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-matrix-workflow-benchmarking

## Summary

Benchmark a metabolite annotation workflow beyond its development matrix by running it on external validation datasets from different biological matrices and instrument types (adipose tissue ESI+/ESI−, amniotic fluid MTBLS666, urine MTBLS816) and computing per-dataset precision and recall against expert manual annotations.

## When to use

Use when an annotation workflow validated on one matrix (e.g. human serum) must be tested on other biological matrices or instrument types, with precision/recall computed against expert manual annotations.

## When NOT to use

- The data are the original human serum development datasets — use the primary workflow evaluation, not the cross-matrix benchmark
- The goal is validating on higher mass accuracy instruments (e.g. Orbitrap) or non-AIF DIA schemes such as SWATH — explicitly untested/unsupported per the paper
- Input is MS/MS spectra for spectral-library matching rather than AIF pseudo-MS/MS feature annotation

## Inputs

- Raw LC-MS AIF chromatograms (centroid mode) for external validation datasets: adipose tissue ESI+ and ESI−, amniotic fluid MTBLS666, urine MTBLS816
- Expert manual annotation tables for each validation dataset (ground truth)
- Metabolite/lipid fragment libraries (.txt or .msp) from the MetaboAnnotatoR GitHub repository

## Outputs

- Per-dataset annotated feature tables from the workflow
- Per-dataset precision and recall values (expected ranges: precision 60–89%, recall 10–88%; serum reference: 82–92% precision, 82–85% recall)
- Aggregated metabolite_annotation_results.csv summarizing annotation results and metrics (Table 2 equivalent)

## How to apply

Obtain the external validation datasets and fragment libraries from the workflow's public repository (here the MetaboAnnotatoR GitHub repo), then run the full R workflow — correlation-based parent–fragment linking combined with molecular fragment matching — on each dataset separately to produce annotated feature tables. Compare automated annotations feature-by-feature against the expert manual annotations to compute precision (fraction of correct annotations) and recall (fraction of manual annotations recovered), restricting to highest-rank scores (1–5) as in the paper's serum evaluation. Report per-dataset metrics in a single results file (metabolite_annotation_results.csv) and benchmark against both the original-matrix performance (82–92% precision, 82–85% recall on human serum) and the state-of-the-art baseline (MS-DIAL). Validate that per-dataset values fall within the reported ranges (precision 60–89%, recall 10–88% for the external datasets, Table 2) and interpret low recall — as on urine/amniotic fluid — against the known cause of false-negatives for nonlipid metabolites due to small fragment-library coverage.

## Related tools

- **MetaboAnnotatoR** (R package under evaluation; runs correlation-based parent–fragment linking plus molecular fragment matching on each external dataset) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (State-of-the-art baseline for AIF annotation used as the comparison benchmark for the cross-matrix precision/recall results)
- **R** (Execution environment; MetaboAnnotatoR is installed and run as an open-source R package) — https://github.com/gggraca/MetaboAnnotatoR

## Examples

```
library(devtools); install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE)
```

## Evaluation signals

- Per-dataset precision values fall within the reported 60–89% range and recall within 10–88% (Table 2)
- Serum reference check: benchmarking code reproduces 82–92% precision and 82–85% recall on the human serum datasets for high-rank scores (1–5)
- Low recall on urine (MTBLS816) and amniotic fluid (MTBLS666) is confirmed and attributable to false-negatives for nonlipid metabolites
- Results match or exceed MS-DIAL annotations on the same datasets
- All per-dataset metrics aggregate cleanly into metabolite_annotation_results.csv with one row per dataset

## Limitations

- Low recall on non-serum matrices stems from the small size of the fragment libraries, particularly the nonlipid library
- Orbitrap/higher mass accuracy instruments were not tested; only AIF is supported (no SWATH or other DIA schemes)
- Precision and recall are highly variable across matrices (60–89% and 10–88%), so single-dataset extrapolation is unreliable

## Evidence

- [abstract] Further validation for other biological matrices and different instrument types showed variable precision (60–89%) and recall (10–88%) particularly for datasets dominated by nonlipid metabolites: "Further validation for other biological matrices and different instrument types showed variable precision (60–89%) and recall (10–88%) particularly for datasets dominated by nonlipid metabolites"
- [discussion] good precision (82–92%) and recall (82–85%) values were obtained for human serum datasets for high-rank scores (1–5).: "good precision (82–92%) and recall (82–85%) values were obtained for human serum datasets for high-rank scores (1–5)."
- [discussion] These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library.: "These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library."
- [abstract] These results equal or outperform those obtained using MS-DIAL software, the current state of the art for AIF data annotation: "These results equal or outperform those obtained using MS-DIAL software, the current state of the art for AIF data annotation"
- [readme] For more details on how the software works, further testing and performance please read the full publication in Analytical Chemistry journal: [evidence span withheld — anchor status: external_doc]
