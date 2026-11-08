---
name: reaction-based-pseudo-ms1-annotation
description: Use when you need to annotate untargeted LC-HRMS MS1 features from biotransformation
  assays (e.g., human liver S9 incubations of xenobiotics) using reaction rules and
  a curated metabolite database, rather than spectral-library 'bottom-up' matching.
when_to_use_negative:
- Input MS2 spectra already match reference spectral libraries with high confidence
  — bottom-up library matching is the appropriate route and RCS adds reaction-rule
  assumptions unnecessarily
- The xenobiotic parent set has no reaction-rule or database coverage (no analog in
  the curated PMDB), so theoretical matching cannot generate candidates
- The analysis goal is intact-compound profiling rather than discovery of biotransformation-derived
  metabolites
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: CMDN
  role: Implements the triple-layered framework including the reaction-based pseudo-MS1
    candidature space; R scripts with the 290 reaction rules and PMDB are provided
    in the repository
  repo: LeaveMeNotTonight/CMDN
- name: MSDial
  role: Compatible software (ver. 4.80) used to process LC-HRMS raw data into the
    MS1 feature table that RCS annotates
- name: BioTransformer 3.0
  role: Compatible in silico biotransformation prediction tool referenced alongside
    CMDN for metabolite candidate generation
- name: CluMSID
  role: R dependency used in the CMDN scripts, supporting MS2 spectral similarity
    work in the downstream SSN layer that validates RCS candidates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:33358dcf1eabc50811054eee27fc456ff59cac680f4d240bd3610885216feae7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - BioTransformer 3.0
  - CMDN
  - CluMSID
  - MSDial
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# reaction-based-pseudo-ms1-annotation

## Summary

Reaction-based pseudo-MS1 candidature space (RCS) annotation from the CMDN framework: applies 290 curated biotransformation reaction rules against a curated pesticide metabolite database (PMDB, N = 492) to theoretically match and deduplicate MS1 features from untargeted LC-HRMS data of enzymatic assays, enabling high-throughput annotation of xenobiotic metabolites without spectral libraries.

## When to use

Use when you need to annotate untargeted LC-HRMS MS1 features from biotransformation assays (e.g., human liver S9 incubations of xenobiotics) using reaction rules and a curated metabolite database, rather than spectral-library 'bottom-up' matching.

## When NOT to use

- Input MS2 spectra already match reference spectral libraries with high confidence — bottom-up library matching is the appropriate route and RCS adds reaction-rule assumptions unnecessarily
- The xenobiotic parent set has no reaction-rule or database coverage (no analog in the curated PMDB), so theoretical matching cannot generate candidates
- The analysis goal is intact-compound profiling rather than discovery of biotransformation-derived metabolites

## Inputs

- Processed LC-HRMS MS1 feature table (e.g., from MS-Dial ver. 4.80) with accurate m/z and intensities from xenobiotic biotransformation assays
- Curated reaction-derived metabolite database (CMDN-annotated PMDB with manual curation, N = 492, Table S2 sheet 1)
- CMDN reaction rule set (n = 290) defining biotransformation mass shifts (e.g., oxidation, demethylation, deethylation)

## Outputs

- Table of MS1 features annotated by theoretical matching against reaction-rule-transformed PMDB masses (6,181 features)
- Deduplicated nonredundant pseudo-MS1 candidate list (3,813 features) forwarded to the MS2 SSN layer
- cmdn_summary_statistics.csv with per-statistic reproduction values and reaction-class prevalences (oxidation 31.7%, demethylation 12.5%, deethylation 11.2%)

## How to apply

First generate the MS1 feature table (e.g., 32,827 total features from the 1,021-pesticide human liver S9 assay processed with MS-Dial ver. 4.80) and derive the differential metabolite space (793 upregulated features) as the feature universe. Then apply the 290 CMDN reaction rules against the CMDN-annotated, manually curated PMDB (N = 492) to compute theoretical biotransformation product masses, matching MS1 features by accurate m/z — this yields 6,181 annotated MS1 features by theoretical matching. Deduplicate redundant assignments (multiple parent-product-rule combinations matching one feature) to retain 3,813 nonredundant MS1 candidates. Finally, pass RCS candidates to the MS2 spectrum similarity network (SSN) layer, where PRM spectra are realigned against candidates; the paper reports an 87.3% MS1-MS2 match rate, so RCS output quality is judged by how many candidates acquire corroborating MS2 spectra. Rationale: this 'top-down' reaction-guided strategy circumvents the scarcity of authentic standards and the reference-library dependence of bottom-up workflows.

## Related tools

- **CMDN** (Implements the triple-layered framework including the reaction-based pseudo-MS1 candidature space; R scripts with the 290 reaction rules and PMDB are provided in the repository) — LeaveMeNotTonight/CMDN
- **MSDial** (Compatible software (ver. 4.80) used to process LC-HRMS raw data into the MS1 feature table that RCS annotates)
- **BioTransformer 3.0** (Compatible in silico biotransformation prediction tool referenced alongside CMDN for metabolite candidate generation)
- **CluMSID** (R dependency used in the CMDN scripts, supporting MS2 spectral similarity work in the downstream SSN layer that validates RCS candidates)

## Examples

```
library(tidyverse); library(OrgMassSpecR); library(MSMSsim); library(msentropy)  # R dependencies for CMDN scripts (see LeaveMeNotTonight/CMDN README)
```

## Evaluation signals

- Theoretical MS1 matching count reproduces the reported 6,181 annotated features and deduplication yields 3,813 retained candidates
- 87.3% of RCS candidates acquire corroborating MS1-MS2 matches in the downstream SSN realignment step
- Reaction-class prevalences among annotations match reported values: oxidation 31.7%, demethylation 12.5%, deethylation 11.2%
- Each computed statistic matches the reported value (e.g., 793 upregulated DMS features feeding the pipeline) within the reported rounding; discrepancies flag failed reproduction
- Downstream per-pesticide metrics (median transformation rate 35.3%; 533 downregulated pesticides, 52.2%) fall in reported ranges

## Limitations

- Annotation is pseudo-MS1 theoretical matching: candidates require independent MS2 corroboration (SSN layer) and, ideally, multichannel validation with analytical/stable-isotopically-labeled standards, RT prediction, time-course logistic modeling, or in vivo mouse studies
- Coverage is bounded by the 290 adopted reaction rules and the manually curated PMDB (N = 492); metabolites outside this rule/database space will be missed
- RCS is demonstrated on human liver S9 enzymatic assays of 1,021 pesticides; transfer to other matrices or chemical classes requires rebuilding the parent database and reaction rules

## Evidence
