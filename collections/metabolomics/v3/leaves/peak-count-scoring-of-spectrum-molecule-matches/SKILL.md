---
name: peak-count-scoring-of-spectrum-molecule-matches
description: Use when scoring query mass spectra (MGF, mzXML, or mzML) against candidate
  molecules in VInSMoC and you need the raw peak count score that determines which
  spectrum-molecule couples exceed the minimum score reporting threshold.
when_to_use_negative:
- You need statistically calibrated significance rather than a raw ranking score —
  peak count scores are biased (e.g. cyclic compounds can out-score correct linear
  matches at equal peak counts), so use the MCMC P value strategy instead.
- Your spectra have already been peak-merged or peak-filtered upstream — the default
  Peak Merge Threshold / Peaks Per Window preprocessing would double-process them;
  adjust advanced settings accordingly.
- The candidate molecule's precursor mass cannot be matched and no molecular variant
  can explain the mass difference Δ — no peak count score will be produced for that
  couple.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: VInSMoC
  role: Scores query mass spectra against a database of candidate molecules and reports
    the peak count score for each spectrum-molecule match above the minimum score
    threshold
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosts VInSMoC; spectrum files and chemical databases are uploaded, associated
    to a project, and scored through a VInSMoC task created on the server
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:624ce36e27e9063790f731d9db695a6a11967f94df987fbba2e7bc513bf5317c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-count-scoring-of-spectrum-molecule-matches

## Summary

Compute the peak count score that VInSMoC uses to rank spectrum-molecule matches: count experimental peaks that align to theoretical fragments of a candidate molecule after the standard peak-merge and peak-filter preprocessing. This score is the raw ranking signal whose bias (e.g. toward cyclic topologies) is later corrected by MCMC P values.

## When to use

Use when scoring query mass spectra (MGF, mzXML, or mzML) against candidate molecules in VInSMoC and you need the raw peak count score that determines which spectrum-molecule couples exceed the minimum score reporting threshold.

## When NOT to use

- You need statistically calibrated significance rather than a raw ranking score — peak count scores are biased (e.g. cyclic compounds can out-score correct linear matches at equal peak counts), so use the MCMC P value strategy instead.
- Your spectra have already been peak-merged or peak-filtered upstream — the default Peak Merge Threshold / Peaks Per Window preprocessing would double-process them; adjust advanced settings accordingly.
- The candidate molecule's precursor mass cannot be matched and no molecular variant can explain the mass difference Δ — no peak count score will be produced for that couple.

## Inputs

- Mass spectrum file in MGF, mzXML, or mzML format (e.g. extract-3_Metal-mix_1mM.mzML from MSV000086287)
- Chemical database of candidate molecules (e.g. NPAtlas, or PubChem/COCONUT for large searches)
- VInSMoC task configuration: search mode (exact/variable), fragmentation mode (Metabolite/Peptide), ion mode, precursor and product ion tolerances, maximum absolute charge, minimum score

## Outputs

- Peak count score per spectrum-molecule couple for scans exceeding the minimum score threshold
- Best hit per input scan with a link to the molecule's entry in the original chemical database
- VInSMoC task result page (task ID) listing hits; can be exported as e.g. vinsmoc_macrocycle_pvalues.csv for tabulated scores

## How to apply

Run VInSMoC on the mass spectrum file against the chosen chemical database (e.g. NPAtlas) and let the default spectral preprocessing apply: peak merging combines peaks within the Peak Merge Threshold of one another while leaving at least the Minimum Peak Count peaks, and peak filtering keeps only the Peaks Per Window most intense peaks in m/z windows of the Peak Filtering Window. For each candidate molecule whose mass matches the query precursor m/z (exact mode) or whose variants explain the precursor mass difference (variable mode), score the couple by the number of experimental peaks matched to theoretical fragments under the precursor and product ion tolerances, using the adduct dictated by the maximum absolute charge and ion mode. The peak count score is the reported value used to filter hits at the minimum score threshold, and each molecule hit is linked back to its entry in the source chemical database. Note that equal peak counts can occur for structurally different molecules (e.g. linear and cyclic topologies can both score 18), so the score alone cannot disambiguate ranking ties; statistical significance via MCMC P values is the complementary correction. If you have already preprocessed your spectra (merged or filtered peaks), override the advanced settings, otherwise leave Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, and Peak Filtering Window at their defaults.

## Related tools

- **VInSMoC** (Scores query mass spectra against a database of candidate molecules and reports the peak count score for each spectrum-molecule match above the minimum score threshold) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosts VInSMoC; spectrum files and chemical databases are uploaded, associated to a project, and scored through a VInSMoC task created on the server) — https://run.npanalysis.org

## Evaluation signals

- Every reported scan in the task result page has a hit whose score exceeds the configured minimum score threshold.
- Recomputed peak counts after preprocessing satisfy the invariant that merged spectra retain at least Minimum Peak Count peaks and at most Peaks Per Window peaks per Peak Filtering Window.
- Matched fragments fall within the configured precursor and product ion tolerances, and m/z-to-mass conversion uses the adduct consistent with the maximum absolute charge (e.g. [M+H]+ for charge 1 in positive mode).
- For the linear-vs-cyclic benchmark, both compounds score 18 on peak count while their significance differs — confirming the score is topology-insensitive as expected.
- Each reported molecule hit resolves to a valid entry in the source database (e.g. NPAtlas) via the link on the task page.

## Limitations

- The peak count score is not topology-aware: linear and cyclic compounds with the same number of matched peaks (e.g. 18) receive identical scores, so ranking ties require the MCMC P value correction.
- Scores are only reported for hits exceeding the minimum score; below-threshold couples produce no output.
- Raw peak count scores are misleading when molecule variants change the theoretical fragment set — the score alone cannot distinguish true variant matches from false identifications.
- Task preprocessing defaults (Peak Merge Threshold, Minimum Peak Count, Peaks Per Window, Peak Filtering Window) assume unprocessed spectra; user-preprocessed input requires overriding these settings.

## Evidence

- [other] Equal raw scores for different topologies: [evidence span withheld — anchor status: not_found]
