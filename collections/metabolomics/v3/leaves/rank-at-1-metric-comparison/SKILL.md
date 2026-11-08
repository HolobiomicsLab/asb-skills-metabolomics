---
name: rank-at-1-metric-comparison
description: Decision support for comparing candidate-ranking tools (e.g. JESTR vs
  SIRIUS vs CFM-ID) when each tool outputs a ranked candidate list per query spectrum
  and you need a single, interpretable top-1 accuracy figure plus a percent-improvement
  delta on matched test subsets.
when_to_use_negative:
- The task is to evaluate ranking quality beyond the top hit — use the full rank@[1-20]
  sweep instead, since rank@1 ignores deeper positions where the paper reports 55.5%–302.6%
  gains.
- Tools are evaluated on different query subsets or candidate pools — percent improvement
  is only meaningful on matched subsets, e.g. filtered MassSpecGym subsets excluding
  molecules seen in SIRIUS/CFM-ID pre-training.
- A stochastic tool's output varies across runs and only a single run is available
  — medians over repeated runs are needed for stable percent-improvement figures.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: JESTR
  role: Produces the evaluated ranked candidate lists via cosine similarity between
    query spectrum and candidate molecule embeddings; its rank@1 is the numerator
    of the improvement calculation.
  repo: https://github.com/HassounLab/JESTR1/
- name: SIRIUS
  role: Baseline ranking tool whose pretrained-model rank@1 (10.41% on the filtered
    subset) is compared against JESTR's to compute the 31.12% improvement.
- name: CFM-ID
  role: Baseline ranking tool whose pretrained-model rank@1 (2.00% on the filtered
    subset) is compared against JESTR's median 6.76% to compute the 238% improvement.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf354
    title: 'JESTR: Joint Embedding Space Technique for Ranking candidate molecules
      for the annotation of untargeted metabolomics data'
  source_content_hash: sha256:7ea1e37e7a1c48848a059380997e840aaf331c61f672c99121e98317a63723ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/JESTR1
  tools:
  - CFM-ID
  - JESTR
  - Python
  - SIRIUS
  - params.yaml
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rank-at-1-metric-comparison

## Summary

Compute and compare rank@1 (top-1 candidate ranking accuracy) across annotation tools on the same query spectra subsets, then express tool differences as percent improvement. This is the core metric used by JESTR to demonstrate gains over SIRIUS and CFM-ID on MassSpecGym subsets.

## When to use

Decision support for comparing candidate-ranking tools (e.g. JESTR vs SIRIUS vs CFM-ID) when each tool outputs a ranked candidate list per query spectrum and you need a single, interpretable top-1 accuracy figure plus a percent-improvement delta on matched test subsets.

## When NOT to use

- The task is to evaluate ranking quality beyond the top hit — use the full rank@[1-20] sweep instead, since rank@1 ignores deeper positions where the paper reports 55.5%–302.6% gains.
- Tools are evaluated on different query subsets or candidate pools — percent improvement is only meaningful on matched subsets, e.g. filtered MassSpecGym subsets excluding molecules seen in SIRIUS/CFM-ID pre-training.
- A stochastic tool's output varies across runs and only a single run is available — medians over repeated runs are needed for stable percent-improvement figures.

## Inputs

- Per-tool ranked candidate lists for a shared set of query spectra (e.g. MassSpecGym test subsets)
- Known ground-truth molecule (InChIKey/SMILES) for each query spectrum
- Optional: baseline_comparison.csv template for saving the per-subset rank@1 table

## Outputs

- Per-tool rank@1 values on each test subset
- Percent improvement of the evaluated tool over each baseline (e.g. +31.12% vs SIRIUS, +238.00% vs CFM-ID)
- A comparison table (e.g. baseline_comparison.csv) with rank@1 entries for all tools on the filtered subsets

## How to apply

Collect, for every query spectrum in the test subset, the ranked candidate list from each tool being compared (JESTR via cosine similarity in its joint embedding space; SIRIUS and CFM-ID via their pretrained models). Rank@1 is the fraction of queries whose correct target molecule appears at position 1 in the ranked list. Compute this fraction per tool per subset, then express the difference as a percent improvement: (JESTR_rank@1 − baseline_rank@1) / baseline_rank@1 × 100. On the filtered MassSpecGym subsets, JESTR reports 13.65% vs SIRIUS's 10.41% (31.12% improvement) and a median 6.76% vs CFM-ID's 2.00% (238% improvement), so both absolute and relative figures should be saved in a comparison table (e.g. baseline_comparison.csv) for verification against reported values. Because CFM-ID's rank@1 can be very low, report the median over repeated runs when tool runs are stochastic rather than a single point estimate.

## Related tools

- **JESTR** (Produces the evaluated ranked candidate lists via cosine similarity between query spectrum and candidate molecule embeddings; its rank@1 is the numerator of the improvement calculation.) — https://github.com/HassounLab/JESTR1/
- **SIRIUS** (Baseline ranking tool whose pretrained-model rank@1 (10.41% on the filtered subset) is compared against JESTR's to compute the 31.12% improvement.)
- **CFM-ID** (Baseline ranking tool whose pretrained-model rank@1 (2.00% on the filtered subset) is compared against JESTR's median 6.76% to compute the 238% improvement.)

## Examples

```
python cand_rank_canopus.py
```

## Evaluation signals

- JESTR's rank@1 on the filtered MassSpecGym subset excluding SIRIUS pre-training molecules must be ~13.65% while SIRIUS achieves ~10.41%, yielding a ~31% improvement.
- On the subset excluding CFM-ID pre-training molecules, CFM-ID's rank@1 should be ~2.00% and JESTR's median ~6.76%, yielding a ~238% improvement.
- The output CSV (e.g. baseline_comparison.csv) must contain a rank@1 entry for every tool (JESTR, SIRIUS, CFM-ID) on each filtered subset.
- Percent-improvement values must be computed on identical query subsets — verify that the denominator and numerator come from the same filtered MassSpecGym subset, not the unfiltered set.
- When CFM-ID or another tool is run multiple times, report the median rank@1, not a single-run value, to stabilize the improvement figure.

## Limitations

- Rank@1 alone under-states JESTR's advantage; the paper reports larger relative gains for rank@[1-20] (55.5%–302.6%), so single-metric comparison can mislead tool selection.
- Percent improvement is highly sensitive to a low baseline (CFM-ID's 2.00% rank@1 inflates the relative gain to 238% even though the absolute difference is only ~4.76 percentage points).
- Comparisons are only valid on subsets that exclude molecules seen in each baseline tool's pre-training; using the unfiltered MassSpecGym set would overstate or understate gains.
- JESTR weights were trained and tested on nVidia A100 with CUDA 11.8; ranking results may vary on different hardware.

## Evidence

- [abstract] JESTR outperforms SIRIUS by 31% and CFM-ID by 238%: [evidence span withheld — anchor status: not_found]
- [methods] MassSpecGym subsets filtering: "appropriate subsets of MassSpecGym dataset"
- [abstract] JESTR cosine-similarity ranking: "JESTR leverages the insight that molecules and their corresponding spectra are views of the same data and effectively embeds their representations in a joint space."
- [readme] Ranking via pretrained notebook: "To use the pretrained model and rank a target molecule against its candidates on a given spectrum, please run the code in the notebook JESTR.ipynb."
- [methods] Candidate ranking CLI: "To rank candidates for the complete NPLIB1 dataset, use the command: python cand_rank_canopus.py."
