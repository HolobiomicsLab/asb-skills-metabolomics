---
name: mass-spectral-database-search
description: Use when experimental tandem MS spectra need to be matched to known molecules
  or previously unreported molecule variants from databases such as PubChem, COCONUT,
  or NPAtlas, especially at atlas scale (e.g. 483M GNPS spectra vs 87M molecules).
when_to_use_negative:
- The spectra have already been preprocessed locally — in that case the README says
  the advanced peak-merging and peak-filtering settings should be changed rather than
  left at defaults, and blindly applying defaults may double-preprocess peaks.
- The goal is de novo sequencing or structural elucidation without any candidate molecule
  database — this skill scores spectra against a fixed candidate database and cannot
  produce identifications absent a molecular library.
- The input data kind is not mass spectra (e.g. already a feature table or metabolite
  intensity matrix) — VInSMoC requires MGF, mzXML, or mzML spectral inputs uploaded
  as 'Mass Spectrum' data.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: VInSMoC
  role: Mass spectral search tool that scores query mass spectra against a database
    of candidate molecules, running in exact or variable search mode on the run.npanalysis.org
    server.
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: 'Hosts VInSMoC as a task: account signup, project creation, data upload/parsing,
    task submission, and monitoring through pending/running states to finished results.'
  repo: https://run.npanalysis.org
merged_aliases:
- mass-spectral-library-search
merged_alias_records:
- alias: mass-spectral-library-search
  slug: mass-spectral-library-search
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  - task_007
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:9a4b880b3fb1a6da8a12f87e1a81eb67dec590561c5c5508835556bc79ef3408
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
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mass-spectral-database-search

## Summary

Scores query mass spectra (MGF, mzXML, mzML) against a database of candidate molecules using VInSMoC on the run.npanalysis.org server, matching precursor masses in exact mode or explaining precursor mass differences via molecule variants in variable mode, with Markov chain Monte Carlo P values to control false identifications.

## When to use

Use when experimental tandem MS spectra need to be matched to known molecules or previously unreported molecule variants from databases such as PubChem, COCONUT, or NPAtlas, especially at atlas scale (e.g. 483M GNPS spectra vs 87M molecules).

## When NOT to use

- The spectra have already been preprocessed locally — in that case the README says the advanced peak-merging and peak-filtering settings should be changed rather than left at defaults, and blindly applying defaults may double-preprocess peaks.
- The goal is de novo sequencing or structural elucidation without any candidate molecule database — this skill scores spectra against a fixed candidate database and cannot produce identifications absent a molecular library.
- The input data kind is not mass spectra (e.g. already a feature table or metabolite intensity matrix) — VInSMoC requires MGF, mzXML, or mzML spectral inputs uploaded as 'Mass Spectrum' data.

## Inputs

- Query mass spectra in MGF, mzXML, or mzML format (uploaded as 'Mass Spectrum' data kind)
- Chemical database of candidate molecules (e.g. NPAtlas, COCONUT, PubChem-derived; all public developer-provided databases are automatically available to every project)
- Precursor and product ion tolerance settings (defaults if unsure)
- Ion mode matching the input spectra (positive or negative)

## Outputs

- Task results page listing the best hit per input scan that exceeded the minimum score reporting threshold, with each molecule linked to its entry in the original chemical database
- Variable-mode variant identifications with mass differences between each hit and its database parent molecule (e.g. ~85,151 unique variants from COCONUT variable mode)
- Statistical significance of spectrum-molecule matches via Markov chain Monte Carlo P values for false identification removal

## How to apply

Create an account and project on run.npanalysis.org, then upload the mass spectral input (MGF, mzXML, or mzML) as a 'Mass Spectrum' data kind and wait for server-side parsing to complete before associating the spectra and a chemical database (e.g. NPAtlas, COCONUT, PubChem-derived) to the project. Create a VInSMoC task, select the spectrum file and database under 'Input Data', and configure 'Task Parameters': search mode 'exact' to match precursor mass or 'variable' to explain precursor-vs-molecule mass differences; fragmentation mode Metabolite for generic molecules or Peptide for generalized peptides; ion mode matching the input spectra; precursor and product ion tolerances (defaults are safe if unsure); maximum absolute charge of 1 for [M+H]+/[M-H]- adducts or 2 to also allow doubly-charged adducts; and a minimum score controlling which hits are reported. Leave the advanced peak preprocessing at defaults (peak merging within Peak Merge Threshold preserving at least Minimum Peak Count peaks; peak filtering keeping Peaks Per Window most intense peaks per Peak Filtering Window) unless the spectra were already preprocessed locally. Submit the task, monitor it through pending and running states until finished, and retrieve from the task page the best hit per input scan exceeding the minimum score threshold; the runtime can be benchmarked — exact-mode atlas-scale search of GNPS vs PubChem (6.28 trillion matches) took 305.12 hours on 128 threads versus ~30 years estimated for Dereplicator+. Validation includes confirming each reported molecule links to its entry in the source chemical database and, for variable-mode variant calls, tabulating high-frequency mass shifts (e.g. -2.0 oxidation, ±14.0 methylation, -16.0 deoxygenation) and recomputing false discovery rate curves as score/P-value thresholds are tightened.

## Related tools

- **VInSMoC** (Mass spectral search tool that scores query mass spectra against a database of candidate molecules, running in exact or variable search mode on the run.npanalysis.org server.) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosts VInSMoC as a task: account signup, project creation, data upload/parsing, task submission, and monitoring through pending/running states to finished results.) — https://run.npanalysis.org

## Evaluation signals

- Every task reaches the finished state after pending/running, and the task page shows the best hit for each input scan whose score exceeded the minimum score threshold.
- Each reported molecule in the results resolves to a link to its entry in the original chemical database it came from.
- For a published-benchmark reproduction, an exact-mode atlas-scale search yields ~43,598 unique known-molecule identifications at minimum score 30 with runtime in the order of the reported 305.12 hours on 128 threads (vs ~30 years estimated for Dereplicator+).
- For variable mode on COCONUT, the mass-shift distribution of hit-to-parent mass differences is dominated by known chemical shifts such as oxidation (-2.0/−15.9949 Da), methylation (±14.0/+14.0157 Da), and deoxygenation (-16.0 Da).
- False discovery rate curves recomputed while sweeping the score cutoff and MCMC P-value cutoff show decreasing FDR as thresholds are tightened.

## Limitations

- Server-side data parsing is asynchronous — uploaded spectra are 'not immediately ready for use' and malformed inputs return an error status, so the declared extension must match the actual format.
- Exact search is limited to known molecules whose masses match the query precursor; explaining unreported variants requires variable mode, which additionally allows precursor mass differences Δ to be explained by molecule variants.
- Advanced peak preprocessing defaults (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) should be left unchanged unless spectra were already preprocessed before upload.
- Runtime depends on input size — the small example finishes almost immediately, but atlas-scale searches can take days of wall-clock time even with 128 threads.

## Evidence
