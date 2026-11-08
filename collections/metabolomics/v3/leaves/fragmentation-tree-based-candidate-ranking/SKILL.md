---
name: fragmentation-tree-based-candidate-ranking
description: 'Use when benchmarking metabolite annotation engines on tandem MS/MS
  spectra: you need the SIRIUS/CSI:FingerID fragmentation-tree baseline ranked candidate
  lists under the same test splits, compound database and precursor mass tolerance
  as the ChemDistiller evaluation.'
when_to_use_negative:
- You only need ChemDistiller's own FingerScorer/annotator output rather than an independent
  state-of-the-art baseline — running ChemDistiller directly (python annotate.py)
  covers that.
- Your spectra are not tandem MS/MS spectra tied to a compound candidate database,
  so fragmentation-tree/fingerprint ranking has nothing to rank against.
tools:
- name: SIRIUS
  role: Computes fragmentation trees for each test spectrum as the basis for candidate
    ranking
- name: CSI:FingerID
  role: Queries the predicted fingerprint and ranks database candidate compounds by
    fingerprint match
- name: ChemDistiller
  role: Defines the shared benchmark conditions (spectrum splits, compound database,
    mass tolerance) and the scorer rates the baseline is compared against
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: MetFrag
  role: Companion in silico fragmentation baseline whose retrieval rates contextualize
    the fragmentation-tree ranking results
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:815d526fc0278391024ff8143d571f5e48358c406ba5187298fb8117b9ca90db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - CSI:FingerID
  - ChemDistiller
  - MetFrag
  - SIRIUS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fragmentation-tree-based-candidate-ranking

## Summary

Benchmark-baseline method that computes fragmentation trees with SIRIUS for each tandem MS test spectrum and queries CSI:FingerID to rank database candidate compounds by fingerprint match, providing the fragmentation-tree-based retrieval-rate column in the ChemDistiller comparison benchmark.

## When to use

Use when benchmarking metabolite annotation engines on tandem MS/MS spectra: you need the SIRIUS/CSI:FingerID fragmentation-tree baseline ranked candidate lists under the same test splits, compound database and precursor mass tolerance as the ChemDistiller evaluation.

## When NOT to use

- You only need ChemDistiller's own FingerScorer/annotator output rather than an independent state-of-the-art baseline — running ChemDistiller directly (python annotate.py) covers that.
- Your spectra are not tandem MS/MS spectra tied to a compound candidate database, so fragmentation-tree/fingerprint ranking has nothing to rank against.

## Inputs

- Train/test tandem MS (MS/MS) spectrum splits used in the ChemDistiller benchmark
- Compiled compound database with pre-calculated fingerprints and fragmentation patterns
- Precursor MS1 peak m/z values and benchmark precursor mass tolerance (ppm)

## Outputs

- Per-spectrum fragmentation trees computed by SIRIUS
- Ranked candidate compound lists from CSI:FingerID fingerprint matching
- Top-k retrieval rates (top-1 and higher k) on the test split for comparison against Table 1

## How to apply

Load the identical train/test tandem MS spectrum splits and the compiled compound database (with pre-calculated fingerprints and fragmentation patterns) used for the ChemDistiller benchmark so all tools are evaluated under identical conditions. Run SIRIUS on each test spectrum to compute fragmentation trees, then query CSI:FingerID to rank candidate compounds by fingerprint match, applying the benchmark's precursor mass tolerance (ChemDistiller's analogous MS1 tolerance is specified in ppm via --delta_mz, e.g. 20 ppm). Parse each ranked result list, score correct molecular-formula/structure hits, and compute top-k (notably top-1) retrieval rates on the test split. Compare side-by-side with ChemDistiller's scorer rates and the MetFrag baseline: under the shared conditions MetFrag can slightly outperform ChemDistiller's best scorer at top-1 on the small database (51.7%/45% versus 50%/44%), and the ensemble of rates should support the claim that ChemDistiller is comparable to or exceeding state-of-the-art solutions.

## Related tools

- **SIRIUS** (Computes fragmentation trees for each test spectrum as the basis for candidate ranking)
- **CSI:FingerID** (Queries the predicted fingerprint and ranks database candidate compounds by fingerprint match)
- **ChemDistiller** (Defines the shared benchmark conditions (spectrum splits, compound database, mass tolerance) and the scorer rates the baseline is compared against) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **MetFrag** (Companion in silico fragmentation baseline whose retrieval rates contextualize the fragmentation-tree ranking results)

## Evaluation signals

- Reproduced CSI:FingerID retrieval rates match the published Table 1 values within a small tolerance on the shared test split.
- Side-by-side comparison shows MetFrag slightly outperforming ChemDistiller's best scorer at top-1 on the small database (51.7%/45% versus 50%/44%).
- All baseline rankings were computed on the same splits, database and precursor mass tolerance as ChemDistiller (identical evaluation conditions).
- Correct molecular-formula/structure hits are identifiable and scored from each parsed ranked result list before computing top-k rates.

## Limitations

- The article's discussion section was unavailable in the provided document ('No changelog found.'), so interpretation caveats beyond the results/abstract could not be traced.
- Benchmark databases and supplementary models are redistributed 'AS IS'; derivative compound databases may inherit commercial-use restrictions from their original sources.
- For the ChemDistiller side of the shared conditions, spectra must be converted to the engine's internal hierarchical format, which constrained cross-tool benchmarking at v0.1.

## Evidence

- [other] Run SIRIUS on each test spectrum to compute fragmentation trees and query CSI:FingerID to rank candidate compounds by fingerprint match.: [evidence span withheld — anchor status: not_found]
- [abstract] Retrieval rates comparable to or exceeding state-of-the-art: "show retrievals rates comparable to or exceeding the ones obtainable by the current state-of-the-art solutions in the field while offering higher throughput, scalability and processing speed"
- [abstract] Compiled database with pre-calculated fingerprints: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints'"
- [readme] MS1 m/z tolerance parameter in ppm: [evidence span withheld — anchor status: external_doc]
