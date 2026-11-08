---
name: dda-spectra-deconvolution
description: Use when processing centroided DDA LC-MS/MS data (e.g. mzML from a standards
  mixture or clinical sample) where chimeric spectra must be deconvolved before MS2-based
  compound identification in R.
when_to_use_negative:
- Input is SWATH-DIA data — use PerformDIADeconvolution (or reconstructChromPeakSpectra)
  instead, which relinks precursors with fragment ions from isolation windows
- Input is already a deconvoluted MS2 feature/annotation table rather than raw DDA
  spectra — re-deconvolution is unnecessary and the workflow starts from mzML
- The goal is MS1-only quantification without compound annotation, where MS2 processing
  adds no value
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalystR 4.0
  role: R package providing PerformDDADeconvolution (elastic-net deconvolution of
    chimeric DDA spectra), PerformSpectrumConsenus, PerformDBSearchingBatch and PerformResultsExport
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Open-source environment for running the MetaboAnalystR pipeline and paired
    statistical tests on matching scores
- name: ProteoWizard
  role: Used to convert raw vendor files to centroid-mode mzML before processing
- name: OptiLCMS
  role: Auto-optimized LC-MS1 feature detection and quantification dependency installed
    alongside MetaboAnalystR
  repo: https://github.com/xia-lab/OptiLCMS
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:19e5388df8584db2860b0eda3583475abd523dd75eee72e2367df10c94af25ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MS-DIAL
  - MS-FINDER
  - MZmine
  - MetaboAnalystR 4.0
  - OptiLCMS
  - ProteoWizard
  - R
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# dda-spectra-deconvolution

## Summary

Deconvolve chimeric DDA MS2 spectra in MetaboAnalystR 4.0 using an auto-optimized elastic-net model over candidate spectra, then build replicate consensus spectra and search them against MS2 reference libraries to improve compound identification and matching scores.

## When to use

Use when processing centroided DDA LC-MS/MS data (e.g. mzML from a standards mixture or clinical sample) where chimeric spectra must be deconvolved before MS2-based compound identification in R.

## When NOT to use

- Input is SWATH-DIA data — use PerformDIADeconvolution (or reconstructChromPeakSpectra) instead, which relinks precursors with fragment ions from isolation windows
- Input is already a deconvoluted MS2 feature/annotation table rather than raw DDA spectra — re-deconvolution is unnecessary and the workflow starts from mzML
- The goal is MS1-only quantification without compound annotation, where MS2 processing adds no value

## Inputs

- Raw centroid-mode LC-DDA-MS/MS data in mzML format (e.g. converted with ProteoWizard), such as the 526-standards mixture from Metabolomics Workbench accession ST002796 (ESI+)
- MS1 feature table produced by the auto-optimized feature detection step in MetaboAnalystR 4.0
- Curated MS2 reference spectra libraries in SQLite format (compiled from HMDB, MoNA, MassBank, GNPS, LipidBlast, MINEs)

## Outputs

- Deconvoluted, non-chimeric MS2 spectra reassigned to their correct precursor features
- Consensus MS2 spectra per feature across replicates (50% fragment frequency threshold)
- Compound identification table with matching scores (0-100 scale) from PerformResultsExport
- Benchmark comparison of identification rates with vs. without deconvolution (e.g. Table 2 CSV such as table2_dda_esiplus_comparison.csv)

## How to apply

Load centroid-mode mzML files into MetaboAnalystR 4.0 and detect/quantify MS1 features with the auto-optimized workflow, then run PerformDDADeconvolution, which assembles Candidate Spectra I-IV (orphan-isotopologue prediction, in-data clean spectra, library spectra, and transformation-network-predicted spectra) and fits a penalized elastic-net model over an auto-optimized grid of 11 alpha values (0 to 1 in 0.1 steps) x 10 estimated lambda values (110 alpha-lambda combinations), selecting the solution with minimal residue per peak. Merge MS2 spectra across replicates with PerformSpectrumConsenus using a 50% fragment frequency threshold (fragments below the threshold are removed). Annotate the consensus spectra with PerformDBSearchingBatch against the curated SQLite MS2 reference libraries, scoring matches as (MS2 similarity + MS1 similarity + RT similarity + 0.5 x isotope similarity)/3.5, scaled to 100, then export results with PerformResultsExport. To quantify the benefit, repeat the identical workflow without deconvolution and compare matching scores or correct-identification rates per compound.

## Related tools

- **MetaboAnalystR 4.0** (R package providing PerformDDADeconvolution (elastic-net deconvolution of chimeric DDA spectra), PerformSpectrumConsenus, PerformDBSearchingBatch and PerformResultsExport) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Open-source environment for running the MetaboAnalystR pipeline and paired statistical tests on matching scores)
- **ProteoWizard** (Used to convert raw vendor files to centroid-mode mzML before processing)
- **OptiLCMS** (Auto-optimized LC-MS1 feature detection and quantification dependency installed alongside MetaboAnalystR) — https://github.com/xia-lab/OptiLCMS

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual = T); mSet <- PerformDDADeconvolution(mSet)
```

## Evaluation signals

- Benchmark on the ST002796 standards mixture (ESI+, 526 standards) should reproduce the reported trend: MetaboAnalystR deconvoluted DDA gives 159/39.2% correctly annotated compounds vs 146/36.0% without deconvolution, exceeding MS-DIAL/MS-FINDER (82/20.2%) and MZmine/SIRIUS (77/19.0%) baselines
- Paired one-tailed Student's t-test over matching scores of n = 99 compounds (ESI+ mixture) should show a significant improvement for deconvolution, consistent with the reported Fig. 4a p = 1.5e-7
- Matching scores must be on the 0-100 scale computed as (MS2 + MS1 + RT + 0.5 x isotope similarity)/3.5 x 100
- Deconvolution should remove chimeric co-isolated fragments — the article notes over half of DDA spectra are chimeric and need to be deconvolved
- Consensus step should retain only fragments observed above the 50% frequency threshold across replicates

## Limitations

- Matching is bounded by reference-library quality: built-in MS2 spectra are compiled from various instruments and experimental conditions, resulting in potentially varying qualities
- MS2-based compound identification relies heavily on the availability of reference spectra databases, which is currently incomplete
- The elastic-net grid (11 alpha x 10 lambda = 110 combinations per peak) is computationally intensive; deconvolution must be re-run entirely to produce the no-deconvolution control for fair comparison
- Stable isotope-labeling analysis is not yet available in the package (future work, no code present)

## Evidence

- [intro] DDA chimeric spectra deconvolution: "over half of them are chimeric and need to be deconvolved"
- [methods] PerformDDADeconvolution function: "The deconvolution of DDA data can be achieved with the function, PerformDDADeconvolution."
- [methods] 50% fragment frequency consensus threshold: "If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed."
- [results] Benchmark numbers Table 2: [evidence span withheld — anchor status: not_found]
- [other] Paired t-test on matching scores (Fig. 4a): [evidence span withheld — anchor status: not_found]
- [other] Matching score formula: [evidence span withheld — anchor status: not_found]
- [readme] README claims on DDA/DIA identification gains: "For both DDA and DIA datasets, MetaboAnalystR 4.0 can increase the true positive rate of chemical identification by > 40% without increasing false identifications."
