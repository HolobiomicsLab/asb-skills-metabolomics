---
name: false-positive-rate-assessment
description: Decision support for selecting an MS1/MS2 mass defect tolerance (mmu
  range) before MDRB matching of unpredictable metabolites to prototypes, when neither
  coverage nor false-positive burden can be assumed a priori.
when_to_use_negative:
- Metabolites are predictable via Metworks 1.3.0.200 metabolic responses, i.e. they
  are handled by predictable-pathway matching rather than MDRB's unpredictable-metabolite
  mass defect bridging.
- A mass defect range has already been validated for the dataset and only routine
  MDRB matching is needed — re-running the ±10/±25/±50/±75 sweep is redundant.
- Only MS2 fragment similarity is available (e.g. MTSF-style single-metabolite matching),
  since mass defect range optimization requires MS1/MS2 mass defect data.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging platform used to run the matching
    procedure at each mass defect range and tabulate coverage/positivity/false-positive
    metrics
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Produces the prototype database and unpredictable-metabolite feature table
    that feed the MDRB comparison runs
- name: R
  role: Implementation language of the MDRB post-processing platform in which the
    range sweep is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:277f3aad6e5bc285ee812534c739d1d622c153d361fba523b075cfcfb89f5ddf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# false-positive-rate-assessment

## Summary

Empirically benchmarks MS1/MS2 mass defect tolerance windows (±10, ±25, ±50, ±75 mmu) in the R-based MDRB metabolite-prototype bridging workflow, selecting the window that best balances pathway coverage, positivity rate, and false-positive bridged pairs for unpredictable metabolite matching.

## When to use

Decision support for selecting an MS1/MS2 mass defect tolerance (mmu range) before MDRB matching of unpredictable metabolites to prototypes, when neither coverage nor false-positive burden can be assumed a priori.

## When NOT to use

- Metabolites are predictable via Metworks 1.3.0.200 metabolic responses, i.e. they are handled by predictable-pathway matching rather than MDRB's unpredictable-metabolite mass defect bridging.
- A mass defect range has already been validated for the dataset and only routine MDRB matching is needed — re-running the ±10/±25/±50/±75 sweep is redundant.
- Only MS2 fragment similarity is available (e.g. MTSF-style single-metabolite matching), since mass defect range optimization requires MS1/MS2 mass defect data.

## Inputs

- Prototype database produced by PATBS
- Unpredictable-metabolite feature table produced by PATBS
- MDRB scoring configuration (mass defect ranges ±10/±25/±50/±75 mmu)

## Outputs

- Per-range metrics table (pathway coverage, positivity rate, false-positive bridged pairs for each mmu window)
- mass_defect_optimization.csv containing per-range metrics and the selected optimal range (±25 mmu)

## How to apply

Load the PATBS-derived prototype database and unpredictable-metabolite feature table into the R-based MDRB post-processing platform, then score metabolite-prototype pairs by sequentially integrating m/z, MS1/MS2 mass defect, and MS2 fragment similarity. Run the matching four times with the mass defect range set to ±10, ±25, ±50, and ±75 mmu. For each run, tabulate covered metabolic pathways, positivity rate, and the number of false-positive bridged pairs. Compare the configurations and select the range that balances the three metrics; in the Semen Armeniacae Amarum case study, ±25 mmu was optimal because ±10 mmu reduced pathway coverage while ±50/±75 mmu raised false positives. Write per-range metrics and the chosen optimum (±25 mmu) to mass_defect_optimization.csv.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging platform used to run the matching procedure at each mass defect range and tabulate coverage/positivity/false-positive metrics) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Produces the prototype database and unpredictable-metabolite feature table that feed the MDRB comparison runs)
- **R** (Implementation language of the MDRB post-processing platform in which the range sweep is executed)

## Examples

```
write.csv(per_range_metrics, "mass_defect_optimization.csv", row.names = FALSE)
```

## Evaluation signals

- Four matching runs are reported, one per mass defect window: ±10, ±25, ±50, ±75 mmu.
- Each run yields three tabulated metrics: pathway coverage, positivity rate, and number of false-positive bridged pairs.
- The selected optimum (±25 mmu) is justified by the trade-off: narrower windows reduce covered metabolic pathways, wider windows raise false positives.
- The output file mass_defect_optimization.csv contains both per-range metrics and the written selected optimum.
- The sweep applies to MS1 and MS2 mass defect dimensions, not only to MS2 fragments.

## Limitations

- No statistical validation, replicate counts, or quantitative performance metrics are reported for the overall strategy, so the ±25 mmu choice rests on the comparative sweep rather than formal validation.
- The optimum is dataset-dependent: a narrower mass defect range reduces covered metabolic pathways while a wider one raises false positives, so ±25 mmu must be re-checked for other herbs or instruments.
- No software versions are stated for the MDRB platform, limiting reproducibility of the range comparison.

## Evidence

- [intro] Mass defect of MS2 fragments of prototypes and metabolites is within a certain range, providing critical information for bridging prototypes and metabolites: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging the prototypes and the metabolites."
- [abstract] Three dimensions of mass spectral data are sequentially analyzed in MDRB scoring: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments, which was sequentially analyzed"
- [abstract] MDRB scoring is based on structural similarity and minimal mass defect variations: "We developed a scoring approach based on the structural similarity and minimal mass defect variations between metabolites and prototypes."
- [discussion] Absence of quantitative performance metrics limits validation of the strategy: [evidence span withheld — anchor status: not_found]
