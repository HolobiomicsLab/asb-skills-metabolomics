---
name: benchmark-reproduction-against-reported-metrics
description: Use when an article reports headline accuracy metrics (e.g., AvRE) for
  a method on named experiments and the deposited spectra/data are available, and
  the task is to reproduce those numbers and compare against reported baselines.
when_to_use_negative:
- The task asks to reproduce qualitative claims or plots rather than the paper's quantitative
  error metrics — instead consider extracting findings directly.
- The data deposit does not contain the input spectra or library files needed to rerun
  the tool, so metrics cannot be recomputed from source data.
- The metric depends on configuration the paper does not document (e.g., unstated
  κmixture/κcomponents values), making exact reproduction unreliable.
edam_topics:
- http://edamontology.org/topic_0592
- http://edamontology.org/topic_3360
tools:
- name: Magnetstein
  role: 'Quantification method under test: treats spectra as probability distributions
    of signal intensity, solves the optimal transport problem, and returns estimated
    relative concentrations pi used to recompute AvRE per experiment.'
  repo: https://gitlab.mimuw.edu.pl/ag181044/masserfront
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c03594
    title: 'Magnetstein: An Open-Source Tool for Quantitative NMR Mixture Analysis
      Robust to Low Resolution, Distorted Lineshapes, and Peak Shifts'
  source_content_hash: sha256:9aebebe2011211f6260086013c1d339d6223f40c01f38ea75dc71566782f48d9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ElsevierSoftwareX/SOFTX-D-25-00358
  tools:
  - Magnetstein
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0592
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# benchmark-reproduction-against-reported-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce an article's reported quantitative benchmarks by re-running the authors' method on the deposited experimental data and recomputing their error metrics, as demonstrated by recovering Magnetstein's per-experiment AvRE values (0.0%, 3.0%, 9.0%, 3.6%, 3.6%) for Experiments 1–5 and tabulating them against MANIQ and ACD/TP baselines.

## When to use

Use when an article reports headline accuracy metrics (e.g., AvRE) for a method on named experiments and the deposited spectra/data are available, and the task is to reproduce those numbers and compare against reported baselines.

## When NOT to use

- The task asks to reproduce qualitative claims or plots rather than the paper's quantitative error metrics — instead consider extracting findings directly.
- The data deposit does not contain the input spectra or library files needed to rerun the tool, so metrics cannot be recomputed from source data.
- The metric depends on configuration the paper does not document (e.g., unstated κmixture/κcomponents values), making exact reproduction unreliable.

## Inputs

- Deposited 1H NMR mixture spectra for Experiments 1–5 from the article's data deposit
- Component library spectra matching each experiment's compounds
- Known gravimetric compositions for the mixtures
- Article-reported benchmark values for the target method and baselines (MANIQ, ACD/Spectrus-Target-Protected Profiling)

## Outputs

- Per-experiment reproduced error metric (AvRE) for the target method
- Comparison table of reproduced vs. reported AvRE for Magnetstein, MANIQ, and ACD/TP (e.g., avre_comparison.csv)

## How to apply

Load the deposited input data (here, the 1H NMR mixture spectra and matching component library spectra for Experiments 1–5) and run the article's tool under the settings it describes — by default all programs run with default settings, with Magnetstein's tunable parameters κmixture and κcomponents documented. Apply any per-experiment configuration the paper states, e.g., for the low-field benchtop data of Experiment 3, set κcomponents higher than κmixture since 'the best results are obtained when κcomponents is higher than κmixture'. Extract the tool's estimated relative concentrations pi and recompute the paper's metric (average relative error, AvRE) against known compositions. Assemble reproduced values for the target method alongside the paper-reported baseline metrics (MANIQ: 12.2, 1.0, 28.8, 34.9, 7.2%; ACD-TP: 18.3, 6.9, 56.9, 39.7, 35.8%) into a single comparison table and validate each reproduced value against the article's reported result within reasonable numerical tolerance.

## Related tools

- **Magnetstein** (Quantification method under test: treats spectra as probability distributions of signal intensity, solves the optimal transport problem, and returns estimated relative concentrations pi used to recompute AvRE per experiment.) — https://gitlab.mimuw.edu.pl/ag181044/masserfront

## Evaluation signals

- Reproduced Magnetstein AvRE per experiment matches the article's reported values within reasonable numerical tolerance: Experiment 1 ≈ 0.0%, Experiment 2 ≈ 3.0%, Experiment 3 ≈ 9.0%, Experiment 4 ≈ 3.6%, Experiment 5 ≈ 3.6%.
- The comparison table contains all three methods (Magnetstein, MANIQ, ACD/TP) for all five experiments, with baseline values matching the reported ones (e.g., Experiment 3: MANIQ 28.8%, ACD-TP 56.9%).
- Recomputed AvRE is derived from Magnetstein's estimated relative concentrations pi checked against known gravimetric compositions, not copied directly from the paper.
- Experiment 3's configuration uses κcomponents higher than κmixture, consistent with the paper's stated best-result setting.
- Per-component relative errors for Experiment 3 fall in the reported ranges (e.g., Magnetstein RE α-pinene 0.0%, limonene 33.3%).

## Limitations

- Performance is sensitive to the tunable parameters κmixture and κcomponents; for small κmixture, 'the method tends to put the smaller component into vortex ω, causing a large relative error'.
- For Experiment 2, the default κmixture includes contaminating (degradation) peaks; performance improves when κmixture is lowered from the default, which complicates exact reproduction.
- Reproduced AvRE matches reported values only within reasonable numerical tolerance; the deposited data and stated settings must be available, and the low-field Experiment 3 requires the paper's specific κcomponents > κmixture configuration.
- The referenced repo (ElsevierSoftwareX/SOFTX-D-25-00358) hosts MasserFront, a UI for Masserstein mass spectrometry analysis — not the Magnetstein tool itself; no Magnetstein repository URL, version number, or Zenodo DOI is provided in the article, so tool provenance for exact reruns is incomplete.

## Evidence

- [results] Magnetstein still performs well with low-resolution spectrum: "Magnetstein, on the other hand, still performs well (AvRE = 9.0%; RE for α-pinene = 0.0%; RE for limonene = 33.3%)"
- [results] κcomponents tuning rationale for Experiment 3: "Note that the best results are obtained when κcomponents is higher than κmixture."
- [results] Default-settings caveat and tunable parameters: "Unless otherwise stated, all programs have been run with the default settings. Later, we discuss how and when the Magnetstein parameters (κmixture and κcomponents) can be tuned"
- [results] Experiment 5 outperformance summary: "Still, Magnetstein achieves a high trueness (AvRE = 3.6%) and outperforms other methods (AvRE = 35.8, 7.2% for ACD/TP, MANIQ)"
- [readme] Repo content mismatch (MasserFront for mass spectrometry): "MasserFront is a React and Flask-based UI application for Masserstein mass spectrometry result analysis."
