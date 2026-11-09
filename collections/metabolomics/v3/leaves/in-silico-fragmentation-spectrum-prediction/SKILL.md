---
name: in-silico-fragmentation-spectrum-prediction
description: 'Use when benchmarking tandem MS metabolite annotation tools: test spectra
  with known structures must be ranked against a compiled compound database using
  predicted fragmentation or fingerprints, and top-k retrieval rates compared to ChemDistiller.'
when_to_use_negative:
- Input spectra are not in ChemDistiller's internal hierarchical format and no format
  conversion is available yet (README notes conversions were planned, not shipped)
- No compound database with pre-calculated fingerprints and fragmentation patterns
  is available, so candidates cannot be ranked by fingerprint/fragment matching
- The goal is de novo structure elucidation of unknowns rather than retrieval-rate
  benchmarking against a known-compound test split
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MetFrag
  role: Baseline tool run from the command line on each test spectrum, ranking database
    candidates by in silico fragmentation against measured MS/MS peaks under the benchmark's
    precursor mass tolerance
- name: CFM-ID
  role: Baseline tool run in single-energy mode to generate predicted spectra and
    score database candidates for each test spectrum
- name: SIRIUS
  role: Baseline tool run on each test spectrum to compute fragmentation trees for
    CSI:FingerID queries
- name: CSI:FingerID
  role: Baseline queried with SIRIUS fragmentation trees to rank candidate compounds
    by fingerprint match
- name: ChemDistiller
  role: Reference annotation engine whose scorers' top-k retrieval rates the baselines
    are compared against; uses SVM-based FingerScorer over compounds with pre-calculated
    fingerprints
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:8f9e55aa77b001a0cd0f34e96bfa72cc26a661139f0a70acaa5eb9c1628932db
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
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# in-silico-fragmentation-spectrum-prediction

## Summary

Ranks candidate compounds for tandem MS spectra by generating predicted in-silico fragment spectra or structural fingerprints and matching them against measured MS/MS peaks, enabling side-by-side benchmarking of MetFrag, CFM-ID and CSI:FingerID against ChemDistiller's scorers under identical database and tolerance conditions.

## When to use

Use when benchmarking tandem MS metabolite annotation tools: test spectra with known structures must be ranked against a compiled compound database using predicted fragmentation or fingerprints, and top-k retrieval rates compared to ChemDistiller.

## When NOT to use

- Input spectra are not in ChemDistiller's internal hierarchical format and no format conversion is available yet (README notes conversions were planned, not shipped)
- No compound database with pre-calculated fingerprints and fragmentation patterns is available, so candidates cannot be ranked by fingerprint/fragment matching
- The goal is de novo structure elucidation of unknowns rather than retrieval-rate benchmarking against a known-compound test split

## Inputs

- Test and train tandem MS spectrum splits with known compound structures (e.g. MassBank test spectra, 863 spectra)
- Compiled chemical compound database in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns
- Precursor MS1 peak m/z values with ppm tolerance setting (delta_mz)
- Baseline tool configurations: MetFrag command-line settings, CFM-ID single-energy mode, SIRIUS fragmentation tree parameters for CSI:FingerID queries

## Outputs

- Ranked candidate compound lists per test spectrum for each tool (MetFrag, CFM-ID, SIRIUS/CSI:FingerID, ChemDistiller)
- Top-k retrieval rates on the test split per tool and scorer
- Side-by-side comparison table validating the claim that ChemDistiller retrieval rates are comparable to or exceeding state-of-the-art solutions

## How to apply

Load the same train/test tandem MS spectra splits and the compiled compound database (with pre-calculated fingerprints and fragmentation patterns) used for the ChemDistiller benchmark so all tools are evaluated under identical conditions, including the same MS1 precursor m/z tolerance (--delta_mz, in ppm). Run MetFrag from the command line on each test spectrum, ranking database candidates by in silico fragmentation against the measured MS/MS peaks; run CFM-ID in single-energy mode to generate predicted spectra and score candidates; run SIRIUS to compute fragmentation trees and query CSI:FingerID to rank candidates by fingerprint match. Parse each tool's ranked result lists, score correct molecular-formula/structure hits, and compute top-k (e.g. top-1) retrieval rates on the test split. Decision point: under the shared conditions MetFrag can slightly outperform ChemDistiller's best scorer at top-1 on the small database (51.7%/45% versus 50%/44%), so judge success by reproduction of published Table 1 values within a small tolerance rather than by universal superiority of any tool.

## Related tools

- **MetFrag** (Baseline tool run from the command line on each test spectrum, ranking database candidates by in silico fragmentation against measured MS/MS peaks under the benchmark's precursor mass tolerance)
- **CFM-ID** (Baseline tool run in single-energy mode to generate predicted spectra and score database candidates for each test spectrum)
- **SIRIUS** (Baseline tool run on each test spectrum to compute fragmentation trees for CSI:FingerID queries)
- **CSI:FingerID** (Baseline queried with SIRIUS fragmentation trees to rank candidate compounds by fingerprint match)
- **ChemDistiller** (Reference annotation engine whose scorers' top-k retrieval rates the baselines are compared against; uses SVM-based FingerScorer over compounds with pre-calculated fingerprints) — https://bitbucket.org/iAnalytica/chemdistillerpython

## Examples

```
python annotate.py --test
```

## Evaluation signals

- Reproduced baseline top-1 retrieval rates match the published Table 1 values within a small tolerance (e.g. MetFrag 51.7%/45% versus ChemDistiller 50%/44% on the small database)
- ChemDistiller test run on the 863 MassBank spectra prints retrieval statistics since these compounds have known chemical structures
- All tools are evaluated on the identical train/test spectrum splits and the identical compiled compound database and precursor mass tolerance (--delta_mz in ppm)
- Each tool's ranked result lists yield correct molecular-formula/structure hits, so top-k rates are computed on the same hit definition across tools
- Comparison reproduces the published conclusion that ChemDistiller retrieval rates are comparable to or exceeding the state-of-the-art solutions

## Limitations

- The shipped ChemDistiller version includes only a small set of compound databases and simple (Linear_weighted) SVM models; Radial_weighted SVMs are larger, slower, but more accurate and must be downloaded separately
- Input spectra must be converted to ChemDistiller's internal hierarchical format; conversions from other formats were not yet implemented at publication time
- Derivative compound databases have unclear commercial-use restrictions inherited from original open databases; consult readme.txt and source database disclaimers before commercial use
- On the small database, MetFrag can slightly outperform ChemDistiller's best scorer at the top-1 position, so benchmark conclusions are condition-dependent

## Evidence

- [abstract] Database with pre-calculated fingerprints: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [abstract] State-of-the-art comparison claim: "show retrievals rates comparable to or exceeding the ones obtainable by the current state-of-the-art solutions in the field while offering higher throughput, scalability and processing speed"
- [readme] Precursor mass tolerance parameter: [evidence span withheld — anchor status: not_found]
- [readme] Test-mode retrieval statistics: [evidence span withheld — anchor status: external_doc]
