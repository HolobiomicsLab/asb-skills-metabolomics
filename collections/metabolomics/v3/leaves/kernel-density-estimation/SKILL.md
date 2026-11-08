---
name: kernel-density-estimation
description: Use when a set of metabolite-pair dissimilarity scores exists and you
  need to validate that a parametric (EM-fitted normal mixture) density faithfully
  represents the empirical distribution, e.g. reproducing Figure 7 of MetPC.
when_to_use_negative:
- The scores are not one-dimensional dissimilarity values from matched metabolite
  pairs (the validation logic assumes a single unimodal-to-trimodal score axis)
- You only need clustering or assignment of scores to mixture components, not a density-vs-density
  agreement check
- A fitted mixture is already benchmarked against a known distribution, making a non-parametric
  KDE redundant
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: scipy.stats.gaussian_kde
  role: Computes the non-parametric Gaussian-kernel density estimate of the dissimilarity
    scores against which the EM-mixture density is compared
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: matplotlib
  role: Overlays the parametric mixture density and KDE curves on a single plot saved
    as figure7_density_comparison.png
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: manual expert review
  role: Visual inspection of the overlaid density curves to confirm close agreement
    between the two estimates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:aeae49d1868808f2db07ba6ec7225ac2278fc3002f4b0b8f9b36e499a68a70e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - manual expert review
  - matplotlib
  - scipy.stats.gaussian_kde
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# kernel-density-estimation

## Summary

Non-parametric density estimation of matched-pair metabolite-identification dissimilarity scores using a Gaussian-kernel KDE, used as an independent check that the EM-fitted three-component normal mixture in the MetPC pipeline captured the score distribution correctly.

## When to use

Use when a set of metabolite-pair dissimilarity scores exists and you need to validate that a parametric (EM-fitted normal mixture) density faithfully represents the empirical distribution, e.g. reproducing Figure 7 of MetPC.

## When NOT to use

- The scores are not one-dimensional dissimilarity values from matched metabolite pairs (the validation logic assumes a single unimodal-to-trimodal score axis)
- You only need clustering or assignment of scores to mixture components, not a density-vs-density agreement check
- A fitted mixture is already benchmarked against a known distribution, making a non-parametric KDE redundant

## Inputs

- Vector of dissimilarity scores for matched metabolite pairs (one-dimensional numeric array from the metabolite-identification step)
- EM-fitted normal-mixture parameters (component means, variances, mixing proportions)

## Outputs

- figure7_density_comparison.png — overlay plot of the EM-mixture density and the Gaussian-kernel KDE on a shared score grid
- Numerical density values for both estimators at each grid point

## How to apply

Load the matched-pair dissimilarity scores produced by the upstream metabolite-identification step of the MetPC pipeline (the hierarchical statistical model handled by the EM algorithm). Define a grid of score values and evaluate the parametric density on it using the EM-estimated component means, variances, and mixing proportions (three normal components). Independently compute a Gaussian-kernel density estimate of the same scores with scipy.stats.gaussian_kde. Overlay both curves in a single matplotlib figure and save it as figure7_density_comparison.png. Rationale: because the EM algorithm treats latent variables in the hierarchical model, the KDE provides a non-parametric ground truth; close visual and numerical agreement confirms the EM appropriately estimated each component's mean and variance.

## Related tools

- **scipy.stats.gaussian_kde** (Computes the non-parametric Gaussian-kernel density estimate of the dissimilarity scores against which the EM-mixture density is compared) — jjs3098/CNU-Bioinformatics-Lab
- **matplotlib** (Overlays the parametric mixture density and KDE curves on a single plot saved as figure7_density_comparison.png) — jjs3098/CNU-Bioinformatics-Lab
- **manual expert review** (Visual inspection of the overlaid density curves to confirm close agreement between the two estimates)

## Examples

```
from scipy.stats import gaussian_kde; kde = gaussian_kde(scores); dens = kde(grid)
```

## Evaluation signals

- The EM-mixture density curve visually overlaps the KDE across the full score range (success criterion of Figure 7)
- Pointwise density differences between the two estimates remain small over the grid (numerical agreement, not just visual)
- The output file figure7_density_comparison.png is produced with both curves on a common axis
- The three-component mixture parameters (means, variances, mixing proportions) yield a density whose modes align with KDE mode locations
- The KDE bandwidth is the scipy default or is documented, so the comparison is reproducible

## Limitations

- Agreement is judged by visual and numerical comparison; the workflow defines no formal goodness-of-fit threshold between the mixture and KDE
- The check validates the EM fit for matched-pair dissimilarity scores only; it does not validate downstream biomarker discovery under fdr2d control
- No discussion-section guidance is available in the extracted article text, so bandwidth choice and edge behavior of the KDE are unspecified

## Evidence

- [other] The parametric density estimate (three-component normal mixture) overlaps the kernel density estimate very well, indicating the EM algorithm appropriately estimated the mean and variance of each component.: [evidence span withheld — anchor status: not_found]
- [other] Workflow step specifying the KDE tool.: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract on the EM component of metabolite identification.: "we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to take care of latent variables."
