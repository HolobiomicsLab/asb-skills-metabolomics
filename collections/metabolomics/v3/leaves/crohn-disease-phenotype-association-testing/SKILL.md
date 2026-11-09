---
name: crohn-disease-phenotype-association-testing
description: Decision support for validating disease-metabolite associations when
  targeted quantitative measurements (e.g. conjugated bile acid mg/kg from LC-MS/MS
  fecal extracts) are available alongside binary phenotype labels such as active vs
  inactive Crohn's disease status.
when_to_use_negative:
- When the goal is discovery of candidate compounds rather than testing pre-specified
  analytes — use reverse metabolomics (synthesized standards + MASST searches of public
  data) instead, since this skill requires a defined analyte panel with absolute concentrations.
- When the outcome variable is multi-class or continuous (e.g. a continuous disease-activity
  score) rather than a two-group phenotype split like active vs inactive, where a
  one-sided Wilcoxon rank-sum design does not apply.
- When concentrations are relative (peak-area-only) without internal-standard normalization
  and matrix-matched calibration, since the reported CD-only increase of conjugated
  bile acids was quantified in mg/kg against a sulfadimethoxine standard.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3337
tools:
- name: MASST
  role: Source of the public MSV000092337 Bruker Q-ToF fecal-extract LC-MS/MS data
    and disease-status metadata that feed the phenotype-association test
- name: Bruker Q-ToF
  role: Instrument on which the conjugated bile acids were detected in human IBD fecal
    samples (positive mode), producing the targeted concentration data tested against
    CD phenotype
- name: scipy
  role: One-sided Wilcoxon rank-sum testing of active vs inactive CD concentrations
    for each of the 19 quantified bile acids
- name: pandas
  role: Assembling the per-sample mg/kg concentration table, group summaries, and
    test statistics into bile_acid_cd_quantification.csv
- name: numpy
  role: Array operations supporting ion-chromatogram integration and calibration-curve
    conversion feeding the association testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:11404d70721974d4cc2a3e3a49c5d31484db05a7def4699b639eea1cd0e01ac1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bruker Q-ToF
  - MASST
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3337
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# crohn-disease-phenotype-association-testing

## Summary

Statistical association testing that links targeted metabolite concentrations to Crohn's disease phenotype, e.g. one-sided Wilcoxon rank-sum tests of 19 conjugated bile acids quantified in mg/kg from the MSV000092337 Q-ToF fecal cohort, to confirm reverse-metabolomics discoveries across human IBD cohorts.

## When to use

Decision support for validating disease-metabolite associations when targeted quantitative measurements (e.g. conjugated bile acid mg/kg from LC-MS/MS fecal extracts) are available alongside binary phenotype labels such as active vs inactive Crohn's disease status.

## When NOT to use

- When the goal is discovery of candidate compounds rather than testing pre-specified analytes — use reverse metabolomics (synthesized standards + MASST searches of public data) instead, since this skill requires a defined analyte panel with absolute concentrations.
- When the outcome variable is multi-class or continuous (e.g. a continuous disease-activity score) rather than a two-group phenotype split like active vs inactive, where a one-sided Wilcoxon rank-sum design does not apply.
- When concentrations are relative (peak-area-only) without internal-standard normalization and matrix-matched calibration, since the reported CD-only increase of conjugated bile acids was quantified in mg/kg against a sulfadimethoxine standard.

## Inputs

- Targeted LC-MS/MS metabolite concentration table (mg/kg) for the analyte panel (e.g. 19 conjugated bile acids with sulfadimethoxine internal standard) from the MSV000092337 Q-ToF dataset
- Sample metadata mapping each fecal LC-MS/MS run to disease status (active CD n=23, inactive CD n=72; UC groups as controls)

## Outputs

- Per-analyte table of one-sided Wilcoxon rank-sum statistics and p-values comparing active vs inactive CD, plus group-level concentration summaries
- Per-sample concentration matrix (mg/kg) aligned to disease-status labels, exported as bile_acid_cd_quantification.csv

## How to apply

Start from targeted metabolite quantification already converted to absolute concentrations (e.g. mg/kg fecal extract via matrix-matched calibration and internal-standard normalization to sulfadimethoxine). Group samples by phenotype metadata — in the MSV000092337 Crohn's disease cohort this means separating active (n=23) from inactive (n=72) individuals using the sample annotations parsed from MassIVE metadata. For each of the 19 quantified bile acids, run a one-sided Wilcoxon rank-sum test (scipy.stats) comparing active vs inactive concentrations, appropriate because concentrations are non-normal and sample sizes are modest. Run the same test against ulcerative colitis groups as a specificity control, since the reported association was CD-specific and not observed for UC. Assemble a single table with per-sample mg/kg values, group summaries, and per-analyte one-sided Wilcoxon statistics and p-values, written to a CSV such as bile_acid_cd_quantification.csv. The result is considered correct when it reproduces the expected outcome — that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr (11 of 19 analytes) are significantly increased in active CD only.

## Related tools

- **MASST** (Source of the public MSV000092337 Bruker Q-ToF fecal-extract LC-MS/MS data and disease-status metadata that feed the phenotype-association test)
- **Bruker Q-ToF** (Instrument on which the conjugated bile acids were detected in human IBD fecal samples (positive mode), producing the targeted concentration data tested against CD phenotype)
- **scipy** (One-sided Wilcoxon rank-sum testing of active vs inactive CD concentrations for each of the 19 quantified bile acids)
- **pandas** (Assembling the per-sample mg/kg concentration table, group summaries, and test statistics into bile_acid_cd_quantification.csv)
- **numpy** (Array operations supporting ion-chromatogram integration and calibration-curve conversion feeding the association testing)

## Examples

```
from scipy.stats import ranksums; import pandas as pd; df = pd.read_csv('bile_acid_cd_quantification.csv'); ranksums(df[df.status=='active'].conc, df[df.status=='inactive'].conc, alternative='greater')
```

## Evaluation signals

- The result table reproduces the reported outcome: 11 of 19 quantified bile acids are significantly increased in active CD, and the increase is CD-specific (not replicated for UC).
- The significantly increased analytes include cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr, matching the paper's validated panel.
- Group sizes match the metadata: n=23 active vs n=72 inactive CD samples from MSV000092337, with each test performed one-sided.
- Output CSV (bile_acid_cd_quantification.csv) contains, for every analyte, per-sample mg/kg values, group summaries, and both the Wilcoxon statistic and p-value.
- Concentrations are traceable to internal-standard-normalized responses converted through matrix-matched calibration curves (sulfadimethoxine as IS), not raw peak areas.

## Limitations

- Public spectra repositories searched by MASST do not encompass all molecules, limiting discovery-phase reanalysis; this skill only confirms associations for analytes that already have synthesized standards and targeted quantification.
- Without isolation and NMR or X-ray structural analyses, isomer identity cannot be fully resolved, so the reported associations may represent mixtures of structurally related isomers.
- The 1.2 billion MASST-searchable spectra may not cover the compound of interest, so phenotype-association testing is only possible for compounds whose MS/MS spectra were matched in public data in the first place.

## Evidence

- [results] 11/19 significant, CD-specific: "Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease."
- [abstract] Discovery via repository-scale reverse metabolomics: [evidence span withheld — anchor status: not_found]
- [discussion] CD-specific diagnostic relevance: "We demonstrated that we could connect newly discovered bile acids to disease phenotypes, something that may eventually have diagnostic value to accurately and non-invasively detect CD."
- [discussion] reverse metabolomics requires an MS/MS spectrum: "metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition, as reverse metabolomics requires a MS/MS spectrum."
- [discussion] isomer ambiguity caveat: "However, without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data."
