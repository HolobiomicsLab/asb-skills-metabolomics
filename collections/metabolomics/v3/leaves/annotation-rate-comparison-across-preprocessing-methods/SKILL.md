---
name: annotation-rate-comparison-across-preprocessing-methods
description: Use when you have feature tables produced by multiple LC/MS preprocessing
  methods or settings and need an objective downstream metric — the fraction of features
  annotated to known metabolites — to rank their quality (e.g. reproducing Fig. 3b).
when_to_use_negative:
- When the goal is to compare preprocessing methods on sample classification or batch-effect
  metrics rather than metabolite annotation coverage
- When the data are acquired in positive-ion mode and the paper's negative-mode adduct
  list would give spurious matches — a positive-mode adduct list must replace it
- When only a single preprocessing method's feature table is available, since the
  skill's purpose is cross-method comparison
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: xMSAnnotator
  role: Annotates feature m/z values against the KEGG database using the negative-mode
    adduct list; annotation counts feed the proportion metric
- name: apLCMS
  role: Produces the feature tables under comparison (single-batch, per-batch, and
    two-stage hybrid preprocessing settings)
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Environment for running apLCMS/xMSAnnotator and computing/tabulating the annotation
    proportions
- name: XCMS
  role: Alternative preprocessing method (centWave peak detection) whose feature table
    is benchmarked against the apLCMS variants
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:160a5559c31ba8d9672ec3468b7aca6f3811351a2b1d1ca43c4fbb481a3076c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - XCMS
  - apLCMS
  - xMSAnnotator
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# annotation-rate-comparison-across-preprocessing-methods

## Summary

A benchmarking skill that compares LC/MS feature tables from different preprocessing methods (two-stage apLCMS, traditional apLCMS, XCMS) by measuring the proportion of features whose m/z can be matched to known metabolites (e.g. KEGG) via xMSAnnotator with a fixed negative-mode adduct list, as done for the ST000868 dataset in the batch-effect preprocessing paper.

## When to use

Use when you have feature tables produced by multiple LC/MS preprocessing methods or settings and need an objective downstream metric — the fraction of features annotated to known metabolites — to rank their quality (e.g. reproducing Fig. 3b).

## When NOT to use

- When the goal is to compare preprocessing methods on sample classification or batch-effect metrics rather than metabolite annotation coverage
- When the data are acquired in positive-ion mode and the paper's negative-mode adduct list would give spurious matches — a positive-mode adduct list must replace it
- When only a single preprocessing method's feature table is available, since the skill's purpose is cross-method comparison

## Inputs

- Feature tables (m/z, retention time, intensity per sample) from each preprocessing method: apLCMS single-batch, apLCMS per-batch, apLCMS two-stage hybrid, and XCMS centWave
- Reference metabolite database: KEGG compound list with neutral masses
- Negative-mode adduct ion list ([M-H]-, [M-2H]2-, [M-2H+Na]-, [M-2H+K]-, [M-2H+NH4]-, [M-H2O-H]-, [M-H+Cl]2-, [M+Cl]-, [M+2Cl]2-)

## Outputs

- Per-method annotation proportions (annotated features / total features), saved as fig3b_annotation_proportions.csv with one proportion per preprocessing setting
- Fig. 3b-style bar chart comparing annotation rates across preprocessing methods

## How to apply

Load each feature table (e.g. from single-batch apLCMS preprocessing, per-batch apLCMS preprocessing, the two-stage hybrid apLCMS method, and XCMS with centWave peak detection). For each table, run xMSAnnotator with KEGG as the reference metabolite database and the paper's negative-mode adduct ion list: [M-H]-, [M-2H]2-, [M-2H+Na]-, [M-2H+K]-, [M-2H+NH4]-, [M-H2O-H]-, [M-H+Cl]2-, [M+Cl]-, [M+2Cl]2-. xMSAnnotator matches each feature's m/z to KEGG compound masses through the allowed adduct transformations. Compute the proportion of annotated features (annotated features divided by total features) per preprocessing setting; the rationale is that a better preprocessing pipeline recovers more real features whose m/z are consistent with known metabolites. Assemble the proportions into a comparison table (fig3b_annotation_proportions.csv) and draw a bar chart; expect similar rates across methods with XCMS/centWave slightly higher.

## Related tools

- **xMSAnnotator** (Annotates feature m/z values against the KEGG database using the negative-mode adduct list; annotation counts feed the proportion metric)
- **apLCMS** (Produces the feature tables under comparison (single-batch, per-batch, and two-stage hybrid preprocessing settings)) — https://github.com/tianwei-yu/apLCMS
- **R** (Environment for running apLCMS/xMSAnnotator and computing/tabulating the annotation proportions)
- **XCMS** (Alternative preprocessing method (centWave peak detection) whose feature table is benchmarked against the apLCMS variants)

## Examples

```
ann <- xMSAnnotator::annotateNeighborhood(feature_table, adductList = c("[M-H]-","[M-2H]2-","[M-2H+Na]-","[M-2H+K]-","[M-2H+NH4]-","[M-H2O-H]-","[M-H+Cl]2-","[M+Cl]-","[M+2Cl]2-"), db = "KEGG")
```

## Evaluation signals

- fig3b_annotation_proportions.csv exists and contains exactly one proportion value per preprocessing setting
- Each computed proportion lies in the interval [0, 1] (annotated features / total features)
- The bar chart reproduces the relative ordering reported in Fig. 3b: methods perform similarly, with XCMS centWave slightly higher
- The same KEGG database and identical nine-adduct negative-mode list were applied to every feature table, so differences reflect preprocessing, not annotation settings

## Limitations

- Annotation rate is a proxy metric: it reflects m/z consistency with known metabolites, not correctness of peak detection or quantification across batches
- The comparison result (XCMS centWave slightly higher) is specific to the ST000868 dataset and negative-mode adduct list; other datasets/modes may order methods differently
- Traditional single-group preprocessing can misalign peaks in ways post hoc correction cannot fix, which may bias downstream annotation counts for that method

## Evidence

- [results] When feature tables from the three preprocessing methods on ST000868 were matched to KEGG metabolites via xMSAnnotator with the negative-mode adduct list, the methods performed similarly, with XCMS with centWave peak detection yielding a slightly higher rate of matching (Fig. 3b).: [evidence span withheld — anchor status: not_found]
- [intro] such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches: "such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [abstract] the new method resulted in feature tables with better consistency, as well as better down-stream analysis results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [methods] cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model: "`cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
