---
name: narcotics-mz-peak-identification
description: 'Use when metabolomics m/z peaks (e.g., the 12 SMART 2.0 narcotics: amphetamine,
  ketamine, cocaine, heroin, delta9-THC, nimetazepam, FM2) need candidate compound
  identities and database-only matching (HMDB/MassBank via SMART 2.'
when_to_use_negative:
- When the goal is definitive one-to-one compound identification from m/z alone —
  the paper notes identical m/z values from other metabolites make direct one-to-one
  comparison infeasible without prior drug information
- When the m/z list covers general metabolomics peaks already well-represented in
  HMDB/MassBank — SMART 2.0's built-in peak-identification module suffices without
  LLM involvement
- When MS/MS fragmentation, reference standards, NMR, or chemical-reaction validation
  data are available — the paper states these are the essential confirmation experiments,
  not LLM responses
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3398
tools:
- name: ChatGPT-4
  role: Primary LLM evaluated for identifying all 12 narcotics from m/z values; queried
    via its API with SI Text S3 prompts
- name: ChatGPT-3.5
  role: Secondary LLM queried via its API with identical prompts and m/z values
- name: Llama3-70B
  role: Open-weight LLM evaluated on the same m/z identification task
- name: Windows Copilot
  role: LLM evaluated with Precision Mode enabled; failed on delta9-THC, nimetazepam,
    and FM2
- name: SMART 2.0
  role: Source of the narcotics example dataset and of the HMDB/MassBank-based peak-identification
    module the LLM approach is compared against
  repo: https://github.com/YuJenL/SMART
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:bc37af3ed5b174d0867bb1d0d0a41ecce915395347f90f202da0c84ffc6d1ffd
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
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3398
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# narcotics-mz-peak-identification

## Summary

Evaluates whether large language model chatbots (ChatGPT-4, ChatGPT-3.5, Llama3-70B, Windows Copilot) can identify narcotics from m/z values alone, using the documented SMART 2.0 peak-identification prompts against the 12-narcotics example dataset. This addresses the paper's postanalysis peak-identification gap, where database matching against HMDB and MassBank fails for narcotics like heroin and nimetazepam.

## When to use

Use when metabolomics m/z peaks (e.g., the 12 SMART 2.0 narcotics: amphetamine, ketamine, cocaine, heroin, delta9-THC, nimetazepam, FM2) need candidate compound identities and database-only matching (HMDB/MassBank via SMART 2.0's peak-identification module) misses entries, prompting exploration of LLM-based peak annotation.

## When NOT to use

- When the goal is definitive one-to-one compound identification from m/z alone — the paper notes identical m/z values from other metabolites make direct one-to-one comparison infeasible without prior drug information
- When the m/z list covers general metabolomics peaks already well-represented in HMDB/MassBank — SMART 2.0's built-in peak-identification module suffices without LLM involvement
- When MS/MS fragmentation, reference standards, NMR, or chemical-reaction validation data are available — the paper states these are the essential confirmation experiments, not LLM responses

## Inputs

- List of m/z values from the SMART 2.0 narcotics example dataset (12 narcotics; e.g., amphetamine, ketamine, cocaine, heroin, delta9-THC, nimetazepam, FM2)
- Peak-identification prompt text from Supporting Information Text S3
- LLM access: ChatGPT-4 / ChatGPT-3.5 API, Llama3-70B endpoint, Windows Copilot (Precision Mode)

## Outputs

- llm_peak_identification.csv with columns model, m/z, and identified compound
- Per-model correctness table (identified vs. failed/misidentified narcotics)
- Comparison of LLM annotations against SMART 2.0's HMDB/MassBank-based peak identification output

## How to apply

First extract the 12 narcotics' m/z values from the SMART 2.0 narcotics example dataset (downloadable from https://github.com/YuJenL/SMART). Load the peak-identification prompts documented in Supporting Information Text S3. Submit identical prompts plus m/z values to each LLM: ChatGPT-4 and ChatGPT-3.5 via their API, Llama3-70B via its model endpoint, and Windows Copilot with Precision Mode enabled to reduce hallucination. Parse each model's natural-language response into a structured table (model, m/z, identified compound). Rationale: SMART 2.0's peak-identification module matches m/z against HMDB and MassBank, but these databases do not cover heroin and nimetazepam, so LLM chatbots are explored as an auxiliary annotation path — the paper explicitly plans to integrate AI methodologies into SMART as an auxiliary tool. Validate against the reported outcome: ChatGPT-4, ChatGPT-3.5, and Llama3-70B identify all 12 narcotics; Copilot misses delta9-THC and nimetazepam and misidentifies FM2.

## Related tools

- **ChatGPT-4** (Primary LLM evaluated for identifying all 12 narcotics from m/z values; queried via its API with SI Text S3 prompts)
- **ChatGPT-3.5** (Secondary LLM queried via its API with identical prompts and m/z values)
- **Llama3-70B** (Open-weight LLM evaluated on the same m/z identification task)
- **Windows Copilot** (LLM evaluated with Precision Mode enabled; failed on delta9-THC, nimetazepam, and FM2)
- **SMART 2.0** (Source of the narcotics example dataset and of the HMDB/MassBank-based peak-identification module the LLM approach is compared against) — https://github.com/YuJenL/SMART

## Examples

```
# Workflow: prompt LLM with m/z values from SMART 2.0 narcotics dataset (SI Text S3 prompt), then compile
# llm_peak_identification.csv: model, m/z, identified_compound for ChatGPT-4/3.5 (API), Llama3-70B, Copilot (Precision Mode)
# compare against SMART 2.0 HMDB/MassBank peak-identification output for the 12 narcotics
```

## Evaluation signals

- llm_peak_identification.csv exists with exactly three columns: model, m/z, and identified compound, covering 12 rows per model
- ChatGPT-4, ChatGPT-3.5, and Llama3-70B correctly identify all 12 narcotics
- Windows Copilot fails to identify delta9-THC and nimetazepam and incorrectly suggests alternatives for FM2
- Each model receives byte-identical prompts (same SI Text S3 text, same 12 m/z values) to ensure a fair cross-model comparison
- Correctness is judged against the known narcotic identities in the SMART 2.0 narcotics dataset, not against HMDB/MassBank (which lack heroin and nimetazepam)

## Limitations

- Windows Copilot (even in Precision Mode) misses delta9-THC and nimetazepam and misidentifies FM2, showing model-dependent hallucination risk
- LLM output is not a validated identity — the paper requires MS/MS, standard comparison, NMR, or chemical-reaction validation to confirm identities
- One-to-one identification from m/z alone is infeasible when other metabolites share the same m/z
- SMART 2.0/HMDB cannot detect heroin and nimetazepam, so no ground-truth database annotation exists for those narcotics
- AI integration into SMART is still planned future work, not a shipped feature

## Evidence

- [full_text] LLMs correctly identify all 12 narcotics (ChatGPT-4, ChatGPT-3.5, Llama3-70B); Copilot misses two and misidentifies one: [evidence span withheld — anchor status: not_found]
- [discussion] SMART 2.0 peak identification uses HMDB and MassBank m/z matching: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [discussion] AI methodologies planned as auxiliary tool, not yet integrated into SMART: "Our plans include integrating AI methodologies into SMART as an auxiliary tool to enhance its capabilities"
- [discussion] Direct one-to-one m/z identification infeasible due to shared m/z from other metabolites: "direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values"
- [readme] Prompts and repo are provided for reproducibility: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
- [discussion] Additional experiments required to confirm peak identities: "additional experiments such as MS/MS analysis, standard reference comparison, nuclear magnetic resonance (NMR) spectroscopy, and chemical reaction validation are essential to ascertain the identities"
