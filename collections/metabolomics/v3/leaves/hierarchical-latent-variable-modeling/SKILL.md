---
name: hierarchical-latent-variable-modeling
description: Use when metabolomics feature data must be assigned to metabolites under
  a probabilistic model containing latent variables, and EM-based estimation of component
  means, variances, and mixing proportions is required.
when_to_use_negative:
- Input is a raw instrument file rather than preprocessed metabolomics features —
  run the MetPC preprocessing steps first
- The downstream goal is biomarker discovery rather than metabolite identification
  — use the fdr2d-controlled biomarker discovery step instead
- The model has no latent variables, so direct maximum-likelihood or closed-form estimation
  applies and EM iteration is unnecessary
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CNU-Bioinformatics-Lab (MetPC) repository
  role: Source repository for the MetPC pipeline code implementing the hierarchical
    statistical model with the EM algorithm for metabolite identification
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:d85bfe895fbf7a4466b099687d37c295c703294ff199918830f3d808c00fe41e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - CNU-Bioinformatics-Lab (MetPC) repository
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hierarchical-latent-variable-modeling

## Summary

Fits a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to handle latent variables during metabolite identification, as implemented in the MetPC pipeline (Kim & Jeong, Metabolites 2019, DOI 10.3390/metabo9050103).

## When to use

Use when metabolomics feature data must be assigned to metabolites under a probabilistic model containing latent variables, and EM-based estimation of component means, variances, and mixing proportions is required.

## When NOT to use

- Input is a raw instrument file rather than preprocessed metabolomics features — run the MetPC preprocessing steps first
- The downstream goal is biomarker discovery rather than metabolite identification — use the fdr2d-controlled biomarker discovery step instead
- The model has no latent variables, so direct maximum-likelihood or closed-form estimation applies and EM iteration is unnecessary

## Inputs

- Preprocessed metabolomics feature table (peak/intensity features) for the metabolite identification task
- Initial values for hierarchical model parameters: component means, variances, and mixing proportions
- Number of EM iterations to log (e.g., at least 200 to verify reported convergence)

## Outputs

- Converged estimates of the four hierarchical-model parameter blocks (means, variances, mixing proportions, latent-variable posteriors)
- Trace plots of the four EM parameter estimates across iterations (em_trace_plots.png, corresponding to Figure 6)
- Convergence log of parameter values per EM iteration

## How to apply

Load the preprocessed metabolomics feature data for the metabolite identification step of the MetPC pipeline. Initialize the hierarchical model parameters — component means, variances, and mixing proportions — then iterate the E-step (compute posterior expectations of the latent variables) and the M-step (update the four parameter blocks), logging parameter values at every iteration until they stabilize. Visualize convergence by generating trace plots of the four EM parameter estimates across iterations (e.g., with matplotlib, saved as em_trace_plots.png). In the article's benchmark, all four parameter trace chains plateau within the first 200 EM iterations, matching the convergence behavior shown in Figure 6; use that plateau as the acceptance criterion. Rationale: the EM coupling explicitly marginalizes latent variables so metabolite identification remains statistically coherent despite unobserved assignments.

## Related tools

- **CNU-Bioinformatics-Lab (MetPC) repository** (Source repository for the MetPC pipeline code implementing the hierarchical statistical model with the EM algorithm for metabolite identification) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Examples

```
import matplotlib.pyplot as plt; [plt.plot(traces[p]) for p in param_names]; plt.savefig('em_trace_plots.png')
```

## Evaluation signals

- All four EM parameter trace chains visually stabilize (plateau) within the first 200 iterations, consistent with Figure 6 of the article
- The E-step/M-step iteration log shows monotone stabilization of parameter values rather than oscillation or divergence
- Final mixing proportions are valid (non-negative, summing to 1) and variance estimates remain positive
- Output artifacts em_trace_plots.png and the per-iteration parameter log are produced and inspectable
- Latent-variable posterior expectations from the E-step are well-defined probabilities in [0, 1] at convergence

## Limitations

- Convergence verification in the article is visual inspection of trace plots (Figure 6), not a formal numerical convergence criterion
- The repository is indexed only under section 'other' with no detailed README content available in context, so exact parameterization of the four M-step blocks must be taken from the paper itself
- Behavior beyond the 200-iteration window is not characterized; chains that have not plateaued by ~200 iterations fall outside the reported convergence regime

## Evidence

- [abstract] EM handles latent variables: "we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to take care of latent variables."
- [abstract] pipeline scope: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
- [abstract] metabolite identification stage: "For metabolite identification, we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm"
