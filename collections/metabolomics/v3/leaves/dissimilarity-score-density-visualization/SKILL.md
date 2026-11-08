---
name: dissimilarity-score-density-visualization
description: Use when you have matched-pair dissimilarity scores from a metabolite-identification
  step and need to verify that a parametric normal-mixture density (EM-fitted) agrees
  with a non-parametric kernel density estimate.
when_to_use_negative:
- You have raw mass-spectrometry peak tables or preprocessing-stage data rather than
  matched-pair dissimilarity scores from the metabolite-identification step
- You need to test biomarker significance under two-dimensional FDR (fdr2d) control
  — that is a separate biomarker-discovery procedure, not a density-model validation
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: scipy.stats.gaussian_kde
  role: Computes the non-parametric Gaussian-kernel density estimate of the dissimilarity
    scores for comparison against the EM mixture density
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: matplotlib
  role: Overlays the parametric mixture density and KDE curves on a single plot and
    saves the figure
  repo: jjs3098/CNU-Bioinformatics-Lab
- name: EM algorithm (three-component normal mixture)
  role: Supplies the parametric density estimate (component means, variances, mixing
    proportions) fitted to the latent-variable model for metabolite identification
  repo: jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:66367e9528bc2dda2a1dc717fdbc17fe97babdb9d95ea602aec25fdecfeef777
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - EM algorithm (three-component normal mixture)
  - matplotlib
  - scipy.stats.gaussian_kde
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dissimilarity-score-density-visualization

## Summary

Validates the EM-fitted three-component normal mixture model for metabolite matched-pair dissimilarity scores by overlaying its parametric density on a Gaussian-kernel non-parametric KDE, reproducing the Figure 7 density comparison of the MetPC article.

## When to use

Use when you have matched-pair dissimilarity scores from a metabolite-identification step and need to verify that a parametric normal-mixture density (EM-fitted) agrees with a non-parametric kernel density estimate.

## When NOT to use

- You have raw mass-spectrometry peak tables or preprocessing-stage data rather than matched-pair dissimilarity scores from the metabolite-identification step
- You need to test biomarker significance under two-dimensional FDR (fdr2d) control — that is a separate biomarker-discovery procedure, not a density-model validation

## Inputs

- Array of dissimilarity scores for matched metabolite pairs (output of the EM-based metabolite-identification step)
- EM-fitted mixture parameters: component means, variances, and mixing proportions (three components)

## Outputs

- Overlay plot of the three-component normal-mixture density and the Gaussian-kernel KDE saved as figure7_density_comparison.png
- Numerical agreement between the parametric mixture density and the KDE across the score range

## How to apply

Load the dissimilarity scores computed in the upstream metabolite-identification step of the MetPC pipeline, where a hierarchical statistical model with an EM algorithm handles latent variables. Evaluate the normal-mixture density on a grid of score values using the EM-fitted component means, variances, and mixing proportions (a three-component mixture is expected here). Independently compute a Gaussian-kernel KDE of the same scores with scipy.stats.gaussian_kde, then overlay both density curves on a single matplotlib plot saved as figure7_density_comparison.png. The rationale is that close overlap between the parametric and non-parametric curves demonstrates the EM algorithm appropriately estimated the mean and variance of each mixture component.

## Related tools

- **scipy.stats.gaussian_kde** (Computes the non-parametric Gaussian-kernel density estimate of the dissimilarity scores for comparison against the EM mixture density) — jjs3098/CNU-Bioinformatics-Lab
- **matplotlib** (Overlays the parametric mixture density and KDE curves on a single plot and saves the figure) — jjs3098/CNU-Bioinformatics-Lab
- **EM algorithm (three-component normal mixture)** (Supplies the parametric density estimate (component means, variances, mixing proportions) fitted to the latent-variable model for metabolite identification) — jjs3098/CNU-Bioinformatics-Lab

## Examples

```
from scipy.stats import gaussian_kde; import matplotlib.pyplot as plt; kde = gaussian_kde(scores); plt.plot(grid, kde(grid)); plt.savefig('figure7_density_comparison.png')
```

## Evaluation signals

- The three-component normal-mixture density curve visually overlaps the Gaussian-kernel KDE closely across the full score range, matching Figure 7 of the article
- Pointwise numerical agreement between mixture density and KDE on a shared grid of score values (small relative differences)
- The fitted mixture has exactly three components with EM-estimated means, variances, and mixing proportions
- The output plot file figure7_density_comparison.png is produced and contains both overlaid density curves

## Limitations

- The validation relies on visual and numerical agreement rather than a formal goodness-of-fit test; close overlap is the stated success criterion
- Agreement is specific to the dissimilarity scores from the MetPC EM-based metabolite identification; a different number of mixture components may not reproduce the Figure 7 comparison

## Evidence

- [abstract] hierarchical EM mixture density vs KDE: "For metabolite identification, we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm"
- [abstract] tool purpose context: "we developed a new bioinformatics tool that includes a few of preprocessing steps and biomarker discovery procedure."
