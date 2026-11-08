---
name: spectrum-molecule-match-scoring
description: Use when experimental LC-MS/MS spectra (MGF, mzXML, or mzML) must be
  searched against a chemical database (e.g. NPAtlas, PubChem, COCONUT) to identify
  known molecules or previously unreported variants, especially when precursor mass
  deltas suggest molecular variants.
when_to_use_negative:
- The spectra have already been preprocessed (peak merging/filtering) elsewhere —
  then the advanced peak-processing defaults should be changed rather than left untouched,
  or a different pipeline is preferable.
- You need an exhaustive structural elucidation of unknown compounds rather than a
  database search that scores spectra against candidate molecules and their variants.
- Only single-charge [M+H]+ / [M-H]- adducts are relevant and a higher maximum absolute
  charge would introduce unwanted [M+2H]++ / [M-2H]-- adduct matches.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: VInSMoC
  role: Mass spectral search tool that scores query mass spectra against a database
    of candidate molecules, supporting exact and variable search modes; used to produce
    the best-hit-per-scan results and score-agnostic P values
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Web platform that hosts VInSMoC and handles data upload, parsing, project/data
    association, task submission, and results display
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:c8db3218e6fcb0fb1457568645ad9ce9db43f06dea21074e731fdac8d80e908d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# spectrum-molecule-match-scoring

## Summary

Scores query mass spectra against a candidate-molecule database via VInSMoC, which matches spectra to exact molecules or variants that explain precursor mass differences, and estimates match significance with a topology-preserving Metropolis–Hastings Markov chain Monte Carlo P value.

## When to use

Use when experimental LC-MS/MS spectra (MGF, mzXML, or mzML) must be searched against a chemical database (e.g. NPAtlas, PubChem, COCONUT) to identify known molecules or previously unreported variants, especially when precursor mass deltas suggest molecular variants.

## When NOT to use

- The spectra have already been preprocessed (peak merging/filtering) elsewhere — then the advanced peak-processing defaults should be changed rather than left untouched, or a different pipeline is preferable.
- You need an exhaustive structural elucidation of unknown compounds rather than a database search that scores spectra against candidate molecules and their variants.
- Only single-charge [M+H]+ / [M-H]- adducts are relevant and a higher maximum absolute charge would introduce unwanted [M+2H]++ / [M-2H]-- adduct matches.

## Inputs

- Query mass spectral files in MGF, mzXML, or mzML format (e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287)
- A chemical database of candidate molecules (NPAtlas, PubChem, COCONUT) available on run.npanalysis.org
- Task configuration: search mode (exact/variable), fragmentation mode (Metabolite/Peptide), ion mode, precursor and product ion tolerances, maximum absolute charge, minimum score

## Outputs

- Per-scan best hit list (saved e.g. as vinsmoc_hits.csv) reporting each input scan whose best hit exceeded the minimum score reporting threshold
- Score-agnostic P values for spectrum–molecule matches computed via Markov chain Monte Carlo simulation
- Links from each reported molecule to its entry in the source chemical database

## How to apply

Upload the mass spectral input (MGF, mzXML, or mzML) as a 'Mass Spectrum' data kind on run.npanalysis.org, wait for parsing to complete, and associate the parsed file plus the chemical database (e.g. NPAtlas) with a project. Create a VInSMoC task, selecting the spectrum file and database under 'Input Data'. In 'Task Parameters', choose the search mode — exact mode matches spectra to molecules whose masses match the precursor m/z, while variable mode matches spectra to molecules whose variants can explain the precursor mass difference Δ — then set the fragmentation mode (Metabolite for generic molecules, Peptide for generalized peptides), the ion mode matching the spectrum (positive or negative), precursor and product ion tolerances, the maximum absolute charge (1 restricts adducts to [M+H]+ or [M-H]-; 2 adds [M+2H]++ or [M-2H]--), and a minimum score that controls which hits are reported. Leave the advanced peak-merging and peak-filtering settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) at defaults unless the spectra were preprocessed before upload. Submit the task, monitor pending/running states, and read the results page, which reports the best hit per input scan exceeding the minimum score threshold. To assess significance, score-agnostic P values are estimated by a Metropolis–Hastings random walk over molecules of matched mass whose transition function is edge-switching — two edges chosen at random with endpoints randomly reconnected — so the P value controls for molecular topology rather than using an approximate Poisson binomial P value.

## Related tools

- **VInSMoC** (Mass spectral search tool that scores query mass spectra against a database of candidate molecules, supporting exact and variable search modes; used to produce the best-hit-per-scan results and score-agnostic P values) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Web platform that hosts VInSMoC and handles data upload, parsing, project/data association, task submission, and results display) — https://run.npanalysis.org

## Examples

```
Upload extract-3_Metal-mix_1mM.mzML as 'Mass Spectrum' at https://run.npanalysis.org, create a VInSMoC task with variable search mode, Metabolite fragmentation, positive ion mode, max charge 1, then submit.
```

## Evaluation signals

- Every reported hit row corresponds to an input scan whose best hit score exceeds the configured minimum score reporting threshold.
- Each reported molecule resolves to a valid entry in the source chemical database (molecule links on the task results page).
- Input data point status transitions from pending to successfully parsed before the task is launched; an error status indicates malformed input or a mismatch between file content and declared extension.
- Task state progresses through pending and running and ends in the finished state, at which the results page is readable.
- For score-agnostic significance, verify the P value was simulated via the Metropolis–Hastings random walk with edge-switching transitions over molecules of matched mass (topology preserved), not via an approximate Poisson binomial P value.

## Limitations

- The approximate Poisson binomial P value for spectrum–molecule matches does not control for molecular topology; only the MCMC-simulated score-agnostic P value addresses this.
- Task runtime can be long depending on input size, despite the tutorial example finishing almost immediately.
- Data inputs are not immediately usable — each data point must be parsed on the server before downstream tools can use it.
- Advanced peak-merging/peak-filtering settings should be left unchanged unless preprocessing was already done before upload.

## Evidence

- [other] Score-agnostic P value via edge-switching MCMC: [evidence span withheld — anchor status: not_found]
- [readme] Task parameters and defaults: [evidence span withheld — anchor status: not_found]
