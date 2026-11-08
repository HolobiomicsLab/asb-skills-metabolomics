---
name: llm-prompt-based-metabolite-annotation
description: Use when a targeted or untargeted LC-MS peak table contains m/z features
  that HMDB/MassBank matching in SMART 2.0 cannot assign (e.g., heroin, nimetazepam),
  and database lookup fails for narcotics-like compounds.
when_to_use_negative:
- When the peak already has a confident database match via SMART 2.0's HMDB/MassBank
  peak identification — the LLM layer is an auxiliary tool for unmatched peaks, not
  a primary annotator.
- When MS/MS fragmentation data or authentic standards are available and required
  for definitive identification — the article states additional experiments such as
  MS/MS analysis, standard reference comparison, NMR spectroscopy, and chemical reaction
  validation are essential to ascertain identities.
- When strict one-to-one identification is needed without prior drug information,
  since identical m/z values from other metabolites make direct comparison infeasible.
tools:
- name: SMART 2.0
  role: produces the m/z peak table and complementary HMDB/MassBank peak-identification
    results that the LLM annotation layer supplements
  repo: https://github.com/YuJenL/SMART
- name: ChatGPT-4
  role: LLM chatbot queried with m/z lists to suggest candidate metabolite identities
- name: ChatGPT-3.5
  role: LLM chatbot queried with m/z lists to suggest candidate metabolite identities
- name: Llama3-70B
  role: open LLM chatbot queried with identical m/z prompts for cross-model comparison
- name: Windows Copilot
  role: LLM chatbot (Precision Mode) queried with identical prompts; benchmarked as
    the failure case
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:c62bba26aef9ad3d2c17ee43adbdd72b1b8f1d9576753b2c5bbac663d3f6f6b0
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# llm-prompt-based-metabolite-annotation

## Summary

Prompt large-language-model chatbots (ChatGPT-4, ChatGPT-3.5, Llama3-70B, Windows Copilot) with unassigned m/z values from untargeted LC-MS metabolomics peak tables to obtain candidate metabolite identities, benchmarked in SMART 2.0's peak-identification context on narcotics data.

## When to use

Use when a targeted or untargeted LC-MS peak table contains m/z features that HMDB/MassBank matching in SMART 2.0 cannot assign (e.g., heroin, nimetazepam), and database lookup fails for narcotics-like compounds.

## When NOT to use

- When the peak already has a confident database match via SMART 2.0's HMDB/MassBank peak identification — the LLM layer is an auxiliary tool for unmatched peaks, not a primary annotator.
- When MS/MS fragmentation data or authentic standards are available and required for definitive identification — the article states additional experiments such as MS/MS analysis, standard reference comparison, NMR spectroscopy, and chemical reaction validation are essential to ascertain identities.
- When strict one-to-one identification is needed without prior drug information, since identical m/z values from other metabolites make direct comparison infeasible.

## Inputs

- Untargeted or targeted LC-MS peak table with m/z values not matched by SMART 2.0's peak-identification module (HMDB/MassBank)
- Standardized peak-identification prompt text from SMART 2.0 Supporting Information Text S3
- List of m/z values for candidate annotation (e.g., the 12 narcotics m/z values from the narcotics example dataset)

## Outputs

- Flat CSV of per-model, per-m/z metabolite assignments (e.g., llm_peak_identification.csv with columns model, m/z, identified compound)
- Comparative annotation report showing which chatbots correctly identified all compounds versus which failed or hallucinated alternatives (e.g., Copilot missing delta9-THC and nimetazepam, misidentifying FM2)

## How to apply

1. Extract the unassigned m/z values (e.g., the 12 narcotics from the SMART 2.0 narcotics example) from the peak table produced by SMART 2.0's peak-analysis module. 2. Load the documented peak-identification prompts from Supporting Information Text S3 rather than writing ad-hoc prompts, so that results are reproducible across models. 3. Submit the identical prompt-plus-m/z list to each chatbot (ChatGPT-4 and ChatGPT-3.5 via API, Llama3-70B, and Windows Copilot with Precision Mode enabled). 4. Parse each model's natural-language response and record the assigned metabolite identity (or failure to assign) per m/z into a flat file such as llm_peak_identification.csv with columns for model, m/z, and identified compound. 5. Rationale: LLMs act as an auxiliary annotation layer on top of SMART 2.0's HMDB- and MassBank-based matching, compensating for compounds absent from those databases; they are not a replacement for MS/MS confirmation.

## Related tools

- **SMART 2.0** (produces the m/z peak table and complementary HMDB/MassBank peak-identification results that the LLM annotation layer supplements) — https://github.com/YuJenL/SMART
- **ChatGPT-4** (LLM chatbot queried with m/z lists to suggest candidate metabolite identities)
- **ChatGPT-3.5** (LLM chatbot queried with m/z lists to suggest candidate metabolite identities)
- **Llama3-70B** (open LLM chatbot queried with identical m/z prompts for cross-model comparison)
- **Windows Copilot** (LLM chatbot (Precision Mode) queried with identical prompts; benchmarked as the failure case)

## Examples

```
Rscript SMART_peak_annotation.R --peak_table narcotics_peaks.csv --mz_list 12_narcotics_mz.txt --prompt_file S3_peak_identification_prompt.txt --model chatgpt4 --output llm_peak_identification.csv
```

## Evaluation signals

- Compiled llm_peak_identification.csv should show ChatGPT-4, ChatGPT-3.5, and Llama3-70B correctly identifying all 12 narcotics m/z values.
- Windows Copilot should fail to identify delta9-THC and nimetazepam and incorrectly suggest alternative compounds for FM2 — deviations from this pattern indicate non-standard prompting.
- Every m/z value must appear exactly once per model in the output table (12 rows × 4 models = 48 rows) with a non-empty compound name or an explicit failure marker.
- Identical prompts and m/z lists must be submitted to each model; any difference in prompt text invalidates cross-model comparison.
- Annotations should be cross-checked against SMART 2.0's HMDB/MassBank results — compounds the databases cannot detect (heroin, nimetazepam) must still be reported by the successful LLMs.

## Limitations

- LLM outputs are promising but not definitive: the article recommends MS/MS analysis, standard reference comparison, NMR spectroscopy, and chemical reaction validation to ascertain identities.
- Direct one-to-one identification of narcotics is not feasible without prior drug information because other metabolites may share identical m/z values.
- AI methodologies are planned as an auxiliary tool integrated into SMART, not yet built in — results must be manually parsed from natural-language responses.
- The authors found results 'promising' but did not claim perfect specificity; failure modes (Copilot's misses and FM2 misidentification) show model-dependent reliability.
- Heroin and nimetazepam are not even encompassed by HMDB, so database back-fallback cannot confirm LLM annotations for those compounds.

## Evidence

- [abstract] We also explored the potential of using large language models for peak annotation and have found the results to be promising.: "We also explored the potential of using large language models for peak annotation and have found the results to be promising."
- [discussion] The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [discussion] Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
- [discussion] direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values: "direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values"
- [discussion] additional experiments such as MS/MS analysis, standard reference comparison, nuclear magnetic resonance (NMR) spectroscopy, and chemical reaction validation are essential to ascertain the identities: "additional experiments such as MS/MS analysis, standard reference comparison, nuclear magnetic resonance (NMR) spectroscopy, and chemical reaction validation are essential to ascertain the identities"
- [discussion] Our plans include integrating AI methodologies into SMART as an auxiliary tool to enhance its capabilities: "Our plans include integrating AI methodologies into SMART as an auxiliary tool to enhance its capabilities"
- [other] 1. Extract the 12 narcotics m/z values from the SMART 2.0 narcotics example dataset. 2. Load the documented peak-identification prompts: [evidence span withheld — anchor status: not_found]
