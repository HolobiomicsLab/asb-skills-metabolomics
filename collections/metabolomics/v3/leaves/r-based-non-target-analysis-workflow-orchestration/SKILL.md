---
name: r-based-non-target-analysis-workflow-orchestration
description: Use when processing LC- or GC-HRMS mzML data for environmental non-target
  screening and you need one R-based pipeline covering feature extraction, blank/replicate/intensity
  filtering, and formula/compound annotation with swappable algorithms.
when_to_use_negative:
- When a reference standard exists and a simple targeted quantification suffices —
  patRoon targets non-target screening of unknowns/CECs where standards are unavailable
- When data is already a cleaned feature table — the skill starts from raw/vendor
  HRMS data pre-treatment (format conversion, centroiding)
- When fully open-source reproducibility of every algorithm is required — some integrated
  algorithms are closed-source
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: patRoon
  role: 'R package providing the harmonized NTA workflow interface: feature finding,
    filter(), formula and compound annotation'
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Companion R package supplying example LC-HRMS data (standard-X / solvent-X)
    for testing and demonstrating the workflow
  repo: https://github.com/rickhelmus/patRoonData
- name: XCMS
  role: Optional feature finding/grouping backend invoked through patRoon's consistent
    interface
- name: OpenMS
  role: Optional feature finding/grouping backend invoked through patRoon
- name: enviPick
  role: Optional feature finding/grouping backend invoked through patRoon
- name: GenForm
  role: Molecular formula candidate calculation for filtered features using accurate
    mass and MS/MS
- name: SIRIUS
  role: Alternative formula/compound annotation engine usable in place of GenForm/MetFrag
- name: MetFrag
  role: In silico compound annotation and ranking against a chemical database (CompTox/PubChem/ChemSpider)
- name: ProteoWizard
  role: msconvert for vendor-to-mzML raw data pre-treatment before feature finding
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:a781aa2a2ec71b5d94af25a4fc4ce43769123a06916a5283628a3d2087a1c3c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - GenForm
  - MetFrag
  - OpenMS
  - ProteoWizard
  - R
  - SIRIUS
  - XCMS
  - enviPick
  - patRoon
  - patRoonData
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# r-based-non-target-analysis-workflow-orchestration

## Summary

Orchestrate a complete environmental non-target analysis (NTA) pipeline in R with patRoon, which harmonizes feature finding, filtering, formula and compound annotation algorithms (XCMS/OpenMS/enviPick, GenForm/SIRIUS, MetFrag) behind one consistent interface on LC-HRMS data.

## When to use

Use when processing LC- or GC-HRMS mzML data for environmental non-target screening and you need one R-based pipeline covering feature extraction, blank/replicate/intensity filtering, and formula/compound annotation with swappable algorithms.

## When NOT to use

- When a reference standard exists and a simple targeted quantification suffices — patRoon targets non-target screening of unknowns/CECs where standards are unavailable
- When data is already a cleaned feature table — the skill starts from raw/vendor HRMS data pre-treatment (format conversion, centroiding)
- When fully open-source reproducibility of every algorithm is required — some integrated algorithms are closed-source

## Inputs

- LC/GC-HRMS raw sample analyses (converted to mzML), including blanks, replicates and spiked samples
- patRoonData example LC-HRMS data (triplicate standard mixture 'standard-X' and blank solvent 'solvent-X', filtered to first 10 minutes, m/z 75-450)
- Local chemical database file for compound annotation (CompTox CSV, or PubChem/ChemSpider access)
- Optional suspect list (e.g. from NORMAN Suspect List Exchange)

## Outputs

- Grouped feature table with per-feature MS/MS peak lists after blank, replicate and intensity filtering (~964 feature groups in the demonstration)
- Molecular formula annotations per feature (GenForm or SIRIUS) with candidate rankings
- Compound annotations and rankings per feature (MetFrag scored against MS/MS peaks), exportable to feature_annotations.csv
- Componentized feature groups (adducts, isotopes, homologs) and estimated ID confidence levels

## How to apply

Load the HRMS sample analyses into R via patRoon (example data via the patRoonData companion package), convert vendor raw files to mzML with ProteoWizard if needed, and find/group features with one of the harmonized algorithms (XCMS, OpenMS or enviPick), retaining MS/MS peak lists per feature. Run the rule-based filter() pipeline in the article's order — blank filter, replicate (minimum replicate abundance) filter, then minimum intensity filter — which in the demonstration reduced ~19000 feature groups to 4719 (-76%), 2933 (-85%) and 964 (-95%); order matters because an intensity pre-filter ensures good quality feature data for subsequent filters, and the replicate filter may need repeating if earlier filters changed results. Annotate formulae with GenForm or SIRIUS using accurate mass plus MS/MS spectra, then rank compound candidates with MetFrag against a chemical database (CompTox via local CSV, or PubChem/ChemSpider). If a known suspect is missing (e.g. aniline), lower the intensity threshold of filter() and re-run rather than assuming absence. Because patRoon exposes all algorithms under a consistent interface, you can mix and evaluate different feature-finding backends without rewriting the workflow.

## Related tools

- **patRoon** (R package providing the harmonized NTA workflow interface: feature finding, filter(), formula and compound annotation) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Companion R package supplying example LC-HRMS data (standard-X / solvent-X) for testing and demonstrating the workflow) — https://github.com/rickhelmus/patRoonData
- **XCMS** (Optional feature finding/grouping backend invoked through patRoon's consistent interface)
- **OpenMS** (Optional feature finding/grouping backend invoked through patRoon)
- **enviPick** (Optional feature finding/grouping backend invoked through patRoon)
- **GenForm** (Molecular formula candidate calculation for filtered features using accurate mass and MS/MS)
- **SIRIUS** (Alternative formula/compound annotation engine usable in place of GenForm/MetFrag)
- **MetFrag** (In silico compound annotation and ranking against a chemical database (CompTox/PubChem/ChemSpider))
- **ProteoWizard** (msconvert for vendor-to-mzML raw data pre-treatment before feature finding)

## Examples

```
install.packages("remotes"); remotes::install_github("rickhelmus/patRoon"); remotes::install_github("rickhelmus/patRoonData")
```

## Evaluation signals

- Filter cascade reduces feature groups in the demonstrated range: ~19000 -> 4719 (-76%, replicate) -> 2933 (-85%, blank) -> 964 (-95%, intensity)
- Known spiked suspects are recovered: 17 of 19 suspects annotated with correct compound as first candidate; correct formula ranked 1st for 13 of 16 and within top 5 for 19 of 19 (100%) compounds
- Each retained feature carries both formula and compound candidate rankings; export to feature_annotations.csv succeeds
- Swapping the feature-finding backend (XCMS/OpenMS/enviPick) under the same script still yields a complete feature table — evidence the harmonized interface is working
- Any missed low-abundance suspect (e.g. aniline) becomes detectable when the filter() intensity threshold is lowered, confirming the filter is the cause, not the annotation step

## Limitations

- The missing suspect (aniline) was not detected at default settings — attributable to the intensity threshold of the filter() function
- Some integrated workflow algorithms are closed-source, limiting full open reproducibility
- No patRoon software version or dependency/algorithm versions are stated in the paper; versions are deferred to external installation instructions
- Benchmark source code and demonstration scripts are only available as supplementary archives, not in the main repository

## Evidence

- [abstract] abstract: "harmonizing various common (primarily open) software tools under a consistent interface"
- [intro] intro: [evidence span withheld — anchor status: not_found]
- [results] results: "the missing suspect (aniline) could be detected when lowering the intensity threshold of the filter() function"
- [readme] readme: [evidence span withheld — anchor status: not_found]
