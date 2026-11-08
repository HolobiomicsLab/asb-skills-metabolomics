---
name: soft-threshold-power-selection
description: Use when building a weighted co-accumulation/co-expression network (e.g.,
  WGCNA) from a metabolite peak-area or expression matrix and a scale-free topology
  fit must be used to pick the soft-thresholding power before dynamic tree cutting.
when_to_use_negative:
- Input is already a module assignment or module eigengene table — the power has already
  been chosen and no network needs to be built.
- Only two-group differential abundance testing is needed (use MetMiner's statistical-analysis/MDAtoolkits
  tools instead of WGCNA).
- The data are raw RNA-seq read counts that first require vst normalization and noise
  filtering per the WGCNA FAQ; power selection should follow that preprocessing, not
  precede it.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: WGCNA-shinyApp
  role: Standalone Shiny app driving the WGCNA workflow (expression-matrix filtering,
    power selection, module detection) on peak-area or expression matrices
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: MetMiner
  role: Pipeline hosting the iterative WGCNA strategy in which the soft-thresholding
    power is chosen from the scale-free topology fit in each of four rounds
  repo: https://github.com/ShawnWx2019/MetMiner
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:3da0b9a32849ffa7ca7b488a6965b9152c44c44924bb407f57e2ecb03ea40483
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/WGCNA-shinyApp
  tools:
  - MDAtoolkits
  - MetMiner
  - WGCNA-shinyApp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# soft-threshold-power-selection

## Summary

Selection of the WGCNA soft-thresholding power from the scale-free topology fit prior to module detection in the MetMiner iterative WGCNA strategy, which is used to screen biomarker metabolites from plant metabolomics peak-area matrices.

## When to use

Use when building a weighted co-accumulation/co-expression network (e.g., WGCNA) from a metabolite peak-area or expression matrix and a scale-free topology fit must be used to pick the soft-thresholding power before dynamic tree cutting.

## When NOT to use

- Input is already a module assignment or module eigengene table — the power has already been chosen and no network needs to be built.
- Only two-group differential abundance testing is needed (use MetMiner's statistical-analysis/MDAtoolkits tools instead of WGCNA).
- The data are raw RNA-seq read counts that first require vst normalization and noise filtering per the WGCNA FAQ; power selection should follow that preprocessing, not precede it.

## Inputs

- Expression matrix of metabolite peak areas (tab-delimited .txt/.tsv; metabolite ID per row, sample ID per column, e.g., corrected peak-picking table from tidyMass)
- Optional trait table (quantitative or qualitative; sample_id plus trait columns, e.g., treat/control or plant_height/yield)

## Outputs

- Selected soft-thresholding power (from the scale-free topology fit)
- Scale-free topology fit diagnostic plots
- Co-abundance module assignments for metabolites (e.g., 12 modules over 1,365 metabolites in the case study)

## How to apply

Format the quantified features (e.g., 1,365 metabolite peak areas across the 206 Arabidopsis F-box mutant samples) as a tab-delimited matrix with metabolite IDs in rows and sample IDs in columns — the WGCNA-shinyApp README forbids spaces or special symbols in sample IDs and does not accept .csv/.xlsx, only .txt/.tsv. Compute pairwise metabolite correlations and pick the soft-thresholding power by examining the scale-free topology fit, as prescribed in the MetMiner iterative WGCNA workflow. Feed the resulting power parameter into module detection with dynamic tree cut; in the MetMiner case study this was repeated iteratively (removing the metabolite with lowest module membership each round) until 12 modules covering 1,365 metabolites were produced after four rounds. MetMiner wraps this in a Shiny GUI (run_metMiner) or the standalone WGCNA-shinyApp so users without programming experience can drive the whole analytical process.

## Related tools

- **WGCNA-shinyApp** (Standalone Shiny app driving the WGCNA workflow (expression-matrix filtering, power selection, module detection) on peak-area or expression matrices) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **MetMiner** (Pipeline hosting the iterative WGCNA strategy in which the soft-thresholding power is chosen from the scale-free topology fit in each of four rounds) — https://github.com/ShawnWx2019/MetMiner

## Examples

```
cd WGCNAshiny && Rscript WGCNAbyClick.v1.R
```

## Evaluation signals

- The chosen power corresponds to a scale-free topology fit consistent with WGCNA's soft-thresholding recommendation before dynamic tree cut is run.
- After the final iterative round, exactly 12 modules are produced covering all 1,365 metabolites, matching the reported case-study outcome; any divergence is flagged.
- The input matrix passes format checks: tab-delimited .txt/.tsv, metabolite IDs in rows, sample IDs in columns, no spaces/special symbols in sample IDs, no purely numeric IDs.
- Downstream module assignments support ClassyFire over-representation enrichment (seven of twelve modules significant in the case study), indicating the network structure was built on a valid power.

## Limitations

- Given the complexity of WGCNA analysis, MetMiner exposes it through a separate Shiny app; the pipeline targets users without programming experience rather than fine-grained parameter control.
- Low-abundance/non-varying features reflect noise and correlations based on mostly-zero counts are not meaningful, so expression filtering must precede power selection (thresholds depend on experimental design, sequencing depth and sample counts).
- No version number, license persistence guarantee for the hosted Shiny apps, or containerized deployment is documented in the article for reproducing the exact power-choice interface.

## Evidence

- [full_text] compute pairwise metabolite correlations, choose the soft-thresholding power from the scale-free topology fit: [evidence span withheld — anchor status: not_found]
- [abstract] iterative WGCNA strategy in abstract: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [discussion] WGCNA-shinyapp introduced in discussion: [evidence span withheld — anchor status: not_found]
- [readme] peak area input format in README: [evidence span withheld — anchor status: external_doc]
- [readme] noise-filtering rationale in README: [evidence span withheld — anchor status: external_doc]
