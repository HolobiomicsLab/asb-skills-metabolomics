---
name: scientific-task-formulation
description: Use when a research question about feature co-association (e.g., CLR
  scoring of MI networks to discount promiscuous features) must be formulated as a
  bounded, computable procedure with defined inputs, tools, and outputs before implementation.
when_to_use_negative:
- The analysis goal is simple pairwise similarity (e.g., Spearman/Pearson co-expression
  cutoffs such as Spearman ≥ 0.60) that MOG already supports natively without CLR
  normalization.
- 'No bounded computable operation can be specified — as the source card found for
  the article section itself: no CLR computation, MI background-distribution procedure,
  or output table is described — and no external CLR reference (e.g., Daub et al.)
  is available.'
- The dataset lacks enough samples to estimate MI background distributions reliably
  for each feature.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MetaOmGraph
  role: Java workbench whose Mutual Information module computes pairwise MI (B-spline
    estimator) on expression data, from which CLR background-distribution comparisons
    are formulated and executed.
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: R
  role: Optional external environment for additional analyses on data subsets sent
    from MOG.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:48a506f99e551c4a493cf5ada5ca5178b1f275e7dd704be7059075357bd07f6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MetaOmGraph
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scientific_task_formulation

## Summary

Converts a research question about feature co-association — here, MetaOmGraph's context likelihood of relatedness (CLR) scoring on mutual-information (MI) values — into a bounded, computable operation specifying inputs, tool steps, parameters, and an output table. It exists because the article's text alone did not yield a bounded CLR computation, so the method must be reconstructed from the MI/CLR description and external references.

## When to use

Use when a research question about feature co-association (e.g., CLR scoring of MI networks to discount promiscuous features) must be formulated as a bounded, computable procedure with defined inputs, tools, and outputs before implementation.

## When NOT to use

- The analysis goal is simple pairwise similarity (e.g., Spearman/Pearson co-expression cutoffs such as Spearman ≥ 0.60) that MOG already supports natively without CLR normalization.
- No bounded computable operation can be specified — as the source card found for the article section itself: no CLR computation, MI background-distribution procedure, or output table is described — and no external CLR reference (e.g., Daub et al.) is available.
- The dataset lacks enough samples to estimate MI background distributions reliably for each feature.

## Inputs

- Large expression dataset (e.g., MOG expression data with metadata, such as the Hu-cancer-RNASeq or AT-microarray datasets)
- MOG project file or indexed expression data loaded into MetaOmGraph
- Research question / task card specifying the CLR method to be operationalized

## Outputs

- Bounded computable operation specification (steps, parameters, decision points)
- Ranked feature-pair table with MI values, background-distribution-normalized scores, and CLR scores
- Set of biologically relevant feature associations with promiscuous features discounted

## How to apply

First, decompose the research question ('How does MOG implement CLR?') into computable primitives: pairwise MI estimation, background-distribution construction, and per-pair normalization. MOG computes CLR by comparing the MI value of each feature pair against the background distribution of MI values that include either member of the pair, so the formulated operation must specify: (1) compute pairwise MI on the expression matrix using MOG's Mutual Information module, which implements the B-spline MI estimator of Daub et al.; (2) for each pair, build the two background distributions of MI values involving each feature separately; (3) normalize each pair's MI against both backgrounds and combine them into a CLR score, ranking feature pairs so that features with promiscuous (high-connectivity) associations are discounted. Verify admissibility: the source card marks this task as 'not admissible' from article text alone — no bounded operation, no MI background-distribution procedure, and no output table are described — so the formulation must be grounded in the CLR description plus the Daub et al. reference cited in the MOG README. The output should be a ranked table of feature pairs with MI, background-normalized scores, and CLR values, judged correct if each score is derived from the pair's MI relative to both of its background distributions.

## Related tools

- **MetaOmGraph** (Java workbench whose Mutual Information module computes pairwise MI (B-spline estimator) on expression data, from which CLR background-distribution comparisons are formulated and executed.) — https://github.com/urmi-21/MetaOmGraph/
- **R** (Optional external environment for additional analyses on data subsets sent from MOG.)

## Examples

```
unzip MetaOmGraph.zip && java -jar MetaOmGraph.jar  # open MOG project, then run the Mutual Information (B-spline MI) module to obtain pairwise MI for CLR scoring
```

## Evaluation signals

- Each feature pair's CLR score is computed from its MI relative to both background distributions — the set of MI values including either member of the pair — not a single global MI distribution.
- Promiscuous features (those in many high-MI pairs) receive discounted CLR scores compared with a raw MI ranking; spot-check that a highly connected feature's top pairs rank lower than in a plain MI sort.
- The output is a concrete ranked table of feature pairs (not a narrative answer), matching the admissibility requirement for a bounded computable operation.
- MI estimation method is identified and citable — the MOG README requires citing Daub et al.'s B-spline MI estimator when the Mutual Information module is used.
- Task formulation is explicitly flagged as not derivable from the article section alone, and the source of the method (CLR description / Daub et al.) is cited.

## Limitations

- The article section text describes no CLR computation, no MI background-distribution procedure, and no output table, so the task is marked not admissible without supplementing from external method literature.
- MI estimation via B-spline functions depends on sample size and binning choices; background distributions for sparsely connected or low-expression features may be poorly estimated.
- CLR normalization discounts promiscuous features by design, so genuinely hub-like biology (rather than technical promiscuity) may be deprioritized.

## Evidence

- [other] CLR background-distribution definition: [evidence span withheld — anchor status: not_found]
- [readme] MI module method reference: [evidence span withheld — anchor status: external_doc]
- [intro] MOG tool scope: "we present MetaOmGraph (MOG), a Java software, to interactively explore and visualize large expression datasets"
- [results] All analyses generated in MOG: "The statistical analyses and visualizations shown were generated exclusively using MOG."
