---
name: computational-runtime-comparison-reporting
description: Use when a paper claims computational feasibility of a pipeline (e.g.,
  'not prohibitively so') and you need to reproduce or report per-stage runtimes on
  a defined input, such as one MS/MS sample with 300 Mass2Motifs.
when_to_use_negative:
- The goal is algorithmic correctness or output-quality validation (e.g., whether
  substructure annotations are right at a 0.1 probability threshold), not computational
  cost measurement
- The input is not a single, well-defined sample — timing heterogeneous batches makes
  comparison against single-sample reference figures meaningless
- The hardware differs fundamentally from laptop-class hardware and no published reference
  for that class exists, making same-order-of-magnitude validation impossible
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MS2LDA
  role: 'Pipeline being benchmarked: its feature-extraction stage (bag-of-fragments
    conversion with neutral-loss extraction and noise filtering) and its two LDA inference
    engines (Gibbs sampling, variational Bayes) are individually timed'
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:97ae0c912096c512a4182765aec0c6e76a33e5492c6acb3a424b8f6985cc1489
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# computational-runtime-comparison-reporting

## Summary

Benchmarking skill for stage-wise wall-clock timing of a multi-stage scientific workflow (here, MS2LDA feature extraction and two LDA inference engines, Gibbs sampling vs. variational Bayes) and comparing measured times against published reference figures on laptop-class hardware.

## When to use

Use when a paper claims computational feasibility of a pipeline (e.g., 'not prohibitively so') and you need to reproduce or report per-stage runtimes on a defined input, such as one MS/MS sample with 300 Mass2Motifs.

## When NOT to use

- The goal is algorithmic correctness or output-quality validation (e.g., whether substructure annotations are right at a 0.1 probability threshold), not computational cost measurement
- The input is not a single, well-defined sample — timing heterogeneous batches makes comparison against single-sample reference figures meaningless
- The hardware differs fundamentally from laptop-class hardware and no published reference for that class exists, making same-order-of-magnitude validation impossible

## Inputs

- Beer3 positive-ionization mode MS/MS data file from deposited research data (dx.doi.org/10.5525/gla.researchdata.313)
- MS2LDA pipeline configuration with 300 Mass2Motifs per data file

## Outputs

- runtime_benchmark.csv with per-stage wall-clock timings (feature extraction, Gibbs sampling, variational Bayes)
- Comparison table mapping measured timings to the paper's reported figures on laptop-class hardware

## How to apply

Run each pipeline stage in isolation with Python's time module wrapping the stage calls: (1) the preprocessing/feature-extraction stage that converts MS/MS data into bag-of-fragments vectors with neutral-loss extraction and noise filtering; (2) LDA inference by Gibbs sampling using the paper's setting of 300 Mass2Motifs per data file; (3) LDA inference by variational Bayes. Use a single, well-defined input — for MS2LDA, the Beer3 positive-ionization mode file from the deposited data (dx.doi.org/10.5525/gla.researchdata.313), which after preprocessing contains around 1,000 MS peaks. Record wall-clock time per stage, compile into a stage-wise table (e.g., runtime_benchmark.csv) alongside the published reference figures, and judge success by each stage falling within the same order of magnitude as the reported value (~20 min feature extraction, ~1 h Gibbs sampling, ~30 min variational Bayes).

## Related tools

- **MS2LDA** (Pipeline being benchmarked: its feature-extraction stage (bag-of-fragments conversion with neutral-loss extraction and noise filtering) and its two LDA inference engines (Gibbs sampling, variational Bayes) are individually timed) — https://github.com/vdhooftcompmet/MS2LDA

## Examples

```
import time; t0=time.time(); run_ms2lda_feature_extraction(beer3_mzml); t1=time.time(); lda_gibbs(n_motifs=300); t2=time.time(); lda_vb(n_motifs=300); t3=time.time(); print(t1-t0, t2-t1, t3-t2)
```

## Evaluation signals

- Feature-extraction stage timing is ~20 minutes (same order of magnitude) on one beer sample
- Gibbs-sampling LDA inference timing is ~1 hour, roughly 2x the variational Bayes time
- Variational Bayes inference timing is ~30 minutes on the same hardware
- Each stage's measured time falls within a plausible tolerance band (same order of magnitude) of the corresponding reported figure
- Timings are reproducible when the run is repeated with identical settings (300 Mass2Motifs, same input file)

## Limitations

- Reference figures are laptop-class hardware dependent; substantially different CPUs, memory, or Python/library versions shift absolute times while preserving relative ordering
- MS2LDA is computationally more costly than simpler tools, though the paper argues 'not prohibitively so' — timings will look slow next to lightweight spectral-matching approaches
- Runtimes scale with sample complexity: the ~20 min / ~1 h / ~30 min figures apply to a preprocessed beer sample of around 1,000 MS peaks; complex mixtures with information-rich MS2 spectra will take longer

## Evidence

- [discussion] Computationally, MS2LDA is more costly than simpler tools, but not prohibitively so.: "Computationally, MS2LDA is more costly than simpler tools, but not prohibitively so."
- [results] Three hundred Mass2Motifs were extracted for each data file and checked for biochemical relevance.: "Three hundred Mass2Motifs were extracted for each data file and checked for biochemical rel- evance."
- [results] After preprocessing, each sample consisted of around 1,000 MS peaks: "After preprocessing, each sample consisted of around 1,000 MS peaks"
- [discussion] Decomposition of spectra into Mass2Motifs using variational Bayesian inference: [evidence span withheld — anchor status: not_found]
- [results] MS2LDA workflow was independently applied to four beer extracts: "The MS2LDA workflow was independently applied to four beer extracts."
