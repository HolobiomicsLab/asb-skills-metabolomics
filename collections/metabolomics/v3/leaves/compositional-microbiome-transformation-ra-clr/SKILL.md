---
name: compositional-microbiome-transformation-ra-clr
description: Use when input is a samples-by-features CSV of raw microbial or metabolite
  counts (e.g., IBD PRISM, cystic fibrosis, soil paired datasets) that must be made
  compositional prior to MiMeNet training.
when_to_use_negative:
- The count matrices are already relative-abundance or CLR transformed — applying
  a second compositional transform would distort them; use 'None' normalization instead.
- The goal is modeling metabolites individually with regularized linear regression
  (MelonnPan-style), where this neural-network preprocessing framing does not apply.
- The analysis is conditional-probability microbe-metabolite co-occurrence estimation
  (mmvec), not whole-metabolome-profile prediction from taxonomic features.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: MiMeNet
  role: Implements the RA/CLR compositional transformation and low-abundance feature
    filtering in its Data Preprocessing stage via the -micro_norm and -metab_norm
    parameters
  repo: https://github.com/YDaiLab/MiMeNet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:e5d6c93f7b268da3ab2c9b3cc7195e46af1f3f9b88fef607345602d6b900a84f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - MiMeNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# compositional-microbiome-transformation-ra-clr

## Summary

Preprocessing skill from MiMeNet that applies a compositional transformation—relative abundance (RA) or centered log-ratio (CLR)—to paired microbiome and metabolome count matrices and filters low-abundant features before neural-network metabolite prediction.

## When to use

Use when input is a samples-by-features CSV of raw microbial or metabolite counts (e.g., IBD PRISM, cystic fibrosis, soil paired datasets) that must be made compositional prior to MiMeNet training.

## When NOT to use

- The count matrices are already relative-abundance or CLR transformed — applying a second compositional transform would distort them; use 'None' normalization instead.
- The goal is modeling metabolites individually with regularized linear regression (MelonnPan-style), where this neural-network preprocessing framing does not apply.
- The analysis is conditional-probability microbe-metabolite co-occurrence estimation (mmvec), not whole-metabolome-profile prediction from taxonomic features.

## Inputs

- micro CSV: comma-delimited matrix of samples by microbial count features
- metab CSV: comma-delimited matrix of samples by metabolite count features
- micro_norm / metab_norm setting: 'RA', 'CLR', or 'None' (when already transformed)

## Outputs

- Compositional (RA or CLR) sample-by-feature matrices for microbiome and metabolome, with low-abundant features removed, fed into MiMeNet cross-validated SCC evaluation

## How to apply

Load the paired microbiome and metabolome CSVs and choose a normalization per matrix via the -micro_norm and -metab_norm flags, selecting RA or CLR; pass 'None' if the data are already transformed. For the shipped IBD (PRISM) dataset the README example applies CLR to the metabolome and skips transformation of the microbiome. After transformation, filter low-abundant microbial and metabolite features so rare features do not dominate the MLPNN training. The RA-vs-CLR choice is a decision point: the article evaluates both, so keep it consistent across training, external validation (external_micro/external_metab), and the shuffled-sample background runs used to define well-predicted metabolites.

## Related tools

- **MiMeNet** (Implements the RA/CLR compositional transformation and low-abundance feature filtering in its Data Preprocessing stage via the -micro_norm and -metab_norm parameters) — https://github.com/YDaiLab/MiMeNet

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbiome_PRISM.csv -metab data/IBD/metabolome_PRISM.csv -micro_norm None -metab_norm CLR -num_run_cv 10 -output IBD
```

## Evaluation signals

- Transformed microbiome and metabolite columns each sum to constant composition (RA) or are valid log-ratio values (CLR), with 'None' confirming no double transformation when inputs are pre-normalized.
- Low-abundance microbial and metabolite features are filtered after transformation, so downstream MiMeNet cross-validation (10 iterations of 10-fold CV) runs on the retained feature set.
- With CLR applied to metabolome and 'None' to microbiome (per the IBD example), reproduced mean SCC values (0.309 IBD annotated, 0.457 CF, 0.264 soil) fall within tolerance of the reported values.
- The same normalization choice is carried through to the external IBD test set and the shuffled-sample background distribution used for the 95th-percentile well-predicted metabolite threshold.

## Limitations

- Not all metabolites are microbe-associated, so even after correct transformation some metabolites retain low prediction correlations, lowering overall mean SCC.
- The soil dataset showed a higher significance threshold, possibly due to longitudinal observations (unresolved in the article).
- Reported performance comparisons (RA vs CLR scatterplots) were generated for the IBD (PRISM) and cystic fibrosis datasets; the optimal transform may differ for new datasets.

## Evidence

- [readme] Transform the microbial features into relative abundance (RA) or center log-ratio (CLR). If the data is already transformed, apply 'None' to skip transformation.: "Transform the microbial features into relative abundance (RA) or center log-ratio (CLR). If the data is already transformed, apply 'None' to skip transformation."
- [methods] MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [supplementary] Scatterplots comparing metabolite correlation prediction between data transformed to relative abundance (RA) and centered log-ratio (CLR) for (A) IBD (PRISM): "Scatterplots comparing metabolite correlation prediction between data transformed to relative abundance (RA) and centered log-ratio (CLR) for (A) IBD (PRISM)"
