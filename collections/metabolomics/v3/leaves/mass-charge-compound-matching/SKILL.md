---
name: mass-charge-compound-matching
description: Use when untargeted LC-MS peak tables contain only m/z values that need
  candidate compound identities, e.g. assigning narcotics peaks from the SMART 2.0
  example dataset via HMDB/MassBank matching or LLM peak annotation.
when_to_use_negative:
- When only m/z matching is insufficient — e.g. if direct one-to-one identification
  without prior drug information is required, since other metabolites can share identical
  m/z values and MS/MS or standard-reference confirmation is needed.
- When the target compound is not represented in HMDB or MassBank — e.g. heroin and
  nimetazepam are absent from HMDB, so HMDB-based matching will miss them.
- When accurate quantification or calibration is the goal — use SMART 2.0's concentration-calibration
  module instead, as low-abundance amphetamine peaks (S/N 0.19) showed detection issues
  unrelated to compound matching.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SMART 2.0
  role: Peak-identification module that matches m/z data against HMDB and MassBank
    for compound assignment in the post-analysis workflow
  repo: https://github.com/YuJenL/SMART
- name: ChatGPT-4
  role: LLM chatbot used with documented peak-identification prompts to identify narcotics
    from m/z values (correctly identified all 12 in the comparison)
- name: ChatGPT-3.5
  role: LLM chatbot used with documented peak-identification prompts to identify narcotics
    from m/z values (correctly identified all 12 in the comparison)
- name: Llama3-70B
  role: Open LLM used with the same peak-identification prompts to assign compound
    identities from m/z values (correctly identified all 12 narcotics)
- name: Windows Copilot
  role: LLM chatbot run with Precision Mode enabled for m/z-based compound identification;
    missed delta9-THC and nimetazepam and misidentified FM2
- name: MassBank
  role: Reference spectral/mass database used by the SMART 2.0 peak-identification
    module for m/z matching
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:a9e0f787e099fc98a0fe02f2fb91ad927f179e00be6512316936d26e0319ce6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - ChatGPT-3.5
  - ChatGPT-4
  - Llama3-70B
  - MassBank
  - SMART 2.0
  - Windows Copilot
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mass-charge-compound-matching

## Summary

Match untargeted peak m/z values against reference metabolite databases (HMDB and MassBank) to assign candidate compound identities, using either the SMART 2.0 peak-identification module or LLM chatbot prompting with documented peak-identification prompts.

## When to use

Use when untargeted LC-MS peak tables contain only m/z values that need candidate compound identities, e.g. assigning narcotics peaks from the SMART 2.0 example dataset via HMDB/MassBank matching or LLM peak annotation.

## When NOT to use

- When only m/z matching is insufficient — e.g. if direct one-to-one identification without prior drug information is required, since other metabolites can share identical m/z values and MS/MS or standard-reference confirmation is needed.
- When the target compound is not represented in HMDB or MassBank — e.g. heroin and nimetazepam are absent from HMDB, so HMDB-based matching will miss them.
- When accurate quantification or calibration is the goal — use SMART 2.0's concentration-calibration module instead, as low-abundance amphetamine peaks (S/N 0.19) showed detection issues unrelated to compound matching.

## Inputs

- Untargeted peak table with m/z values (e.g. the 12 narcotics m/z values from the SMART 2.0 narcotics example dataset)
- Peak-identification prompts from the article's Supporting Information Text S3 (for the LLM route)
- Access to HMDB and MassBank reference m/z records (for the SMART 2.0 route)
- Access to ChatGPT-4 / ChatGPT-3.5 API, Llama3-70B, or Windows Copilot (Precision Mode) for the LLM route

## Outputs

- Compound identity assignment per m/z (or an explicit failure-to-assign record), e.g. compiled into a table with columns model, m/z, and identified compound
- Per-model annotation outcome such as llm_peak_identification.csv listing which LLM identified which narcotics

## How to apply

Extract the m/z values of interest (e.g. the 12 narcotics from the SMART 2.0 narcotics example dataset) and run them through one of two routes. Route 1: use the SMART 2.0 post-analysis peak-identification module, which matches m/z data against HMDB and MassBank records; note that HMDB does not cover heroin and nimetazepam, so coverage depends on the chosen database. Route 2: submit the peak-identification prompts from the article's Supporting Information (Text S3) together with the m/z values to ChatGPT-4, ChatGPT-3.5, Llama3-70B, or Windows Copilot (with Precision Mode enabled), then parse the natural-language responses into a table of model, m/z, and identified compound. Direct one-to-one m/z matching alone is not decisive because other metabolites can share identical m/z values — confirm identities with additional evidence such as MS/MS fragments (e.g. identical fragments at m/z 91, 119, 136 for both amphetamine MS1 peaks), standard-reference comparison, NMR, or chemical-reaction validation.

## Related tools

- **SMART 2.0** (Peak-identification module that matches m/z data against HMDB and MassBank for compound assignment in the post-analysis workflow) — https://github.com/YuJenL/SMART
- **ChatGPT-4** (LLM chatbot used with documented peak-identification prompts to identify narcotics from m/z values (correctly identified all 12 in the comparison))
- **ChatGPT-3.5** (LLM chatbot used with documented peak-identification prompts to identify narcotics from m/z values (correctly identified all 12 in the comparison))
- **Llama3-70B** (Open LLM used with the same peak-identification prompts to assign compound identities from m/z values (correctly identified all 12 narcotics))
- **Windows Copilot** (LLM chatbot run with Precision Mode enabled for m/z-based compound identification; missed delta9-THC and nimetazepam and misidentified FM2)
- **MassBank** (Reference spectral/mass database used by the SMART 2.0 peak-identification module for m/z matching) — https://github.com/MassBank/MassBank-data

## Examples

```
Rscript -e "library(SMART); SMART_peak_identification(mz_table = 'narcotics_mz.csv', db = c('HMDB','MassBank'))"
```

## Evaluation signals

- The compiled annotation table matches the reported outcome: ChatGPT-4, ChatGPT-3.5, and Llama3-70B identify all 12 narcotics, while Windows Copilot misses delta9-THC and nimetazepam and misidentifies FM2.
- Every m/z row has either an assigned compound or an explicit failure-to-assign entry — no silent blanks in the output table.
- SMART 2.0 m/z matching reproduces the article's HMDB coverage result: heroin and nimetazepam are not detected because HMDB does not include them.
- Compounds assigned solely from m/z are flagged for follow-up MS/MS / standard-reference / NMR / chemical-reaction validation when identities are uncertain.
- LLM prompts used are the documented peak-identification prompts from Supporting Information Text S3, not ad-hoc wording.

## Limitations

- HMDB does not encompass heroin and nimetazepam, so SMART 2.0 peak identification cannot detect them.
- Direct one-to-one comparison of narcotics by m/z is not feasible because other metabolites can share identical m/z values; additional experiments (MS/MS, standard reference comparison, NMR, chemical reaction validation) are essential to ascertain identities.
- LLM-based annotation is promising but AI methodologies were not yet integrated into SMART at the time of writing (planned future work), so chatbot results should be treated as auxiliary.
- Different LLMs show unequal performance — Windows Copilot failed to identify delta9-THC and nimetazepam and incorrectly suggested alternative compounds for FM2, so chatbot choice affects results.
- Consistency of LC-MS experimental conditions (which affects m/z accuracy) is required but not guaranteed; variations were observed in the study.

## Evidence

- [discussion] The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [discussion] Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
- [discussion] direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values: "direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values"
- [discussion] MS/MS analysis at 500 ppb, which revealed that both MS1 peaks produced identical fragmentation patterns (m/z 91, 119, and 136): "MS/MS analysis at 500 ppb, which revealed that both MS1 peaks produced identical fragmentation patterns (m/z 91, 119, and 136)"
- [abstract] We also explored the potential of using large language models for peak annotation and have found the results to be promising.: "We also explored the potential of using large language models for peak annotation and have found the results to be promising."
- [readme] Post-analysis: Execute peak identification and concentration calibration.: "Post-analysis: Execute peak identification and concentration calibration."
- [other] ChatGPT-4, ChatGPT-3.5, and Llama3-70B correctly identified all 12 narcotics, whereas Windows Copilot failed to identify delta9-THC and nimetazepam and incorrectly identified FM2: [evidence span withheld — anchor status: not_found]
