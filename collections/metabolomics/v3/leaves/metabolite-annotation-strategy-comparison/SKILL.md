---
name: metabolite-annotation-strategy-comparison
description: Use when each query molecule has multiple constituent spectra (e.g.,
  three collision-energy levels) and you must decide whether to merge spectra into
  a consensus spectrum before candidate ranking or aggregate per-spectrum rankings
  afterward.
when_to_use_negative:
- Molecules with fewer or more than three constituent spectra — the evaluation is
  explicitly limited to the test subset where a molecule has exactly three spectra,
  so counts differ from 1,531 cases
- Only a single query spectrum per molecule is available — no consensus spectrum or
  rank aggregation is meaningful
- The goal is a MassSpecGym challenge other than molecule retrieval (e.g., de novo
  molecule generation or spectrum simulation), where ranking strategies do not apply
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MVP
  role: 'Provides the candidate-ranking model used in both arms: mol-cs consensus-spectrum
    ranking view (aggregate-then-rank) and per-spectrum ranking with average/reciprocal-rank
    aggregation; run via test.py with params_formSpec.yaml or params_binnedSpec.yaml
    and the pretrained checkpoint'
  repo: https://github.com/HassounLab/MVP
- name: MassSpecGym
  role: Benchmark supplying the test set (split into molecules with exactly three
    spectra) and the candidate retrieval settings (by mass, by formula) used for the
    rank@1 comparison
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: lightning
  role: PyTorch Lightning framework underlying MVP's checkpoint loading and test-time
    candidate-ranking execution
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:cda7546a41eff3dcdb136c50b4d8df2d55efe597b70bb8548220188e6944c6d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/MVP
  tools:
  - MVP
  - MassSpecGym
  - lightning
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metabolite-annotation-strategy-comparison

## Summary

Benchmarks two inference strategies for molecules with multiple query MS/MS spectra: aggregate-then-rank (consensus spectrum fed to MVP's mol-cs ranking view) versus rank-then-aggregate (average rank/reciprocal rank over per-spectrum candidate rankings), evaluated as rank@1 on the MassSpecGym benchmark.

## When to use

Use when each query molecule has multiple constituent spectra (e.g., three collision-energy levels) and you must decide whether to merge spectra into a consensus spectrum before candidate ranking or aggregate per-spectrum rankings afterward.

## When NOT to use

- Molecules with fewer or more than three constituent spectra — the evaluation is explicitly limited to the test subset where a molecule has exactly three spectra, so counts differ from 1,531 cases
- Only a single query spectrum per molecule is available — no consensus spectrum or rank aggregation is meaningful
- The goal is a MassSpecGym challenge other than molecule retrieval (e.g., de novo molecule generation or spectrum simulation), where ranking strategies do not apply

## Inputs

- MassSpecGym test-set spectra restricted to molecules with exactly three constituent spectra (one per collision energy level)
- Candidate molecule lists per query (retrieved by mass and by formula)
- Pretrained MVP checkpoint (trained on MassSpecGym)
- MVP configuration file: params_formSpec.yaml or params_binnedSpec.yaml

## Outputs

- rank@1 (and rank@k) for the consensus-spectrum (aggregate-then-rank) strategy per candidate setting
- rank@1 (and rank@k) for the rank-then-aggregate strategy (average rank, reciprocal rank) per candidate setting
- Structured CSV comparison table of the two strategies

## How to apply

Load the MassSpecGym test set and restrict it to molecules with exactly three constituent spectra (1,531 cases in the paper's results). For the aggregate-then-rank arm, run MVP's candidate ranking with the pretrained MassSpecGym checkpoint via test.py (params_formSpec.yaml or params_binnedSpec.yaml), using the mol-cs ranking view that consumes the consensus spectrum. For the rank-then-aggregate arm, rank candidates for each constituent spectrum individually, then combine per-spectrum rankings into a molecule-level ranking using average rank and reciprocal rank aggregation. Compute rank@1 (and rank@k) for both arms over the same subset and candidates — both by mass and by formula. The expected outcome, serving as a validation target, is that consensus-spectrum annotation wins on rank@1 in both settings: 35.99% vs. 26.37% for mass-candidates and 13.96% vs. 11.10% for formula-candidates. Save the side-by-side comparison as a structured CSV.

## Related tools

- **MVP** (Provides the candidate-ranking model used in both arms: mol-cs consensus-spectrum ranking view (aggregate-then-rank) and per-spectrum ranking with average/reciprocal-rank aggregation; run via test.py with params_formSpec.yaml or params_binnedSpec.yaml and the pretrained checkpoint) — https://github.com/HassounLab/MVP
- **MassSpecGym** (Benchmark supplying the test set (split into molecules with exactly three spectra) and the candidate retrieval settings (by mass, by formula) used for the rank@1 comparison) — https://github.com/pluskal-lab/MassSpecGym
- **lightning** (PyTorch Lightning framework underlying MVP's checkpoint loading and test-time candidate-ranking execution)

## Examples

```
python test.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- Subset size check: exactly 1,531 test molecules with exactly three spectra after filtering
- Consensus arm reproduces rank@1 ≈ 35.99% (mass-candidates) and ≈ 13.96% (formula-candidates)
- Rank-then-aggregate arm reproduces rank@1 ≈ 26.37% (average/rank aggregation, mass-candidates) and ≈ 11.10% (formula-candidates)
- Invariant: consensus (aggregate-then-rank) rank@1 must exceed the aggregated per-spectrum rank@1 in both candidate settings
- Final comparison delivered as a structured CSV with both strategies' rank@1 side by side

## Limitations

- The comparison is limited to the test subset where each molecule has exactly three spectra (1,531 cases); conclusions may not extend to molecules with more or fewer spectra
- The advantage is demonstrated for MVP's mol-cs ranking view on MassSpecGym; other annotation tools were not evaluated under this aggregate-vs-rank comparison
- The MVP README's preprocessed MassSpecGym download commands have empty wget URLs, so preprocessed files may need to be regenerated via data_preprocess.py or obtained elsewhere

## Evidence

- [abstract] On the MassSpecGym benchmark, annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation.: "annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation"
- [intro] Two inference strategies compared: aggregate-then-rank vs rank-then-aggregate.: "We evaluate two strategies for leveraging multiple spectra per molecule: an aggregate-then-rank strategy, which combines spectral information prior to candidate ranking"
- [results] Evaluation restricted to test molecules with exactly three spectra.: "we limit the evaluation to a subset of the test set for which a test molecule has exactly three spectra"
- [abstract] Headline rank@1 numbers for both candidate settings.: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [abstract] Baseline rank@1 numbers for ranking with individual spectra.: "achieving 26.37% and 11.10% rank@1 for candidates by mass and formula, respectively"
- [readme] MVP's stated usage for candidate ranking given a spectrum.: "MVP can be used to rank a set of molecular candidates given a spectrum."
