---
name: cross-validated-spearman-correlation-evaluation
description: Use when comparing metabolite-prediction models (e.g., MiMeNet vs. MelonnPan
  or NED) on paired microbiome-metabolome datasets and you need per-metabolite SCC
  plus a count of well-predicted metabolites.
when_to_use_negative:
- You only have unpaired or single-omics data (no paired microbial and metabolite
  feature tables) - the SCC protocol requires both profiles per sample.
- You need mechanistic or biological interpretation rather than prediction-accuracy
  benchmarking - the article notes all these methods 'are still limited in providing
  biological plausibility and mechanistic insights'.
- Your metabolite features lack any microbial association signal - the paper warns
  'not all metabolites may be associated with microbes', so some metabolites will
  have low SCC regardless of model quality.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
tools:
- name: MiMeNet
  role: Generates metabolite predictions and the cross-validated SCC evaluation via
    its -num_run_cv, -num_cv, and -threshold arguments; reference implementation of
    the 10x10-fold protocol.
  repo: github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Per-metabolite Elastic Net linear regression baseline evaluated under the
    identical 10x10-fold CV to produce the comparison SCC values (0.108, 0.276, -0.272,
    0.168).
  repo: biobakery/melonnpan
- name: BiomeNED
  role: Encoder-decoder neural network comparator also scored with cross-validated
    SCC on the IBD (External) dataset.
  repo: vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:e0f0ecb92d3faecc290f51fcf42906e2c26b2a862f4b0b1c3193ad33c2ef2d3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiomeNED
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# cross-validated-spearman-correlation-evaluation

## Summary

Evaluates microbiome-to-metabolome predictors by running repeated (10 iterations of 10-fold) cross-validation and scoring each metabolite via held-out Spearman correlation coefficients (SCC), distinguishing well-predicted metabolites from a shuffled background.

## When to use

Use when comparing metabolite-prediction models (e.g., MiMeNet vs. MelonnPan or NED) on paired microbiome-metabolome datasets and you need per-metabolite SCC plus a count of well-predicted metabolites.

## When NOT to use

- You only have unpaired or single-omics data (no paired microbial and metabolite feature tables) - the SCC protocol requires both profiles per sample.
- You need mechanistic or biological interpretation rather than prediction-accuracy benchmarking - the article notes all these methods 'are still limited in providing biological plausibility and mechanistic insights'.
- Your metabolite features lack any microbial association signal - the paper warns 'not all metabolites may be associated with microbes', so some metabolites will have low SCC regardless of model quality.

## Inputs

- paired microbiome (samples x microbial features) and metabolome (samples x metabolite features) CSV matrices for IBD (PRISM), cystic fibrosis, and soil datasets
- external IBD validation dataset (microbial and metabolite feature tables)
- optional annotation file marking the subset of annotated metabolites
- cross-validation configuration: number of CV iterations (10) and folds (10)

## Outputs

- per-metabolite Spearman correlation coefficients from held-out folds
- mean SCC across metabolites per dataset
- count of well-predicted metabolites (SCC above the 95th percentile of shuffled background)
- comparison table against reference values, e.g. melonnpan_scc_comparison.csv

## How to apply

Split paired microbial-feature and metabolite-abundance matrices into 10 iterations of 10-fold cross-validation, sharing hyper-parameters tuned from the first partition across all iterations (per MiMeNet). For each metabolite, compute the Spearman correlation coefficient between true and predicted abundances on held-out folds; for the IBD external dataset, evaluate SCC on the fixed test-set predictions instead. To identify well-predicted metabolites, generate background SCC distributions by shuffling sample labels in each CV iteration and keep metabolites whose SCC exceeds the 95th percentile of that background. Aggregate the mean SCC across metabolites and the number of well-predicted metabolites per dataset to compare against reference values (IBD PRISM 0.309/366, CF 0.457/143, soil 0.264/29, IBD external 0.275/308 for MiMeNet). This protocol applies equally when benchmarking the MelonnPan Elastic Net baseline (mean SCC 0.108/0.276/-0.272/0.168 and 198/104/4/186 well-predicted metabolites) and NED, allowing direct model comparison under the identical CV scheme.

## Related tools

- **MiMeNet** (Generates metabolite predictions and the cross-validated SCC evaluation via its -num_run_cv, -num_cv, and -threshold arguments; reference implementation of the 10x10-fold protocol.) — github.com/YDaiLab/MiMeNet
- **MelonnPan** (Per-metabolite Elastic Net linear regression baseline evaluated under the identical 10x10-fold CV to produce the comparison SCC values (0.108, 0.276, -0.272, 0.168).) — biobakery/melonnpan
- **BiomeNED** (Encoder-decoder neural network comparator also scored with cross-validated SCC on the IBD (External) dataset.) — vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro ibd_micro.csv -metab ibd_metab.csv -output out/ -micro_norm RA -metab_norm RA -num_run_cv 10 -num_cv 10
```

## Evaluation signals

- Mean SCC across all metabolites matches reported values: IBD PRISM ~0.309, cystic fibrosis ~0.457, soil ~0.264, IBD external ~0.275 for MiMeNet.
- Number of well-predicted metabolites exceeds the 95th-percentile shuffled background and matches reference counts (366/143/29/308 for MiMeNet; 198/104/4/186 for MelonnPan).
- Shuffled-background SCC distributions must be generated by permuting samples in each CV iteration - well-predicted counts depend on this background.
- Hyper-parameters are tuned once from the first CV partition and shared across all 10 iterations (no per-fold re-tuning).
- External IBD validation uses a fixed test set rather than CV; SCC is computed on those predictions and reported separately (~0.275 mean).

## Limitations

- Not all metabolites are microbe-associated, so some metabolites have low SCC and the overall mean correlation is depressed.
- The soil dataset shows a higher well-predicted threshold, possibly due to its longitudinal observations (unresolved in the paper).
- The evaluation measures prediction accuracy only - the article notes all methods including MiMeNet 'are still limited in providing biological plausibility and mechanistic insights'.
- SCC can be negative on some datasets (e.g., soil baseline -0.272), meaning per-metabolite values should be inspected before interpreting the mean.

## Evidence

- [abstract] mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264: "Using ten iterations of 10-fold cross-validation on three paired microbiome-metabolome datasets"
- [supplementary] 10 iterations of 10-fold CV with shared hyperparameters: "Using 10-iterations of 10-fold cross-validation, evaluations using shared hyper-parameters tuned from the first partition"
- [methods] SCC-based well-predicted metabolite identification: "Then models are evaluated in a cross-validated fashion resulting in Spearman correlation coefficients (SCC) for each metabolite"
- [methods] 95th percentile background threshold: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [readme] background generation via shuffling: "-background BACKGROUND            Directory with previously generated background"
- [readme] per-metabolite SCC in MelonnPan training summary: [evidence span withheld — anchor status: external_doc]
- [abstract] negative baseline SCC in soil: "mean Spearman correlation coefficients increase from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264"
