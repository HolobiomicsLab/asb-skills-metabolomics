---
name: mass-tolerance-library-filtering
description: Use when performing library matching against an augmented spectral library
  where the query molecule's molecular mass is known with some accuracy, and the full
  library search is too large or introduces false candidates outside the plausible
  mass window.
when_to_use_negative:
- When the molecular mass of the query sample cannot be determined with a few-Dalton
  accuracy — the filter depends on knowing the mass and would incorrectly exclude
  the true candidate
- When evaluating predictors or similarity metrics on a fully unfiltered search where
  mass information is deliberately held out, since filtering confounds the comparison
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: NEIMS
  role: Provides the library-matching framework whose mass-filtering option prunes
    the augmented library by molecular-mass tolerance before similarity search
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Used to generate the molecular representations from which molecular masses
    of library and query molecules are obtained for filtering
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:69d39e4d8bb3e1452206e03f7b3aeb8008b957184fdee5f145e789748ef0d976
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Vizier
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# mass-tolerance-library-filtering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Restrict the NEIMS library-matching candidate set to spectra whose molecular mass falls within a 5 Da tolerance of the query molecule's mass before computing weighted cosine similarity, shrinking the search space to a median of 6696 spectra per query and improving recall@10.

## When to use

Use when performing library matching against an augmented spectral library where the query molecule's molecular mass is known with some accuracy, and the full library search is too large or introduces false candidates outside the plausible mass window.

## When NOT to use

- When the molecular mass of the query sample cannot be determined with a few-Dalton accuracy — the filter depends on knowing the mass and would incorrectly exclude the true candidate
- When evaluating predictors or similarity metrics on a fully unfiltered search where mass information is deliberately held out, since filtering confounds the comparison

## Inputs

- Augmented reference library (NIST main library with spectra of query-set molecules replaced by NEIMS predictions)
- NIST 2017 replicates library query spectra
- Molecular mass of each query molecule

## Outputs

- Per-query rank of the correct candidate spectrum under the 5 Da mass filter
- Per-query filtered library sizes (median ~6696 spectra)
- Recall@10 statistics and per-query ranks (e.g., saved to mass_filter_results.csv)

## How to apply

For each query spectrum from the NIST 2017 replicates library, extract the query molecule's molecular mass and filter the augmented reference library (NIST main library with NEIMS-predicted spectra substituted for query-set molecules) to retain only candidate spectra whose molecular mass differs by at most 5 Da from the query. Compute weighted cosine similarity (eq 1) between the query spectrum and every mass-compatible candidate, record the rank of the correct candidate spectrum, and evaluate recall@10 (fraction of queries whose correct spectrum ranks at or below 10). The rationale is that for experimental setups where molecular mass is known with some accuracy, pruning mass-incompatible candidates both reduces computation and improves matching accuracy; the 5 Da tolerance reduces the library to a median of 6696 spectra per query molecule.

## Related tools

- **NEIMS** (Provides the library-matching framework whose mass-filtering option prunes the augmented library by molecular-mass tolerance before similarity search) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Used to generate the molecular representations from which molecular masses of library and query molecules are obtained for filtering)

## Evaluation signals

- Filtered library size per query should be ~6696 spectra at the median with a 5 Da tolerance
- Recall@10 with the 5 Da mass filter should exceed recall@10 obtained on the unfiltered augmented library
- Every retained candidate's molecular mass must lie within 5 Da of the query molecule's mass (no mass-incompatible spectra survive filtering)
- The correct candidate spectrum should not be removed by the filter for queries reported as correctly matched

## Limitations

- The filter is only applicable when the molecular mass of the sample is known with some accuracy; otherwise mass-incompatible pruning can exclude the true hit
- NEIMS does not model isotope intensity peaks in ion fragments, so a small shift is needed for isotopes and exact-mass matching would be unreliable

## Evidence

- [readme] Library-matching task setup from README: [evidence span withheld — anchor status: external_doc]
