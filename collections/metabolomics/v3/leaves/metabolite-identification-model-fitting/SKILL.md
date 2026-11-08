---
name: metabolite-identification-model-fitting
description: Use when merged GCxGC/TOF-MS replicate peak lists must be scored against
  a reference metabolite library to identify which library metabolites are present,
  with posterior confidence per assignment.
when_to_use_negative:
- The goal is biomarker discovery rather than metabolite identification — that downstream
  step is governed by two-dimensional FDR (fdr2d) control, not the EM identification
  model
- Input is already a feature/quantification table with metabolite identities assigned,
  so no peak-to-library matching is needed
- No reference metabolite library spectra are available to compute cosine similarity
  candidate matches
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: MetPC
  role: Pipeline implementing preprocessing and the EM-based metabolite identification
    workflow
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
- name: Expectation-Maximization (EM) algorithm
  role: Fits the hierarchical statistical model that handles latent variables for
    peak-to-metabolite assignment
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
- name: CNU-Bioinformatics-Lab (MetPC) repository
  role: Code repository hosting the EM fitting and trace-plotting scripts
  repo: https://github.com/jjs3098/CNU-Bioinformatics-Lab
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.3390/metabo9050103
    title: 'MetPC: Metabolite Pipeline Consisting of Metabolite Identification and
      Biomarker Discovery Under the Control of Two-Dimensional FDR'
  source_content_hash: sha256:1d636f165a222ce3139cc59a3128ce95090c027acc115fff7c99bc90c46e15b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jjs3098/CNU-Bioinformatics-Lab
  tools:
  - CNU-Bioinformatics-Lab (MetPC) repository
  - Expectation-Maximization (EM) algorithm
  - MetPC
  - jjs3098/CNU-Bioinformatics-Lab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolite-identification-model-fitting

## Summary

Fits a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to assign GCxGC/TOF-MS peaks to metabolites from a reference library, yielding per-metabolite posterior confidence measures. It is the metabolite identification step of the MetPC pipeline.

## When to use

Use when merged GCxGC/TOF-MS replicate peak lists must be scored against a reference metabolite library to identify which library metabolites are present, with posterior confidence per assignment.

## When NOT to use

- The goal is biomarker discovery rather than metabolite identification — that downstream step is governed by two-dimensional FDR (fdr2d) control, not the EM identification model
- Input is already a feature/quantification table with metabolite identities assigned, so no peak-to-library matching is needed
- No reference metabolite library spectra are available to compute cosine similarity candidate matches

## Inputs

- Preprocessed GCxGC/TOF-MS replicate peak lists (e.g., two replicate runs of a 35-compound mixture, 76 merged peaks per run)
- Reference metabolite library replicate spectra for candidate matching
- Preprocessed metabolomics feature data used for the metabolite identification task

## Outputs

- CSV table of metabolite name and posterior confidence measure (e.g., metabolite_identification_dataset1.csv)
- Ranked list of metabolites by posterior probability (top-20 most likely present)
- EM parameter trace plots (em_trace_plots.png) showing convergence of the four parameter estimates

## How to apply

Load the replicate peak lists (e.g., Dataset1 of Experiment I: a 35-compound mixture yielding 76 merged peaks per run) and compute cosine similarity scores between each merged peak spectrum and the replicate spectra in the reference metabolite library to generate candidate matches. Fit the hierarchical statistical model with the EM algorithm — initializing component means, variances, and mixing proportions — to handle latent variables and estimate peak-to-metabolite assignments, logging the four parameter blocks at every iteration. Compute the posterior confidence measure for each candidate metabolite assignment in each replicate run, then rank metabolites by posterior probability and assemble a table with columns for metabolite name and posterior confidence measure. Verify convergence by generating trace plots of the four EM parameter estimates; the article reports all chains plateau within the first 200 iterations (Figure 6). Save the identification table (e.g., metabolite_identification_dataset1.csv) and check the top-20 ranked metabolites against Results 3.1.1-3.1.2.

## Related tools

- **MetPC** (Pipeline implementing preprocessing and the EM-based metabolite identification workflow) — https://github.com/jjs3098/CNU-Bioinformatics-Lab
- **Expectation-Maximization (EM) algorithm** (Fits the hierarchical statistical model that handles latent variables for peak-to-metabolite assignment) — https://github.com/jjs3098/CNU-Bioinformatics-Lab
- **CNU-Bioinformatics-Lab (MetPC) repository** (Code repository hosting the EM fitting and trace-plotting scripts) — https://github.com/jjs3098/CNU-Bioinformatics-Lab

## Examples

```
import matplotlib.pyplot as plt; plt.plot(range(len(param_trace)), param_trace); plt.savefig('em_trace_plots.png')
```

## Evaluation signals

- All four EM parameter trace chains visually stabilize within the first 200 iterations, consistent with Figure 6
- Top-20 metabolites ranked by posterior probability reproduce the identification results reported in Results 3.1.1-3.1.2 for Dataset1
- Output table contains exactly the per-run columns: metabolite name and posterior confidence measure
- Each replicate run contains the expected number of merged peaks (76 for Dataset1's 35-compound mixture)

## Limitations

- Convergence is validated visually from trace plots within 200 iterations rather than by a formal numerical stopping criterion
- Validation results are tied to Dataset1 of Experiment I (35-compound mixture); other datasets require re-checking against the corresponding article sections
- No discussion-section text was available to detail caveats or failure modes of the identification step

## Evidence

- [abstract] Hierarchical model + EM for latent variables: "we considered a hierarchical statistical model coupled with an Expectation-Maximization (EM) algorithm to take care of latent variables."
- [results] EM convergence within 200 iterations: [evidence span withheld — anchor status: not_found]
- [results] Identification output table and top-20 ranking: [evidence span withheld — anchor status: not_found]
