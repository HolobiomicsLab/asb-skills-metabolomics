---
name: markov-chain-monte-carlo-significance-estimation
description: Decision support for disambiguating VInSMoC spectrum–molecule matches
  whose raw peak-count scores are equal or mis — especially when candidate topologies
  (linear vs cyclic) score identically and only statistical significance separates
  a true from a false hit.
when_to_use_negative:
- When raw peak-count scores alone already unambiguously rank the correct molecule
  far above all candidates (no score tie or bias to correct).
- When the analysis goal is variant discovery or variable-mode searching itself rather
  than significance assessment of already-computed matches — use VInSMoC's search
  modes directly instead.
- When input spectra have already been heavily preprocessed such that default Peak
  Merge Threshold / Peaks Per Window settings would distort the scoring used as MCMC
  input.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Scores query mass spectra against candidate molecules, computes the peak count
    score, and estimates the statistical significance of each spectrum–molecule match
    via its Markov chain Monte Carlo P value strategy
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosted server on which VInSMoC is run to obtain the MCMC P values for each
    spectrum–molecule couple
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:d63218aa0dd856f93f04444ef776cb578e2e74dec0184098161b31eac1005f4c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# markov-chain-monte-carlo-significance-estimation

## Summary

Estimate the statistical significance of mass spectrum–molecule matches in VInSMoC using a Markov chain Monte Carlo (MCMC) P value strategy, which corrects the raw peak-count score bias that favors, e.g., cyclic over linear macro-topologies when both receive equal scores (e.g. 18) but only the linear compound is correct.

## When to use

Decision support for disambiguating VInSMoC spectrum–molecule matches whose raw peak-count scores are equal or misleading — especially when candidate topologies (linear vs cyclic) score identically and only statistical significance separates a true from a false hit.

## When NOT to use

- When raw peak-count scores alone already unambiguously rank the correct molecule far above all candidates (no score tie or bias to correct).
- When the analysis goal is variant discovery or variable-mode searching itself rather than significance assessment of already-computed matches — use VInSMoC's search modes directly instead.
- When input spectra have already been heavily preprocessed such that default Peak Merge Threshold / Peaks Per Window settings would distort the scoring used as MCMC input.

## Inputs

- Experimental mass spectra of linear and cyclic macro-topology candidate molecules (MGF, mzXML, or mzML format), e.g. from Supplementary Fig. 18
- Candidate molecule structures (linear and cyclic topologies) to score the spectra against
- VInSMoC peak count scores for each spectrum–molecule couple

## Outputs

- MCMC-estimated P values for each spectrum–molecule match (e.g. 4.005 × 10^-19 for the correct linear compound, 8.900 × 10^-14 for the cyclic compound)
- Tabulated scores and P values, e.g. saved as vinsmoc_macrocycle_pvalues.csv
- Disambiguated best molecule identification per query spectrum based on statistical significance rather than raw score

## How to apply

After VInSMoC scores each query mass spectrum (MGF, mzXML, or mzML) against candidate molecules, run its Markov chain Monte Carlo P value estimation on each spectrum–molecule couple rather than trusting the raw peak count score alone. In the linear-vs-cyclic macro-topology benchmark (Supplementary Fig. 18), both candidates obtain a peak count score of 18, so the raw score cannot identify the correct linear compound; the MCMC P value strategy resolves this by assigning the correct linear compound a far smaller P value (4.005 × 10^-19) than the cyclic compound (8.900 × 10^-14). Operationally, obtain the paired spectra, run VInSMoC with default spectral preprocessing (peak merging within the Peak Merge Threshold while preserving at least Minimum Peak Count peaks, and peak filtering keeping the Peaks Per Window most intense peaks per Peak Filtering Window), compute the peak count score for each couple, then estimate each match's significance via the MCMC strategy. The output should be tabulated (e.g. as vinsmoc_macrocycle_pvalues.csv) and checked against the reference P values to confirm the bias correction worked — the cyclic compound's higher-or-equal raw score must not translate into a lower P value.

## Related tools

- **VInSMoC** (Scores query mass spectra against candidate molecules, computes the peak count score, and estimates the statistical significance of each spectrum–molecule match via its Markov chain Monte Carlo P value strategy) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosted server on which VInSMoC is run to obtain the MCMC P values for each spectrum–molecule couple) — https://run.npanalysis.org

## Evaluation signals

- Correct linear compound must receive a P value of 4.005 × 10^-19 and the cyclic compound 8.900 × 10^-14 (reproduction check).
- The cyclic compound's raw peak count score must equal (here 18) but not yield a smaller P value than the correct linear compound — confirming the bias correction.
- The computed P values must be saved and tabulated in vinsmoc_macrocycle_pvalues.csv and match the expected values within numerical tolerance.
- Spectra must pass VInSMoC's default preprocessing: peak merging leaves at least Minimum Peak Count peaks, and peak filtering keeps Peaks Per Window most intense peaks per Peak Filtering Window.
- Every spectrum–molecule couple evaluated must have both a peak count score and an MCMC P value reported — no couple should be missing a P value.

## Limitations

- The raw peak count score alone is misleading for topology disambiguation — it cannot separate equal-scoring linear vs cyclic candidates; only the MCMC P value corrects this bias.
- VInSMoC's P value estimation removes some but not all false identifications — it is a statistical filter, not a guarantee of correctness.
- Uploaded spectra must match the declared file format (MGF, mzXML, or mzML); malformed inputs fail parsing on the server.

## Evidence

- [results] MCMC P values disambiguate equal raw scores: [evidence span withheld — anchor status: not_found]
- [supplementary] Workflow definition for the benchmark: [evidence span withheld — anchor status: not_found]
