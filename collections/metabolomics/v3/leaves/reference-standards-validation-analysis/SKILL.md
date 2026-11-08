---
name: reference-standards-validation-analysis
description: Use when validating formula-annotation accuracy on authentic reference
  standards (e.g., the MSV000088661 dataset) and you need quantitative NEME (mDa)
  and isotopic profile cosine similarity (PCS %) metrics to compare against published
  benchmarks.
when_to_use_negative:
- When the goal is structural or MS2-level annotation rather than MS1 formula-assignment
  accuracy — this skill only measures isotopic profile matching, not tandem-MS or
  structural confidence
- When input is raw unprocessed LC/HRMS with no peaklists — peaks must first be extracted
  with the IDSL.IPA workflow and annotated by IDSL.UFA before NEME/PCS can be computed
- When no authentic standard compounds or deposited isotopic profiles exist for the
  analysis — validation requires matched theoretical and experimental profiles
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL.UFA
  role: R package that assigns molecular formulas via MS1 isotopic profile matching;
    its annotation output and theoretical isotopic profiles (IPDB) are the objects
    being validated
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Generates the chromatographic peak information (m/z-RT peaklists) that IDSL.UFA
    annotates, upstream of the validation metrics
  repo: github.com/idslme/IDSL.IPA
- name: R
  role: Language used to compute NEME and PCS per compound and aggregate means across
    standards
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:939446d303f71aaee125d14eed48d6d787901ead5b69e62804abf5e12003748a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# reference-standards-validation-analysis

## Summary

Benchmarks IDSL.UFA molecular formula assignments against authentic reference standard compounds by computing normalized Euclidean mass error (NEME) and profile cosine similarity (PCS) between experimental and theoretical MS1 isotopic profiles.

## When to use

Use when validating formula-annotation accuracy on authentic reference standards (e.g., the MSV000088661 dataset) and you need quantitative NEME (mDa) and isotopic profile cosine similarity (PCS %) metrics to compare against published benchmarks.

## When NOT to use

- When the goal is structural or MS2-level annotation rather than MS1 formula-assignment accuracy — this skill only measures isotopic profile matching, not tandem-MS or structural confidence
- When input is raw unprocessed LC/HRMS with no peaklists — peaks must first be extracted with the IDSL.IPA workflow and annotated by IDSL.UFA before NEME/PCS can be computed
- When no authentic standard compounds or deposited isotopic profiles exist for the analysis — validation requires matched theoretical and experimental profiles

## Inputs

- MSV000088661 authentic reference standards LC/HRMS dataset (mzXML/mzML/netCDF MS1 data)
- Deposited IDSL.UFA output with theoretical and experimental integrated isotopic profile spectra from Zenodo record 5803968
- Per-compound IDSL.UFA annotation results (assigned molecular formulas, monoisotopic masses)

## Outputs

- Per-compound table of NEME (mDa) and PCS (%) values for 367 authentic standard compounds
- Aggregate mean NEME (~0.70 mDa) and aggregate mean PCS (~99.968%) across all compounds

## How to apply

Download the MSV000088661 authentic reference standards dataset together with the deposited IDSL.UFA output from Zenodo accession https://zenodo.org/record/5803968, which contains the theoretical and experimental integrated isotopic profile spectra across chromatography for each standard. Load the per-compound IDSL.UFA annotation results and extract the paired theoretical and experimental isotopic profiles across chromatography. For each compound's assigned formula, compute the NEME (mean error of the monoisotopic mass in mDa) and the pairwise cosine similarity (PCS) between theoretical and experimental isotopic profiles. Assemble a per-compound table of NEME and PCS values and aggregate the means across all compounds. Correctness is judged by aggregate mean NEME close to ~0.70 mDa and aggregate mean PCS close to ~99.968% over 367 authentic standard compounds. Because IDSL.UFA annotation requires only MS1, this validation exercises the isotopic profile matching approach directly against raw MS1 spectral data.

## Related tools

- **IDSL.UFA** (R package that assigns molecular formulas via MS1 isotopic profile matching; its annotation output and theoretical isotopic profiles (IPDB) are the objects being validated) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Generates the chromatographic peak information (m/z-RT peaklists) that IDSL.UFA annotates, upstream of the validation metrics) — github.com/idslme/IDSL.IPA
- **R** (Language used to compute NEME and PCS per compound and aggregate means across standards)

## Examples

```
library(IDSL.UFA); UFA_workflow("UFA_parameters.xlsx")  # then load Zenodo record 5803968 profiles and compute NEME/PCS per standard
```

## Evaluation signals

- Aggregate mean NEME across all compounds is close to the reported ~0.70 mDa
- Aggregate mean PCS across all compounds is close to the reported ~99.968%
- Per-compound table covers all 367 authentic standard compounds with both NEME and PCS values
- Each compound's assigned formula produces a theoretical isotopic profile that can be paired with an experimental integrated profile across chromatography
- Monositopic mass errors are reported in mDa, consistent with HRMS (Orbitrap-class) accuracy expectations

## Limitations

- Validation reflects MS1-only isotopic profile matching; it does not certify structural identity of the annotated formulas
- Results depend on the IPDB isotopic-profile database version and IDSL.UFA parameters, which are not stated in the deposited reproducibility details
- Benchmark values (~0.70 mDa NEME, ~99.968% PCS) are specific to the 367-compound MSV000088661 standards set and may not transfer to other instrument platforms or compound classes

## Evidence

- [other] Across 367 authentic standard compounds, IDSL.UFA achieved an average NEME of 0.70 mDa and an average PCS of 99.968% between experimental and theoretical isotopic profiles.: [evidence span withheld — anchor status: not_found]
- [readme] UFA isotopic profile matching approach: [evidence span withheld — anchor status: external_doc]
