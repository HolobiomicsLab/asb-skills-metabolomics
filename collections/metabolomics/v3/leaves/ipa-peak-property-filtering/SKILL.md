---
name: ipa-peak-property-filtering
description: Use when a per-sample IDSL.IPA peak list (m/z–RT with isotopologue pairs
  and area) must be filtered against an IPDB formula database so only isotopic-profile-compatible
  candidates are scored and ranked by IDSL.UFA.
when_to_use_negative:
- The input is already a curated feature/annotation table with assigned formulas —
  re-filtering would discard valid annotations
- MS2 spectra are required for annotation (IDSL.UFA pipeline is MS1-only) and DDA/DIA
  deconvolution is the goal — use IDSL.CSA instead
- The chemical space cannot be bounded by any formula source and exhaustive elemental
  enumeration is needed — IDSL.UFAx is the appropriate tool
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: IDSL.IPA
  role: Generates the per-sample peaklists and aligned peak table that feed the filtering
    and ranking pipeline
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFA
  role: Executes the PCS/NEME/NDCS/RCS candidate filtering, equation (6) scoring,
    and formula ranking from the UFA parameter spreadsheet
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: R
  role: Runtime environment for both packages
  repo: https://cran.r-project.org/package=IDSL.UFA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:4a1a1df1732693d80e65358f08b07c3f4303f3ae976af62cdf7964511254e7c1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.IPA
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# ipa-peak-property-filtering

## Summary

Screens peaks in an IDSL.IPA per-sample peak list against candidate molecular formulas using the PCS, NEME, NDCS, and RCS isotopic-profile and isotopologue-pair criteria before IDSL.UFA scoring, so only chemically plausible formulas advance to the equation (6) ranking stage.

## When to use

Use when a per-sample IDSL.IPA peak list (m/z–RT with isotopologue pairs and area) must be filtered against an IPDB formula database so only isotopic-profile-compatible candidates are scored and ranked by IDSL.UFA.

## When NOT to use

- The input is already a curated feature/annotation table with assigned formulas — re-filtering would discard valid annotations
- MS2 spectra are required for annotation (IDSL.UFA pipeline is MS1-only) and DDA/DIA deconvolution is the goal — use IDSL.CSA instead
- The chemical space cannot be bounded by any formula source and exhaustive elemental enumeration is needed — IDSL.UFAx is the appropriate tool

## Inputs

- Per-sample peaklist csv from IDSL.IPA (peak m/z, RT, nIsoPair, peak area, correlated isotopologue intensities)
- IPDB theoretical isotopic profile database (.Rdata), e.g. RefMetDB positive or negative mode
- UFA parameter spreadsheet (UFA_parameters.xlsx)
- MS1-level HRMS data files (mzXML, mzML, netCDF)

## Outputs

- Per-peak table of surviving candidate molecular formulas with rank, equation (6) score, and IPA peak properties (nIsoPair, RCS, peak area)
- Aligned annotated peak table summarizing detection frequency and median ranks across samples
- Per-hit metrics: PCS, NEME, NDCS, RCS, and experimental vs theoretical R13C ratio

## How to apply

Run the IDSL.IPA workflow on raw mzXML/mzML/netCDF data to obtain per-sample peaklists, then feed them together with the IPDB (.Rdata) formula library into IDSL.UFA via the UFA parameter spreadsheet (PARAM0004 IPDB address, PARAM0011 peaklists directory, PARAM0012 peak_alignment directory). IDSL.UFA applies the candidate-screening filters — Profile cosine similarity (PCS), normalized Euclidean mass error (NEME), number of detected common isotopologue pairs (NDCS), and retention-of-candidates score (RCS) — each weighted with parameter and default value 1, to discard formulas whose theoretical isotopic profile or mass error is inconsistent with the observed peak. Surviving candidates then receive a composite score from equation (6) combining PCS, RCS, NEME, and the experimental-to-theoretical R13C ratio, and are ranked in descending order. The rationale is to prune a large chemical space (e.g. 209,592 positive-mode ion formulas in IDSL.ExposomeDB) down to isotopically and mass-defensible candidates before expensive scoring, and to keep the filter weights tunable so the stringency can be adjusted per instrument.

## Related tools

- **IDSL.IPA** (Generates the per-sample peaklists and aligned peak table that feed the filtering and ranking pipeline) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFA** (Executes the PCS/NEME/NDCS/RCS candidate filtering, equation (6) scoring, and formula ranking from the UFA parameter spreadsheet) — https://cran.r-project.org/package=IDSL.UFA
- **R** (Runtime environment for both packages) — https://cran.r-project.org/package=IDSL.UFA

## Examples

```
library(IDSL.UFA); UFA_workflow("UFA_parameters.xlsx") # PARAM0004=IPDB .Rdata, PARAM0011=peaklists, PARAM0012=peak_alignment, PARAM0014=output
```

## Evaluation signals

- Every surviving candidate formula in the output csv must carry a rank, score, and IPA-derived peak property column (nIsoPair, RCS, peak area)
- Reported annotations should satisfy the ranking criterion used in the paper: median rank ≤ 5 per aligned peak, and top rank ≤ 30 with number of hits ≤ 30 for unique molecular formulas
- PCS, NEME, NDCS, and RCS weights are all set to 1 in the default parameter spreadsheet; verify these defaults are present in PARAM settings
- Aligned-table coverage should reach a substantial fraction of peaks — the paper reported 199,837 unique molecular formulas on the aligned table under the top-rank ≤ 30 / hits ≤ 30 threshold
- Filter weights can be tightened or loosened by editing the parameter spreadsheet without changing the IPDB or raw data

## Limitations

- Formula assignment relies on MS1 data only; peaks lacking adequate isotopologue pairs (low nIsoPair) will have few surviving candidates
- Filter stringency depends on instrument mass accuracy — poor mass calibration inflates NEME and can drop true positives
- The pipeline presumes the IDSL.IPA peaklists and peak_alignment directories were produced with consistent RT correction across batches; inconsistent upstream processing degrades RCS
- 'Database dependent' formula sources may miss exposure compounds poorly represented in biochemical databases, per the article's critique of prior tools

## Evidence

- [results] Candidate filtering before equation (6) scoring: [evidence span withheld — anchor status: not_found]
- [results] Equation (6) score composition and default weights: [evidence span withheld — anchor status: not_found]
- [readme] Workflow entry points and required parameter fields: [evidence span withheld — anchor status: not_found]
