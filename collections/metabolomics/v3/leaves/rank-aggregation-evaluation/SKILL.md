---
name: rank-aggregation-evaluation
description: Use when a molecule has multiple query spectra (e.g. three collision-energy
  levels) and you must choose between consensus-spectrum annotation versus aggregating
  per-spectrum candidate rankings, with rank@1 as the decision metric.
when_to_use_negative:
- Test molecules with fewer or more than exactly three spectra — the article limits
  this comparison to 'a subset of the test set for which a test molecule has exactly
  three spectra', so applying it to molecules with 1–2 or 4+ spectra extrapolates
  beyond the evidence.
- Single-spectrum annotation tasks where no aggregation decision exists — there is
  nothing to aggregate and the consensus-vs-ranking comparison is moot.
- Spectrum simulation or de novo generation challenges in MassSpecGym — this skill
  only addresses the molecule retrieval task's ranking aggregation choice.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MVP
  role: 'Provides the candidate-ranking model and both ranking views: the mol-cs consensus-spectrum
    view (aggregate-then-rank) and per-spectrum rankings aggregated via average rank
    / reciprocal rank (rank-then-aggregate); invoked through test.py with a params
    YAML.'
  repo: https://github.com/HassounLab/MVP
- name: MassSpecGym
  role: Supplies the benchmark dataset, candidate pools by mass and formula, the predefined
    train/val/test splits, and the rank@k evaluation metrics used to compare the two
    aggregation strategies.
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: lightning
  role: Underpins MVP's model/data-module interface (MassSpecDataModule / LightningModule
    subclasses) that loads the pretrained checkpoint and runs the ranking evaluation
    loop.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:52edd22d885f0b45cf7e95aed340d4658f69a09ee7eeca2defcfa4d379587867
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/MVP
  tools:
  - MVP
  - MassSpecGym
  - lightning
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rank-aggregation-evaluation

## Summary

Benchmarks whether annotating a molecule with multiple MS/MS spectra via an aggregate-then-rank consensus spectrum outperforms rank-then-aggregate strategies (average rank / reciprocal rank) on the MassSpecGym test set, using MVP's candidate-ranking views and rank@1 metrics.

## When to use

Use when a molecule has multiple query spectra (e.g. three collision-energy levels) and you must choose between consensus-spectrum annotation versus aggregating per-spectrum candidate rankings, with rank@1 as the decision metric.

## When NOT to use

- Test molecules with fewer or more than exactly three spectra — the article limits this comparison to 'a subset of the test set for which a test molecule has exactly three spectra', so applying it to molecules with 1–2 or 4+ spectra extrapolates beyond the evidence.
- Single-spectrum annotation tasks where no aggregation decision exists — there is nothing to aggregate and the consensus-vs-ranking comparison is moot.
- Spectrum simulation or de novo generation challenges in MassSpecGym — this skill only addresses the molecule retrieval task's ranking aggregation choice.

## Inputs

- MassSpecGym test-set spectra for molecules with exactly three constituent spectra (per-spectrum MS/MS, formSpec or binnedSpec representation)
- Consensus spectra constructed per molecule from the constituent spectra
- Candidate molecule lists by mass and by formula (MassSpecGym retrieval candidate pools)
- Pretrained MVP checkpoint (trained on MassSpecGym) with params_formSpec.yaml or params_binnedSpec.yaml configuration

## Outputs

- Structured CSV comparison table reporting rank@1 (and rank@k) for aggregate-then-rank (mol-cs consensus spectrum) versus rank-then-aggregate (average rank, reciprocal rank), per candidate mode (mass, formula)
- Per-molecule ranking lists for both strategies, enabling downstream inspection of disagreement cases

## How to apply

Load the MassSpecGym test set and restrict to test molecules with exactly three constituent spectra (1,531 cases in the article). Run MVP's candidate ranking twice per molecule: once using the consensus-spectrum 'mol-cs' ranking view (aggregate-then-rank) with the pretrained checkpoint loaded via test.py and params_formSpec.yaml or params_binnedSpec.yaml, and once per individual constituent spectrum, then aggregate those per-spectrum rankings into a molecule-level ranking using average rank and reciprocal rank (rank-then-aggregate). Evaluate both candidate-retrieval modes — candidates by mass and candidates by formula — and compute rank@1 (and rank@k) for both strategies. The expected outcome, as reported in the article, is that the consensus-spectrum view wins: 35.99% vs 26.37% rank@1 for candidates by mass and 13.96% vs 11.10% for candidates by formula. Save the comparison as a structured CSV table keyed by strategy, candidate mode, and metric.

## Related tools

- **MVP** (Provides the candidate-ranking model and both ranking views: the mol-cs consensus-spectrum view (aggregate-then-rank) and per-spectrum rankings aggregated via average rank / reciprocal rank (rank-then-aggregate); invoked through test.py with a params YAML.) — https://github.com/HassounLab/MVP
- **MassSpecGym** (Supplies the benchmark dataset, candidate pools by mass and formula, the predefined train/val/test splits, and the rank@k evaluation metrics used to compare the two aggregation strategies.) — https://github.com/pluskal-lab/MassSpecGym
- **lightning** (Underpins MVP's model/data-module interface (MassSpecDataModule / LightningModule subclasses) that loads the pretrained checkpoint and runs the ranking evaluation loop.)

## Examples

```
python test.py --param_pth params_binnedSpec.yaml
```

## Evaluation signals

- The subset of test molecules evaluated must have exactly three spectra per molecule (1,531 cases); verify this count before comparing strategies.
- Expected quantitative result: aggregate-then-rank (mol-cs) achieves 35.99% rank@1 for candidates by mass and 13.96% for candidates by formula, versus 26.37% and 11.10% for rank-then-aggregate — the consensus spectrum should win in both candidate modes.
- Both candidate modes (mass-based and formula-based retrieval) must be evaluated; a comparison covering only one mode is incomplete relative to the article's Table 2 setup.
- The output CSV must distinguish strategies (consensus spectrum vs average rank vs reciprocal rank) and metrics (rank@1, rank@k) so the two aggregation families are directly comparable row-by-row.
- Check that per-spectrum rankings are aggregated into a single molecule-level ranking before computing rank@1 in the rank-then-aggregate arm — computing per-spectrum rank@1 and averaging that is a different (incorrect) metric.

## Limitations

- The evaluation is restricted to test molecules with exactly three spectra, limiting generalization to molecules with other numbers of constituent spectra.
- Aggregation strategies other than average rank and reciprocal rank were not examined, so the reported superiority of consensus spectra may not hold for all rank-aggregation schemes.
- MVP's representation quality could benefit from more accurate peak labeling and additional views, and losses that prioritize ranking molecular candidates are not currently included — both may affect the ranking comparison.
- Preprocessed MassSpecGym download URLs in the MVP README are empty, so users must preprocess the dataset themselves via data_preprocess.py before running the comparison.

## Evidence

- [abstract] On the MassSpecGym benchmark, annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation.: "annotation using query consensus spectra significantly outperforms rank aggregation strategies based on constituent spectrum annotation"
- [abstract] Using the consensus spectrum view, MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively; ranking using individual spectra achieves 26.37% and 11.10%.: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [results] The evaluation is limited to test molecules with exactly three spectra (1,531 cases).: "we limit the evaluation to a subset of the test set for which a test molecule has exactly three spectra"
- [intro] Two inference strategies are compared: aggregate-then-rank versus rank-then-aggregate.: "We evaluate two strategies for leveraging multiple spectra per molecule: an aggregate-then-rank strategy, which combines spectral information prior to candidate ranking, and a rank-then-aggregate"
- [readme] Pretrained MVP is invoked via test.py with a params YAML (params_formSpec.yaml or params_binnedSpec.yaml).: "python test.py --param_pth params_formSpec.yaml"
- [readme] MVP is trained on the MassSpecGym dataset, whose splits and metrics the comparison relies on.: "Our model is trained on [MassSpecGym dataset](https://github.com/pluskal-lab/MassSpecGym). Follow their instruction to download the spectra and candidate dataset."
