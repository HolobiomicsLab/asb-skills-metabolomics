---
name: dehp-biotransformation-spot-check
description: Use when a BioTransformer metabolite-prediction run over a large chemical
  list (e.g., the ~20,061-chemical human exposome set) needs a positive-control spot-check
  before trusting aggregate per-module metabolite counts.
when_to_use_negative:
- When validating predictions for chemicals that, like DEHP, are outside the run's
  scope — e.g., chemicals with MW >900 Da or nonmatched chemical categories that the
  main workflow already excluded (4,225 chemicals were excluded on these grounds).
- When the goal is to benchmark BioTransformer's prediction accuracy in general —
  the article notes BioTransformer cannot predict all metabolites and results may
  be inaccurate for some chemicals, so this spot-check only validates setup, not comprehensive
  coverage.
- When the analysis target is biotransformation half-life (HLB) estimation via the
  Iterative Fragment Selection approach rather than metabolite structure prediction.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0605
tools:
- name: BioTransformer
  role: Machine-learning tool whose EC-based, CYP450, phase II, and human gut microbial
    modules are run on DEHP to generate the metabolite list compared against the published
    GUI example
- name: Java
  role: Runtime environment for executing BioTransformer and its GUI-based example
    workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:517caa967db246b72d05322c33650e4c3e40675fa9d81e9202d37baa0adbbe46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - Java
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# DEHP biotransformation spot-check

## Summary

A validation routine that reproduces the BioTransformer GUI example for DEHP by running the EC-based, CYP450, phase II, and human gut microbial prediction modules on DEHP and comparing the output metabolite list against the published GUI example, confirming the in-silico biotransformation setup behaves as expected.

## When to use

Use when a BioTransformer metabolite-prediction run over a large chemical list (e.g., the ~20,061-chemical human exposome set) needs a positive-control spot-check before trusting aggregate per-module metabolite counts.

## When NOT to use

- When validating predictions for chemicals that, like DEHP, are outside the run's scope — e.g., chemicals with MW >900 Da or nonmatched chemical categories that the main workflow already excluded (4,225 chemicals were excluded on these grounds).
- When the goal is to benchmark BioTransformer's prediction accuracy in general — the article notes BioTransformer cannot predict all metabolites and results may be inaccurate for some chemicals, so this spot-check only validates setup, not comprehensive coverage.
- When the analysis target is biotransformation half-life (HLB) estimation via the Iterative Fragment Selection approach rather than metabolite structure prediction.

## Inputs

- DEHP chemical structure (SMILES or equivalent) used in the BioTransformer GUI example
- Published BioTransformer GUI example metabolite list for DEHP (reference output)
- BioTransformer installation (Java) configured with EC-based, CYP450, phase II, and human gut microbial modules

## Outputs

- Predicted DEHP biotransformation metabolite list from the local BioTransformer run
- Comparison result (match/mismatch) between local predictions and the published GUI example metabolites
- Go/no-go signal for accepting the large-scale metabolite prediction counts

## How to apply

Retrieve the BioTransformer GUI example for DEHP, then run the same prediction modules (EC-based, CYP450 phase I, phase II, and human gut microbial) on the DEHP structure via BioTransformer (Java-based). Deduplicate identical predicted structures and compare the resulting metabolite list against the metabolites reported in the published GUI example. Treat the spot-check as a gate on the full-scale run: only after DEHP reproduces the expected metabolites should the aggregate counts for the 20,061 chemicals (95,976 total; 19,212 EC-based; 72,193 CYP450; 15,762 phase II; 6,337 gut microbial) be accepted as correctly generated. The rationale is that BioTransformer's machine-learning predictions can be inaccurate for some chemicals, so a single well-characterized compound with a vendor-documented example output anchors the pipeline configuration.

## Related tools

- **BioTransformer** (Machine-learning tool whose EC-based, CYP450, phase II, and human gut microbial modules are run on DEHP to generate the metabolite list compared against the published GUI example)
- **Java** (Runtime environment for executing BioTransformer and its GUI-based example workflow)

## Examples

```
java -jar biotransformer.jar -q DEHP.smiles -b all --ecbased --cyp450 --phaseii --gutmicro
```

## Evaluation signals

- The DEHP metabolite list produced locally must reproduce the metabolites reported in the BioTransformer GUI example for DEHP.
- The same module configuration (EC-based, CYP450, phase II, human gut microbial) must be used as in the GUI example.
- Aggregate counts from the full run must match the reported totals (95,976 total; 19,212 EC-based; 72,193 CYP450; 15,762 phase II; 6,337 gut microbial) from 20,061 chemicals.
- Per-module counts must be recorded individually so a mismatch in one module does not silently pass as a correct total.
- Deduplication of identical metabolite structures must be applied before comparing counts.

## Limitations

- BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals (article discussion).
- The spot-check validates only DEHP; agreement on DEHP does not guarantee accuracy for other chemical categories.
- Chemicals with MW >900 Da or nonmatched chemical categories were excluded from the parent run, so the spot-check does not speak to those exclusions.
- A successful DEHP comparison is a necessary but not sufficient condition — full-run validation also requires the aggregate per-module counts to match the reported values.

## Evidence

- [other] Retrieve the BioTransformer GUI example for DEHP, run the same prediction modules on DEHP, and compare the output metabolite list against the published GUI example: [evidence span withheld — anchor status: not_found]
- [discussion] BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals: "BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals"
- [abstract] biotransformation products were predicted using the previously developed BioTransformer machine-learning method: "biotransformation products were predicted using the previously developed BioTransformer machine-learning method"
- [other] BioTransformer predicted 95,976 metabolites from 20,061 chemicals: [evidence span withheld — anchor status: not_found]
