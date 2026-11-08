---
name: training-convergence-curve-assessment
description: Use when training the scFEA/FLUXestimator flux solver on single-cell
  RNA-seq data and you need to judge goodness of fit via the convergence curves of
  the flux balance loss and non-negative loss before trusting predicted fluxes.
when_to_use_negative:
- Input is not transcriptomics data — the article notes flux estimation for proteomics
  or metabolomics data is not yet supported
- You need a fully quantitative acceptance threshold rather than qualitative plateau
  inspection; the paper only demonstrates qualitative convergence (Figure S6) and
  formal fluxome evaluation was still to be released
- You only need downstream analyses (clustering, perturbation) with already-converged
  flux outputs and no retraining
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scFEA
  role: Stand-alone PyTorch implementation whose training loop produces the per-epoch
    flux balance and non-negative losses to be assessed; outputs predicted_flux and
    balance files
  repo: https://github.com/changwn/scFEA
- name: FLUXestimator
  role: Web-server implementation of the scFEA pipeline whose flux balance is a quadratic
    loss function, framing the convergence criterion being assessed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad444
    title: 'FLUXestimator: a webserver for predicting metabolic flux and variations
      using transcriptomics data'
  source_content_hash: sha256:6efa45b2c93bb4ffb94baefdaf648619337a9375b296a23de9b2ef73cf447ccb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/changwn/scFEA
  repo_url: https://github.com/changwn/scFEA
  tools:
  - FLUXestimator
  - scFEA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# training-convergence-curve-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess the training convergence of the scFEA flux-balance neural-network solver by plotting the flux balance (quadratic) loss, the non-negative loss, and the total loss versus training epoch, verifying that the losses decrease and plateau, qualitatively reproducing the paper's supplementary Figure S6 behavior.

## When to use

Use when training the scFEA/FLUXestimator flux solver on single-cell RNA-seq data and you need to judge goodness of fit via the convergence curves of the flux balance loss and non-negative loss before trusting predicted fluxes.

## When NOT to use

- Input is not transcriptomics data — the article notes flux estimation for proteomics or metabolomics data is not yet supported
- You need a fully quantitative acceptance threshold rather than qualitative plateau inspection; the paper only demonstrates qualitative convergence (Figure S6) and formal fluxome evaluation was still to be released
- You only need downstream analyses (clustering, perturbation) with already-converged flux outputs and no retraining

## Inputs

- Single-cell RNA-seq count or normalized count matrix (genes x cells, e.g. CSV in input_dir such as Melissa_full.csv)
- Module-gene mapping file (e.g. module_gene_m168.csv for human)
- Stoichiometry matrix linking intermediate metabolites to modules (e.g. cmMat_171.csv)
- Per-epoch training loss values (flux balance loss, non-negative loss, total loss) from the PyTorch training loop

## Outputs

- Loss-versus-epoch convergence plot (flux balance loss, non-negative loss, total loss)
- Convergence verdict (plateaued vs. not converged) for the flux-balance optimization
- Predicted per-module flux matrix (output_flux_file, e.g. predicted_flux.csv) and metabolite balance/stress matrix (output_balance_file), released only if convergence is acceptable

## How to apply

Run scFEA (python src/scFEA.py) on the single-cell input matrix, which trains one multi-layer feed-forward PyTorch network per metabolic module and minimizes a quadratic mass-balance loss plus a non-negativity penalty with the Adam optimizer. During training, record the flux balance loss, the non-negative loss, and their total at each epoch, then plot all three curves versus epoch in the Figure S6 style (blue = flux balance loss, red = non-negative loss, black dash = total loss). Judge convergence by confirming each loss decreases monotonically and reaches a plateau rather than oscillating or diverging; the README notes convergence curves are the basis for evaluating goodness of fit of the predicted fluxome. If the curves fail to plateau (e.g. on very large datasets without GPU acceleration), switch to the sampling-then-fitting strategy: train on a sampled subset of cells per cell type, then use the fitting function to predict flux on all cells.

## Related tools

- **scFEA** (Stand-alone PyTorch implementation whose training loop produces the per-epoch flux balance and non-negative losses to be assessed; outputs predicted_flux and balance files) — https://github.com/changwn/scFEA
- **FLUXestimator** (Web-server implementation of the scFEA pipeline whose flux balance is a quadratic loss function, framing the convergence criterion being assessed)

## Examples

```
python src/scFEA.py --input_dir data --res_dir output --test_file Melissa_full.csv
```

## Evaluation signals

- The flux balance (quadratic) loss and non-negative loss both decrease monotonically and plateau over epochs, matching the Figure S6 pattern on the pancreatic cancer cell line data
- The total loss (black-dash curve) converges to a stable value rather than oscillating or diverging
- Converged runs yield finite, non-negative predicted fluxes written to output_flux_file and a metabolite balance matrix written to output_balance_file
- For datasets >10,000 cells, the sampling-then-fitting strategy is used when full-batch training fails to converge within reasonable time without GPU acceleration

## Limitations

- Formal quantitative evaluation of the predicted fluxome (goodness-of-fit benchmarks and cross-validation examples) was listed as to-be-released in the README, so assessment remains largely qualitative
- For datasets with more than 10,000 single cells, the general pipeline is too time-consuming without GPU acceleration, complicating convergence checks
- Loss hyperparameters for the flux balance and non-negative losses are set separately, and different settings form different experiments, affecting comparability of convergence curves

## Evidence
