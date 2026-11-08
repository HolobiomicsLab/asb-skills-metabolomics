---
name: bile-amidate-feature-annotation
description: Use when an untargeted LC–MS/MS feature table from human fecal/stool
  cohorts contains unannotated features suspected to be bile acid–amino acid conjugates
  and the goal is to assign bile-amidate identity via repository spectral matching
  and test IBD/CD association.
when_to_use_negative:
- The input data are already an annotated feature table of known bile acid–amino acid
  conjugates and no repository spectral matching is needed
- The samples come from cohorts outside the IBD/stool-fecal phenotype metadata scope,
  so the CD vs non-IBD vs UC stratification and Wilcoxon testing step is not meaningful
- The analysis goal is discovery of novel structures rather than annotation/validation
  of already-synthesized bile amidate reference spectra
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MASST
  role: Repository-scale spectral search engine used to match synthesized bile amidate
    MS/MS spectra against ~1.2 billion public spectra in GNPS/MassIVE, returning matched
    datasets and unique files for annotation.
- name: Bruker Q-ToF
  role: Acquisition instrument for the positive-mode reference tandem MS spectra of
    the synthesized conjugated bile acids deposited as MSV000087522.
- name: Thermo QE Orbitrap
  role: Acquisition instrument for the negative-mode reference tandem MS spectra of
    the conjugated bile acids detected in iHMP2 stool samples, deposited as MSV000087523.
- name: Agilent 6560 IM-Q-ToF MS
  role: Acquisition instrument for the negative-mode reference tandem MS spectra of
    conjugated bile acids detected in bacterial cultures.
- name: conda
  role: Environment manager used to install dependencies and run the local workflow
    for library generation and deployment of the reverse-metabolomics pipeline.
  repo: https://github.com/Wang-Bioinformatics-Lab/GNPS2_DeploymentTooling
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41586-023-06906-8
    title: Reverse metabolomics for the discovery of chemical structures from humans
  source_content_hash: sha256:0988f58ade88b07f66ca5fe0f510404c500aa2165f222ea6efbfab6b093e430c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wang-Bioinformatics-Lab/GNPS2_DeploymentTooling
  tools:
  - Agilent 6560 IM-Q-ToF MS
  - Bruker Q-ToF
  - MASST
  - Thermo QE Orbitrap
  - conda
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bile-amidate-feature-annotation

## Summary

Annotate untargeted metabolomics features as conjugated bile acids (bile amidates) by matching MS/MS spectra of synthesized cholic- and secondary-bile-acid–amino-acid conjugates against public repository data via MASST, then stratify matched features by IBD phenotype to test Crohn's disease association.

## When to use

Use when an untargeted LC–MS/MS feature table from human fecal/stool cohorts contains unannotated features suspected to be bile acid–amino acid conjugates and the goal is to assign bile-amidate identity via repository spectral matching and test IBD/CD association.

## When NOT to use

- The input data are already an annotated feature table of known bile acid–amino acid conjugates and no repository spectral matching is needed
- The samples come from cohorts outside the IBD/stool-fecal phenotype metadata scope, so the CD vs non-IBD vs UC stratification and Wilcoxon testing step is not meaningful
- The analysis goal is discovery of novel structures rather than annotation/validation of already-synthesized bile amidate reference spectra

## Inputs

- Reference tandem MS spectra of synthesized bile amidates from MassIVE deposits MSV000087522 (Bruker Q-ToF, positive mode) and MSV000087523 (Thermo QE Orbitrap, negative mode)
- MASST job/USI identifiers for the synthesized compounds from supplementary material
- Public relative-quantification feature tables for Metabolomics Workbench studies PR000639 (iHMP2 stool cohort) and PR000677 (PRISM IBD fecal cohort)
- IBD phenotype metadata stratifying samples into non-IBD control, Crohn's disease (CD), and ulcerative colitis (UC)

## Outputs

- Per-compound MASST match table with compound identifier, MASST job/USI, instrument platform, total match count, and number of unique matched files
- bile_amidate_wilcoxon_results.csv with feature identifier, conjugate assignment, test group pair, raw p-value, BH-adjusted q-value, and group median direction

## How to apply

Obtain reference tandem MS spectra of the synthesized bile amidates from the MassIVE deposits MSV000087522 (Bruker Q-ToF, positive mode) and MSV000087523 (Thermo QE Orbitrap, negative mode), matching the acquisition polarity to the sample's instrument platform. Submit each compound's MS/MS spectrum as a MASST search against the ~1.2 billion public spectra in GNPS/MassIVE, deduplicating matched dataset files so each public file is counted once per compound. Parse the returned MASST results and compile a per-compound match table (compound identifier, MASST job/USI, instrument platform, total match count, unique matched files). Then, for each cohort (e.g. iHMP2 stool PR000639, PRISM fecal PR000677), download the relative-quantification feature table, subset rows to the confirmed bile-amidate features (cholic acid and secondary bile acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp, Tyr), stratify samples by IBD phenotype (non-IBD, CD, UC), run two-sided Wilcoxon rank-sum tests (scipy.stats.mannwhitneyu) per pairwise comparison, apply Benjamini-Hochberg FDR correction (statsmodels.stats.multitest.multipletests, method='fdr_bh'), and write feature identifier, conjugate assignment, group pair, raw p, BH q, and median direction to a results CSV. Validate that cholic-acid conjugates to Glu, Ile/Leu, Phe, Thr, Trp and Tyr show significantly increased abundance in CD versus non-IBD (q < 0.05, positive direction) while secondary bile acid amidates show no significant difference.

## Related tools

- **MASST** (Repository-scale spectral search engine used to match synthesized bile amidate MS/MS spectra against ~1.2 billion public spectra in GNPS/MassIVE, returning matched datasets and unique files for annotation.)
- **Bruker Q-ToF** (Acquisition instrument for the positive-mode reference tandem MS spectra of the synthesized conjugated bile acids deposited as MSV000087522.)
- **Thermo QE Orbitrap** (Acquisition instrument for the negative-mode reference tandem MS spectra of the conjugated bile acids detected in iHMP2 stool samples, deposited as MSV000087523.)
- **Agilent 6560 IM-Q-ToF MS** (Acquisition instrument for the negative-mode reference tandem MS spectra of conjugated bile acids detected in bacterial cultures.)
- **conda** (Environment manager used to install dependencies and run the local workflow for library generation and deployment of the reverse-metabolomics pipeline.) — https://github.com/Wang-Bioinformatics-Lab/GNPS2_DeploymentTooling

## Examples

```
from scipy.stats import mannwhitneyu; from statsmodels.stats.multitest import multipletests; u, p = mannwhitneyu(cd_vals, ctrl_vals, alternative='two-sided'); q = multipletests(pvals, method='fdr_bh')[1]
```

## Evaluation signals

- Aggregate MASST results reproduce the reported summary of 16,587 matches from 6,603 unique files covering 145 of the synthesized compounds
- Cholic-acid conjugates to Glu, Ile/Leu, Phe, Thr, Trp and Tyr show significantly increased abundance in CD versus non-IBD (q < 0.05, positive direction)
- Conjugates of secondary bile acids show no significant difference between CD and non-IBD groups
- Each public dataset file is counted once per compound after deduplication, avoiding inflated match counts
- BH-adjusted q-values are computed within each pairwise comparison (non-IBD vs CD, non-IBD vs UC, CD vs UC) rather than pooled across comparisons

## Limitations

- Isomer ambiguity: without isolation and NMR or X-ray structural analyses, one cannot exclude the possibility that different isomers are represented by the data
- The MS/MS matching protocol does not guarantee against false discovery of closely related isomers or compounds, though it prevents completely inaccurate identifications
- MASST public spectra (~1.2 billion) do not encompass all molecules, limiting re-analysis of public data in the discovery phase
- Metabolites must be present in sufficient quantities or ionize well to be detected and selected for MS/MS acquisition

## Evidence

- [abstract] Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn's disease.: "Validation using four distinct human IBD cohorts showed that cholic acids conjugated to Glu, Ile/Leu, Phe, Thr, Trp or Tyr are increased in Crohn’s disease."
- [supplementary] Conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode.: "conjugated bile acids detected in iHMP2 stool samples using a Thermo QE Orbitrap in negative mode."
- [supplementary] Conjugated bile acids detected in bacterial cultures on a Bruker Q-ToF in positive mode.: "conjugated bile acids detected in bacterial cultures on a Bruker Q-ToF in positive mode."
- [readme] Readme instructions for running the reverse metabolomics library generation workflow locally.: "You will need to have conda, mamba, and nextflow installed to run things locally."
- [results] Validation of the four-cohort IBD result via Wilcoxon rank-sum tests with BH correction.: [evidence span withheld — anchor status: not_found]
