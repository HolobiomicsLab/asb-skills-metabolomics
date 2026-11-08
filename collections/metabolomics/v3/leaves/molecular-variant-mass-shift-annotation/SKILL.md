---
name: molecular-variant-mass-shift-annotation
description: Use when a GNPS-format mass spectral dataset (MGF/mzXML/mzML) has been
  searched with VInSMoC variable mode and the goal is to explain precursor-vs-molecule
  mass differences as known chemical mass shifts and quantify FDR across score thresholds.
when_to_use_negative:
- When only exact-match identifications of known molecules are needed — use VInSMoC
  exact mode, which matches spectra to molecules whose masses match the precursor
  m/z rather than explaining a mass difference
- When the input spectra have already been pre-processed locally — the README says
  advanced preprocessing settings 'should be left unchanged unless you have already
  down some preprocessing on your input spectra before uploading to our server'
- When the analysis requires biosynthesis pathway reconstruction (e.g. promothiocin
  B / depsidomycin in Streptomyces) rather than mass-shift annotation of variable-mode
  hits
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: VInSMoC
  role: Mass spectral search tool run in variable mode on the npanalysis.org server;
    scores query mass spectra against a database of candidate molecules whose variants
    can explain the precursor mass difference
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: 'Hosts VInSMoC: account creation, project organization, data upload/parsing
    of MGF/mzXML/mzML, task submission and monitoring'
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:812feb8aadd567d8f0f742a56fecd03ae3ab27af355193cd0de7078f6ec0b85f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# molecular-variant-mass-shift-annotation

## Summary

Run VInSMoC in variable mode against a chemical database (e.g. COCONUT) on the run.npanalysis.org server, collect the mass difference between each hit and its parent molecule, and interpret high-frequency shifts as known chemical transformations (oxidation +15.9949 Da, methylation +14.0157 Da, deoxygenation -15.9949 Da) while sweeping score/P-value cutoffs to build false discovery rate curves.

## When to use

Use when a GNPS-format mass spectral dataset (MGF/mzXML/mzML) has been searched with VInSMoC variable mode and the goal is to explain precursor-vs-molecule mass differences as known chemical mass shifts and quantify FDR across score thresholds.

## When NOT to use

- When only exact-match identifications of known molecules are needed — use VInSMoC exact mode, which matches spectra to molecules whose masses match the precursor m/z rather than explaining a mass difference
- When the input spectra have already been pre-processed locally — the README says advanced preprocessing settings 'should be left unchanged unless you have already down some preprocessing on your input spectra before uploading to our server'
- When the analysis requires biosynthesis pathway reconstruction (e.g. promothiocin B / depsidomycin in Streptomyces) rather than mass-shift annotation of variable-mode hits

## Inputs

- Mass spectral input file in MGF, mzXML, or mzML format (e.g. a file from MSV000086287)
- Chemical database to search against (COCONUT; NPAtlas is also available on the server)
- VInSMoC task parameters: search mode=variable, ion mode (positive), maximum absolute charge (1 or 2), precursor/product ion tolerances, minimum score

## Outputs

- Task results page listing the best hit per input scan exceeding the minimum score, with links to each molecule's database entry
- Per-hit mass difference between each hit and its database parent molecule
- Aggregated set of unique variant identifications (~85,151 from COCONUT variable mode) with tabulated mass-shift frequencies
- mass_shift_distribution.png (distribution of high-frequency mass shifts)
- fdr_curves.png (false discovery rate as score/P-value cut-offs are swept)

## How to apply

Upload the mass spectral input (MGF, mzXML, or mzML, e.g. from MSV000086287) to run.npanalysis.org, wait for server-side parsing to finish, and associate both the spectra and the COCONUT database with a project. Create a VInSMoC task with search mode set to 'variable', ion mode matching the input spectra (positive), maximum absolute charge 1 (giving [M+H]+/[M-H]-) or 2 (adding doubly-charged adducts), default precursor/product ion tolerances, and a minimum score controlling reported hits; leave the advanced preprocessing settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) unchanged when spectra were not pre-processed locally. From the finished task page, take the best hit per input scan exceeding the minimum score along with the mass difference between each hit and its database parent molecule. Aggregate the unique variant identifications (about 85,151 from COCONUT variable mode), tabulate the frequency of shifts such as oxidation (+15.9949 Da), methylation (+14.0157 Da), and deoxygenation (-15.9949 Da), and plot mass_shift_distribution.png. Then recompute hit lists while sweeping the score cut-off and the MCMC-estimated P-value cut-off and plot FDR curves (fdr_curves.png); VInSMoC estimates statistical significance of spectrum-molecule matches with a Markov chain Monte Carlo strategy to remove false identifications.

## Related tools

- **VInSMoC** (Mass spectral search tool run in variable mode on the npanalysis.org server; scores query mass spectra against a database of candidate molecules whose variants can explain the precursor mass difference) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosts VInSMoC: account creation, project organization, data upload/parsing of MGF/mzXML/mzML, task submission and monitoring) — https://run.npanalysis.org

## Evaluation signals

- The reproduced mass-shift distribution is dominated by the same high-frequency transformations reported in Extended Data Fig. 6/7: oxidation (+15.9949 Da), methylation (+14.0157 Da), deoxygenation (-15.9949 Da)
- FDR curves show monotonically decreasing false discovery rate as score and MCMC-estimated P-value thresholds are tightened, qualitatively matching Extended Data Fig. 7
- Every reported hit comes from an input scan whose score exceeded the minimum score reporting threshold; unmatched scans are excluded
- Uploaded data points show a 'successfully parsed' status before task submission; malformed inputs show an error status requiring format/extension re-check
- The count of unique COCONUT variable-mode identifications is in the same order as the reported ~85,151 (~85,000) previously unreported variants

## Limitations

- Variable-mode identifications are variants whose exact structures may be previously unreported, so mass-shift assignments are putative (e.g. ±14.0 corresponds to methylation only by frequency correspondence to known shifts)
- Peak merging and peak filtering settings (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) must not be changed unless spectra were pre-processed locally, or scoring behavior differs from the reference run
- Task execution time depends on input size; tasks may spend extended time in the 'Running' state
- Only [M+H]+/[M-H]- adducts are considered with maximum absolute charge 1; doubly-charged adducts require charge 2, otherwise some spectra cannot be matched

## Evidence
