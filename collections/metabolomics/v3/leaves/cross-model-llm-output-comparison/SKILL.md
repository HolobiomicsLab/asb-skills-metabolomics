---
name: cross-model-llm-output-comparison
description: Use when you have a set of m/z values from metabolomics peaks and need
  to determine which LLM(s) correctly annotate them, by running the same peak-identification
  prompt across multiple models and tabulating per-model successes and failures.
when_to_use_negative:
- You need definitive one-to-one compound identification — identical m/z values can
  arise from other metabolites, and the article notes direct one-to-one comparison
  of narcotics is not feasible without prior drug information
- The peaks lack MS/MS fragmentation data or reference standards — the article states
  additional experiments (MS/MS, standard comparison, NMR, chemical reaction validation)
  are essential to ascertain identities, so LLM output should not be treated as proof
- You need reproducible, scripted identification tied to a curated database — SMART
  2.0's own peak-identification module matching against HMDB and MassBank is the appropriate
  path instead
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: ChatGPT-4
  role: LLM model queried via API with the peak-identification prompts and m/z values;
    identified all 12 narcotics
- name: ChatGPT-3.5
  role: LLM model queried via API with the peak-identification prompts and m/z values;
    identified all 12 narcotics
- name: Llama3-70B
  role: Open-weight LLM model queried with the identical prompts and m/z values; identified
    all 12 narcotics
- name: Windows Copilot
  role: LLM model run with Precision Mode enabled; failed to identify delta9-THC and
    nimetazepam and misidentified FM2
- name: SMART 2.0
  role: R tool whose peak-identification module (m/z matching against HMDB and MassBank)
    provides the database-driven alternative to LLM-based annotation
  repo: https://github.com/YuJenL/SMART
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:7304321ef8f564272d14d59d45ca71ecd43151c8be87d9e5c73a2e5d4d0dae1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - ChatGPT-3.5
  - ChatGPT-4
  - Llama3-70B
  - SMART 2.0
  - Windows Copilot
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-model-llm-output-comparison

## Summary

Benchmark large language models (ChatGPT-4, ChatGPT-3.5, Llama3-70B, Windows Copilot) on the same peak-annotation task by submitting identical m/z-based prompts and comparing which metabolites each model assigns. Used to assess whether LLMs can serve as an auxiliary peak-identification aid in metabolomics.

## When to use

Use when you have a set of m/z values from metabolomics peaks and need to determine which LLM(s) correctly annotate them, by running the same peak-identification prompt across multiple models and tabulating per-model successes and failures.

## When NOT to use

- You need definitive one-to-one compound identification — identical m/z values can arise from other metabolites, and the article notes direct one-to-one comparison of narcotics is not feasible without prior drug information
- The peaks lack MS/MS fragmentation data or reference standards — the article states additional experiments (MS/MS, standard comparison, NMR, chemical reaction validation) are essential to ascertain identities, so LLM output should not be treated as proof
- You need reproducible, scripted identification tied to a curated database — SMART 2.0's own peak-identification module matching against HMDB and MassBank is the appropriate path instead

## Inputs

- List of m/z values for peaks requiring identification (e.g. 12 narcotics m/z values from the SMART 2.0 narcotics example dataset)
- Peak-identification prompts from Supporting Information Text S3
- Ground-truth identity for each m/z (known narcotic names) for validation

## Outputs

- llm_peak_identification.csv with columns for model, m/z, and identified compound
- Per-model annotation comparison table showing which m/z values each model identified or failed to assign

## How to apply

Extract the target m/z values (e.g. the 12 narcotics m/z values from the SMART 2.0 narcotics example dataset) and load the documented peak-identification prompts from Supporting Information Text S3. Submit the identical prompts plus m/z values through each model's interface — ChatGPT-4 and ChatGPT-3.5 via their API, Llama3-70B via its model endpoint, and Windows Copilot with Precision Mode enabled. Parse each model's natural-language response and record the assigned metabolite identity (or an explicit failure-to-assign) for every m/z value into a tabular file such as llm_peak_identification.csv with columns for model, m/z, and identified compound. Validate by comparing the compiled table against known ground truth: in the reference run, ChatGPT-4, ChatGPT-3.5, and Llama3-70B identified all 12 narcotics while Windows Copilot missed delta9-THC and nimetazepam and incorrectly suggested alternative compounds for FM2.

## Related tools

- **ChatGPT-4** (LLM model queried via API with the peak-identification prompts and m/z values; identified all 12 narcotics)
- **ChatGPT-3.5** (LLM model queried via API with the peak-identification prompts and m/z values; identified all 12 narcotics)
- **Llama3-70B** (Open-weight LLM model queried with the identical prompts and m/z values; identified all 12 narcotics)
- **Windows Copilot** (LLM model run with Precision Mode enabled; failed to identify delta9-THC and nimetazepam and misidentified FM2)
- **SMART 2.0** (R tool whose peak-identification module (m/z matching against HMDB and MassBank) provides the database-driven alternative to LLM-based annotation) — https://github.com/YuJenL/SMART

## Examples

```
pandas.read_csv('llm_peak_identification.csv')  # columns: model, m/z, identified_compound
```

## Evaluation signals

- Compiled llm_peak_identification.csv matches the reported outcome: ChatGPT-4, ChatGPT-3.5, and Llama3-70B identify all 12 narcotics, Copilot misses delta9-THC and nimetazepam and misidentifies FM2
- Every row in the output table has a non-empty entry in the model, m/z, and identified-compound columns (or an explicit failure marker), with no silent omissions
- All models were given the byte-identical prompt text and m/z list, verified against the prompts loaded from Supporting Information Text S3
- Copilot runs confirm Precision Mode was enabled, since that setting affects annotation behaviour
- Identified compounds are checked against known ground-truth names for each m/z rather than accepted at face value

## Limitations

- Windows Copilot failed to identify delta9-THC and nimetazepam and incorrectly suggested alternative compounds for FM2, showing per-model variance in annotation reliability
- One-to-one identification of narcotics without prior drug information is not feasible because other metabolites can share identical m/z values
- LLM results were only described as promising — the authors state they plan to integrate AI methodologies into SMART as an auxiliary tool, implying LLM output is not yet a substitute for database matching or experimental validation (MS/MS, NMR, standards)
- Heroin and nimetazepam are absent from HMDB, and even SMART 2.0 cannot detect them, so no reference annotation exists for those m/z values

## Evidence

- [other] ChatGPT-4, ChatGPT-3.5, and Llama3-70B correctly identified all 12 narcotics, whereas Windows Copilot failed to identify delta9-THC and nimetazepam and incorrectly identified FM2: [evidence span withheld — anchor status: not_found]
- [supplementary] load the documented peak-identification prompts from Supporting Information Text S3: [evidence span withheld — anchor status: not_found]
- [other] compile per-model assignments into a tabular output: [evidence span withheld — anchor status: not_found]
- [discussion] direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values: "direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values"
- [abstract] We also explored the potential of using large language models for peak annotation and have found the results to be promising: "We also explored the potential of using large language models for peak annotation and have found the results to be promising"
- [discussion] additional experiments such as MS/MS analysis, standard reference comparison, nuclear magnetic resonance (NMR) spectroscopy, and chemical reaction validation are essential to ascertain the identities: "additional experiments such as MS/MS analysis, standard reference comparison, nuclear magnetic resonance (NMR) spectroscopy, and chemical reaction validation are essential to ascertain the identities"
- [discussion] Our plans include integrating AI methodologies into SMART as an auxiliary tool to enhance its capabilities: "Our plans include integrating AI methodologies into SMART as an auxiliary tool to enhance its capabilities"
- [discussion] Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
