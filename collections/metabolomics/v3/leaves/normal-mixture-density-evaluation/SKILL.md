---
name: normal-mixture-density-evaluation
description: Use when dissimilarity scores from matched metabolite pairs have been
  fitted with a normal mixture via the EM algorithm and you must validate the fitted
  component means, variances, and mixing proportions against a non-parametric density
  estimate.
when_to_use_negative:
- The scores have not yet been fitted with an EM-estimated normal mixture, so there
  are no component means, variances, or mixing proportions to evaluate
- The analysis goal is biomarker discovery under fdr2d control rather than validating
  the density estimate of the metabolite-identification model
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: scipy.stats.gaussian_kde
  role: Computes the non-parametric Gaussian-kernel density estimate of the matched-pair
    dissimilarity scores for comparison against the parametric mixture density
- name: matplotlib
  role: Overlays the EM-mixture density and KDE curves on a single figure saved as
    figure7_density_comparison.png
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:7e23029d65996a5e788f4fa875b3a36d31e4c309735f449eae27eb44c082c166
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - matplotlib
  - scipy.stats.gaussian_kde
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# normal-mixture-density-evaluation

## Summary

Evaluates the EM-fitted three-component normal mixture density for matched metabolite-pair dissimilarity scores on a grid and overlays it with a Gaussian-kernel KDE to verify that the parametric estimate of the metabolite-identification hierarchical model is consistent with a non-parametric reference.

## When to use

Use when dissimilarity scores from matched metabolite pairs have been fitted with a normal mixture via the EM algorithm and you must validate the fitted component means, variances, and mixing proportions against a non-parametric density estimate.

## When NOT to use

- The scores have not yet been fitted with an EM-estimated normal mixture, so there are no component means, variances, or mixing proportions to evaluate
- The analysis goal is biomarker discovery under fdr2d control rather than validating the density estimate of the metabolite-identification model

## Inputs

- Array of dissimilarity scores for matched metabolite pairs from the metabolite-identification step
- EM-fitted mixture parameters: component means, variances, and mixing proportions (three-component normal mixture)

## Outputs

- Overlay plot of the parametric mixture density and the Gaussian-kernel KDE (e.g. figure7_density_comparison.png)
- Numerical agreement check between the mixture density and the KDE across the score grid

## How to apply

Load the matched-pair dissimilarity scores produced by the upstream metabolite-identification step of the MetPC pipeline, which uses a hierarchical statistical model coupled with an EM algorithm to handle latent variables. Evaluate the three-component normal mixture density at a grid of score values using the EM-estimated component means, variances, and mixing proportions. Independently compute a Gaussian-kernel kernel density estimate (e.g. scipy.stats.gaussian_kde) of the same scores. Overlay both curves with matplotlib and save the comparison (e.g. figure7_density_comparison.png). The decision point is visual and numerical agreement between the two densities: close overlap indicates the EM algorithm appropriately estimated the mean and variance of each component, reproducing Figure 7 of the article.

## Related tools

- **scipy.stats.gaussian_kde** (Computes the non-parametric Gaussian-kernel density estimate of the matched-pair dissimilarity scores for comparison against the parametric mixture density)
- **matplotlib** (Overlays the EM-mixture density and KDE curves on a single figure saved as figure7_density_comparison.png)

## Examples

```
from scipy.stats import gaussian_kde; import numpy as np; kde = gaussian_kde(scores); grid = np.linspace(scores.min(), scores.max(), 500); k = kde(grid)
```

## Evaluation signals

- The parametric three-component mixture density curve visually overlaps the kernel density estimate very well across the score range
- The two density estimates agree numerically pointwise on the evaluation grid, matching the comparison shown in Figure 7 of the article
- The output figure (figure7_density_comparison.png) contains both curves plotted over the same score range with comparable peak locations and heights
- The mixture components reflect the EM-fitted means, variances, and mixing proportions without rescaling or refitting

## Limitations

- Agreement is judged visually and by close overlap; the validation is a comparison criterion rather than a formal statistical test
- The skill presumes the upstream EM fit (three-component normal mixture) is available; it validates but does not itself fit the mixture

## Evidence

- [abstract] Parametric vs KDE agreement: "we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to take care of latent variables."
- [other] Workflow: KDE and overlay: [evidence span withheld — anchor status: not_found]
- [other] Workflow: mixture density evaluation: [evidence span withheld — anchor status: not_found]
- [other] Finding: overlap criterion: [evidence span withheld — anchor status: not_found]
