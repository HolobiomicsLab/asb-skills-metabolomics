---
name: spectral-match-false-discovery-estimation
description: Use when quantifying how many exact spectral library match results (peak-level
  bipartite matching with normalized dot-product scores) are expected to be false
  positives, i.e. attaching FDR estimates to GNPS dereplication output.
when_to_use_negative:
- The goal is molecular networking or spectrum-to-spectrum comparison (MSCluster consensus
  spectra, top-K mutual-neighbor edges), not library-search confidence estimation
- Input spectra are not being compared against a reference spectral library (e.g.
  no library spectra or precursor m/z available), so no match set exists to estimate
  FDR over
- Modification-tolerant (variable dereplication) search is intended, where peak edges
  differing by the precursor mass delta are added — this is a distinct search mode
  from the exact match FDR workflow
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Executes the exact spectral library search whose match-level results are the
    input to FDR estimation; provides crowdsourced reference libraries
- name: MassIVE
  role: Repository supplying the public MS/MS datasets against which library match
    rates and FDRs are evaluated
- name: CCMS github page
  role: Source code for the GNPS exact spectral library search and its FDR estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:98e24bf8809931c44129e5ff5a463f60f8cde18eeb6d685d23cb83b17ef7aeac
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CCMS github page
  - GNPS
  - MassIVE
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-match-false-discovery-estimation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Estimates the false discovery rate of exact spectral library searches in GNPS, where library-to-query MS/MS peak matches are found by maximum bipartite matching and scored with a normalized dot product. It attaches a statistical confidence layer to dereplication calls so reported compound identifications can be trusted.

## When to use

Use when quantifying how many exact spectral library match results (peak-level bipartite matching with normalized dot-product scores) are expected to be false positives, i.e. attaching FDR estimates to GNPS dereplication output.

## When NOT to use

- The goal is molecular networking or spectrum-to-spectrum comparison (MSCluster consensus spectra, top-K mutual-neighbor edges), not library-search confidence estimation
- Input spectra are not being compared against a reference spectral library (e.g. no library spectra or precursor m/z available), so no match set exists to estimate FDR over
- Modification-tolerant (variable dereplication) search is intended, where peak edges differing by the precursor mass delta are added — this is a distinct search mode from the exact match FDR workflow

## Inputs

- Query MS/MS spectra with precursor charge state, precursor m/z, and peak lists
- GNPS community reference MS/MS spectral library entries (with provenance/downloadable input files)
- User-defined search parameters: fragment mass tolerance (Thompson), precursor m/z tolerance, minimum matched peaks, minimum spectral match score

## Outputs

- Estimated false discovery rates for the exact spectral library search (per Supplementary Note 3)
- Documented search parameter set (analogous to Supplementary Table 8)
- Table of best matches per query with dot-product score, matched peak count, and FDR-qualified annotation status

## How to apply

Run the exact spectral library search first: pre-filter candidates by same precursor charge state, precursor m/z within a user-defined Thompson tolerance, a minimum number of matched peaks, and a minimum spectral match score; then solve maximum bipartite matching of library and query peaks under the fragment mass tolerance and score each match with a normalized dot product. Over the resulting match set, estimate the false discovery rate of the exact spectral library search as documented in the paper's Supplementary Note 3, using the search parameters recorded in Supplementary Table 8 so the FDR is reproducible. Report the FDR alongside the best matches (port_id, precursor m/z, matched library compound, dot-product score, matched peak count) rather than raw score counts alone. The rationale is that without an FDR estimate, dereplications against the crowdsourced GNPS-Community reference libraries cannot be statistically vetted — the paper explicitly reports 'estimated false discovery rates of the exact spectral library search' as part of validating the workflow.

## Related tools

- **GNPS** (Executes the exact spectral library search whose match-level results are the input to FDR estimation; provides crowdsourced reference libraries)
- **MassIVE** (Repository supplying the public MS/MS datasets against which library match rates and FDRs are evaluated)
- **CCMS github page** (Source code for the GNPS exact spectral library search and its FDR estimation)

## Examples

```
from scipy.optimize import linear_sum_assignment; rows, cols = linear_sum_assignment(-score_matrix)
```

## Evaluation signals

- Only query spectra whose best match exceeds the user-defined minimum spectral match score, same charge state, m/z within Thompson tolerance, and minimum matched peak count are counted as matches before FDR computation
- An explicit FDR estimate accompanies the match table (mirroring Supplementary Note 3 reporting), not just raw scores
- The full search parameter set is recorded (mirroring Supplementary Table 8) so the FDR can be reproduced
- Matched fraction is sanity-checked against the expected low coverage — the paper reports only 1.01% of all spectra in public GNPS datasets matched to third-party reference libraries, so implausibly high match rates warrant investigation
- For each query, exactly one highest-scoring library match is reported, with dot-product score and matched peak count populated

## Limitations

- FDR estimation details are deferred to Supplementary Note 3 and search parameters to Supplementary Table 8, which are not included in the main text; exact decoy/threshold methodology must be recovered from the supplement or source code
- Insufficient chemical space coverage limits interpretation: only 1.01% of spectra in public GNPS datasets matched to third-party reference libraries
- Results are sensitive to user-defined thresholds (Thompson m/z tolerance, minimum matched peaks, minimum match score) that are set per job, so FDRs are not comparable across differently parameterized searches

## Evidence
