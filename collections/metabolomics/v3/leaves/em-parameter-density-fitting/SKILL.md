---
name: em-parameter-density-fitting
description: Use when you have fitted mixture-model parameters (component means, variances,
  mixing weights) from an EM step and need to check they yield a density consistent
  with the empirical distribution of the underlying scores.
when_to_use_negative:
- Input scores come from a model with no EM-fitted mixture parameters, so there is
  no parametric density to validate
- The goal is biomarker discovery / fdr2d multiple-testing control rather than validating
  metabolite-identification model parameters
edam_operation: http://edamontology.org/operation_0320
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: scipy.stats.gaussian_kde
  role: Non-parametric Gaussian kernel density estimator used as the reference density
    against which the EM mixture density is compared
- name: matplotlib
  role: Overlay of the parametric mixture density and the KDE on a single plot saved
    as figure7_density_comparison.png
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:fcbd8b370ab2f84eefcca7886e71433ee02f5338a1216e8f3eb9b9801f217457
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - matplotlib
  - scipy.stats.gaussian_kde
  edam_operation: http://edamontology.org/operation_0320
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# em-parameter-density-fitting

## Summary

Evaluate the density implied by EM-fitted parameters (means, variances, mixing proportions) of a hierarchical normal-mixture model and cross-validate it against a non-parametric Gaussian-kernel density estimate on the same scores, as done for matched-pair metabolite-identification dissimilarity scores in MetPC.

## When to use

Use when you have fitted mixture-model parameters (component means, variances, mixing weights) from an EM step and need to check they yield a density consistent with the empirical distribution of the underlying scores.

## When NOT to use

- Input scores come from a model with no EM-fitted mixture parameters, so there is no parametric density to validate
- The goal is biomarker discovery / fdr2d multiple-testing control rather than validating metabolite-identification model parameters

## Inputs

- Vector of dissimilarity scores for matched metabolite pairs (output of the EM-based metabolite-identification step)
- EM-fitted mixture parameters: per-component means, variances, and mixing proportions

## Outputs

- Density values of the parametric normal mixture and the Gaussian KDE on a common score grid
- Overlay plot of both density curves saved as figure7_density_comparison.png

## How to apply

Load the dissimilarity scores of matched metabolite pairs computed in the upstream metabolite-identification step. Evaluate the mixture density over a grid of score values using the EM-estimated component means, variances, and mixing proportions (MetPC uses a hierarchical statistical model coupled with an EM algorithm to handle latent variables). Independently compute a kernel density estimate of the same scores with a Gaussian-kernel KDE (scipy.stats.gaussian_kde). Overlay both curves on a single matplotlib plot (reproducing Figure 7) and judge success by close visual and numerical agreement between the parametric mixture density and the KDE across the score range — agreement indicates the EM algorithm appropriately estimated the mean and variance of each component.

## Related tools

- **scipy.stats.gaussian_kde** (Non-parametric Gaussian kernel density estimator used as the reference density against which the EM mixture density is compared)
- **matplotlib** (Overlay of the parametric mixture density and the KDE on a single plot saved as figure7_density_comparison.png)

## Examples

```
from scipy.stats import gaussian_kde, norm; grid = np.linspace(scores.min(), scores.max(), 500); mixture_d = sum(w*norm.pdf(grid, mu, np.sqrt(s2)) for w, mu, s2 in zip(props, means, variances)); kde_d = gaussian_kde(scores)(grid)
```

## Evaluation signals

- The EM-mixture density curve overlaps the Gaussian-kernel KDE closely across the full score range (visual check, Figure 7 reproduction)
- Numerical agreement between the two density estimates on the shared evaluation grid is close (small pointwise differences)
- The number of mixture components (three) and their fitted means/variances produce a density whose modes match the empirical KDE modes
- Output file figure7_density_comparison.png is produced with both curves plotted on one axis

## Limitations

- Validation is visual and numerical agreement with KDE; a poor fit does not by itself diagnose which EM component (mean vs. variance vs. proportion) is misestimated
- The KDE is itself bandwidth-dependent, so agreement does not prove absolute parametric correctness
- The article provides no discussion-section text in the available context, so caveats about edge-case score distributions are not documented

## Evidence

- [abstract] For metabolite identification, we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm: "For metabolite identification, we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm"
- [other] workflow: evaluate mixture density and KDE: [evidence span withheld — anchor status: not_found]
- [other] finding: parametric overlaps KDE: [evidence span withheld — anchor status: not_found]
- [other] workflow: save comparison figure: [evidence span withheld — anchor status: not_found]
