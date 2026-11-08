---
name: metabolomics-benchmark-reproduction
description: Use when the goal is to reproduce or validate a published metabolomics
  analysis — such as matching the SMART 2.0 narcotics or breast-cancer demo outputs,
  or replicating LLM peak-identification results on 12 narcotics m/z values.
when_to_use_negative:
- Input is an untargeted LC-MS dataset for a novel biological question — this skill
  replays published benchmarks, not discovery analyses.
- The target analysis involves metabolites absent from the reference resources (e.g.
  heroin and nimetazepam, which HMDB does not include and SMART 2.0 cannot detect).
- Direct one-to-one identification is expected without prior drug information — the
  paper notes this is infeasible because other metabolites may share identical m/z
  values.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMART 2.0
  role: The R tool whose modules (import through post-analysis) are the benchmark
    being reproduced; run with the published parameters on the narcotics and breast-cancer
    example datasets.
  repo: https://github.com/YuJenL/SMART
- name: Pbine
  role: Numerical-integration p-value combination method used to reproduce integrative
    omics pathway significance values (e.g. FDR-corrected p = 2.80 × 10⁻¹⁰ for hsa04151).
  repo: https://github.com/Yinchun-Lin/Pbine
- name: MassBank
  role: MassBank records underpin the peak-identification post-analysis module used
    in the reproduction of narcotics identification.
  repo: https://github.com/MassBank/MassBank-data
- name: ChatGPT-4
  role: LLM submitted identical peak-identification prompts and 12 narcotics m/z values;
    benchmark expects all 12 correctly identified.
- name: ChatGPT-3.5
  role: LLM submitted identical peak-identification prompts and 12 narcotics m/z values;
    benchmark expects all 12 correctly identified.
- name: Llama3-70B
  role: LLM submitted identical peak-identification prompts and 12 narcotics m/z values;
    benchmark expects all 12 correctly identified.
- name: Windows Copilot
  role: LLM run with Precision Mode; benchmark expects failure to identify delta9-THC
    and nimetazepam and incorrect identification of FM2.
- name: IntLIM
  role: R package with a documented vignette and example data; serves as a reference
    benchmark for reproducing integrative metabolomics–transcriptomics workflows.
  repo: https://github.com/Mathelab/IntLIM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:81153015a03a10c698342e1189462c915c5bda6655e91a3acbbd9ae973aeb302
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - ChatGPT-3.5
  - ChatGPT-4
  - IntLIM
  - Llama3-70B
  - MassBank
  - Pbine
  - SMART 2.0
  - Windows Copilot
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolomics-benchmark-reproduction

## Summary

Reproduces a published metabolomics benchmark or demo analysis by replaying the exact tool versions, prompts, and parameters used in the original study — e.g. the SMART 2.0 narcotics/breast-cancer examples or the LLM peak-identification comparison — and validating the output against reported outcomes.

## When to use

Use when the goal is to reproduce or validate a published metabolomics analysis — such as matching the SMART 2.0 narcotics or breast-cancer demo outputs, or replicating LLM peak-identification results on 12 narcotics m/z values.

## When NOT to use

- Input is an untargeted LC-MS dataset for a novel biological question — this skill replays published benchmarks, not discovery analyses.
- The target analysis involves metabolites absent from the reference resources (e.g. heroin and nimetazepam, which HMDB does not include and SMART 2.0 cannot detect).
- Direct one-to-one identification is expected without prior drug information — the paper notes this is infeasible because other metabolites may share identical m/z values.

## Inputs

- SMART 2.0 narcotics example dataset (m/z values for 12 narcotics)
- Supporting Information Text S3 peak-identification prompts
- breast cancer demonstrative dataset from SMART 2.0 example data
- LLM access endpoints for ChatGPT-4, ChatGPT-3.5, Llama3-70B, and Windows Copilot (Precision Mode)
- SMART 2.0 user guide and example data from https://github.com/YuJenL/SMART

## Outputs

- llm_peak_identification.csv with columns model, m/z, identified compound
- reproduced SMART 2.0 module outputs (TIC, mass spectra, QC tables, pathway significance values)
- comparison report confirming each module's output matches reported values or documenting deviations

## How to apply

First obtain the authors' reference materials: the SMART 2.0 GitHub repo (YuJenL/SMART) provides the code, a user guide, and example data covering the narcotics and breast-cancer demonstrative datasets. For each module in the published workflow — data import, visualization, peak analysis/annotation, preprocessing, quality control, batch-effect analysis, statistical analysis, and post-analysis — apply the same parameter choices the paper used, then diff your outputs against the reported values (e.g. m/z 91/119/136 fragments for amphetamine MS/MS, S/N ratio 0.19 for amphetamine). For LLM-based peak identification, extract the 12 narcotics m/z values from the SMART 2.0 narcotics example, load the documented peak-identification prompts from Supporting Information Text S3, and submit the identical prompt/m-z pairs to each model (ChatGPT-4, ChatGPT-3.5, Llama3-70B, Windows Copilot with Precision Mode); compile responses into a table with columns model, m/z, and identified compound, then verify that ChatGPT-4, ChatGPT-3.5, and Llama3-70B identify all 12 narcotics while Windows Copilot misses delta9-THC and nimetazepam and misidentifies FM2. When reproducing pathway analysis, check that e.g. the PI3K-Akt signaling pathway (hsa04151) returns an FDR-corrected p-value near 2.80 × 10⁻¹⁰ using the Pbine method, and the pentose phosphate pathway shows ORA p-values ≈ 0.02202 (gene) and 0.02009 (metabolite).

## Related tools

- **SMART 2.0** (The R tool whose modules (import through post-analysis) are the benchmark being reproduced; run with the published parameters on the narcotics and breast-cancer example datasets.) — https://github.com/YuJenL/SMART
- **Pbine** (Numerical-integration p-value combination method used to reproduce integrative omics pathway significance values (e.g. FDR-corrected p = 2.80 × 10⁻¹⁰ for hsa04151).) — https://github.com/Yinchun-Lin/Pbine
- **MassBank** (MassBank records underpin the peak-identification post-analysis module used in the reproduction of narcotics identification.) — https://github.com/MassBank/MassBank-data
- **ChatGPT-4** (LLM submitted identical peak-identification prompts and 12 narcotics m/z values; benchmark expects all 12 correctly identified.)
- **ChatGPT-3.5** (LLM submitted identical peak-identification prompts and 12 narcotics m/z values; benchmark expects all 12 correctly identified.)
- **Llama3-70B** (LLM submitted identical peak-identification prompts and 12 narcotics m/z values; benchmark expects all 12 correctly identified.)
- **Windows Copilot** (LLM run with Precision Mode; benchmark expects failure to identify delta9-THC and nimetazepam and incorrect identification of FM2.)
- **IntLIM** (R package with a documented vignette and example data; serves as a reference benchmark for reproducing integrative metabolomics–transcriptomics workflows.) — https://github.com/Mathelab/IntLIM

## Examples

```
git clone https://github.com/YuJenL/SMART && Rscript -e 'library(SMART); SMART::run()'
```

## Evaluation signals

- LLM identification table matches the reported outcome: ChatGPT-4, ChatGPT-3.5, and Llama3-70B identify all 12 narcotics; Windows Copilot misses delta9-THC and nimetazepam and misidentifies FM2.
- Pathway reproducibility check: PI3K-Akt signaling pathway (hsa04151) FDR-corrected p-value ≈ 2.80 × 10⁻¹⁰ using the Pbine method.
- Pentose phosphate pathway ORA p-values ≈ 0.02202 (gene) and 0.02009 (metabolite).
- Amphetamine-specific checks: MS/MS fragmentation at m/z 91, 119, and 136, and S/N ratio 0.19, reproduce the reported values.
- Identified metabolite assignments can be cross-verified against HMDB and MassBank m/z matching used by the peak-identification module.

## Limitations

- SMART 2.0 cannot detect heroin and nimetazepam because HMDB does not include them.
- Direct one-to-one identification of narcotics without prior drug information is not feasible because other metabolites may share identical m/z values.
- Low-abundance peaks may be mis-detected without provided retention times; amphetamine detection requires re-evaluation of peak detection and LC-MS experimental conditions.
- Additional experiments (MS/MS, standard comparison, NMR, chemical reaction validation) are needed to confirm the identity of unknown peaks before treating any reproduction as definitive.
- Consistency in LC-MS experimental conditions is required for a reliable calibration curve; variations across runs undermine quantification accuracy.

## Evidence

- [abstract] The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART.: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
- [abstract] This study employs narcotics data and breast cancer data as demonstrative examples: "This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities."
- [full_text] ChatGPT-4, ChatGPT-3.5, and Llama3-70B correctly identified all 12 narcotics: [evidence span withheld — anchor status: not_found]
- [supplementary] Extract the 12 narcotics m/z values from the SMART 2.0 narcotics example dataset.: [evidence span withheld — anchor status: not_found]
- [discussion] Peak identification uses m/z data from HMDB and MassBank: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [readme] SMART streamlines the complete analysis flow: "SMART streamlines the complete analysis flow from initial data preprocessing to advanced downstream data analysis, consisting of the following eight modules."
- [readme] Pbine combines p-values via numerical integration: [evidence span withheld — anchor status: external_doc]
- [discussion] HMDB does not encompass heroin and nimetazepam: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
- [discussion] Additional experiments are essential to ascertain identities: "additional experiments such as MS/MS analysis, standard reference comparison, nuclear magnetic resonance (NMR) spectroscopy, and chemical reaction validation are essential to ascertain the identities"
