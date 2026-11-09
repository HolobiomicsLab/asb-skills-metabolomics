---
name: cross-tool-annotation-comparison
description: 'Use when benchmarking tandem MS/MS metabolite annotation: identical
  spectra splits and compound databases must be run through multiple tools and top-k
  retrieval rates compared side by side.'
when_to_use_negative:
- The goal is to improve annotation accuracy rather than measure it — use ChemDistiller's
  own annotate.py pipeline instead
- Spectra lack known ground-truth structures, so correct-hit retrieval rates cannot
  be scored
- Only one tool is being evaluated and no cross-tool baseline comparison is required
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: Reference annotation engine whose scorers define the comparison baseline;
    provides spectra splits, HDF5 compound databases and retrieval statistics
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: MetFrag
  role: Baseline run from the command line; ranks database candidates by in silico
    fragmentation against measured MS/MS peaks
- name: CFM-ID
  role: Baseline run in single-energy mode; generates predicted spectra and scores
    database candidates
- name: SIRIUS
  role: Baseline; computes fragmentation trees for each test spectrum before querying
    CSI:FingerID
- name: CSI:FingerID
  role: Baseline; ranks candidate compounds by fingerprint match given SIRIUS fragmentation
    trees
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:5b73a653951b8b9f3ac20e5dcfac1311803a3a37f98b04850c739e2125fe258c
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-tool-annotation-comparison

## Summary

Reproduces retrieval-performance comparison rows for tandem-MS metabolite annotation tools (MetFrag, CFM-ID, SIRIUS/CSI:FingerID) against ChemDistiller's scorers, so that claims of comparable or exceeding state-of-the-art retrieval rates can be independently verified under identical test conditions.

## When to use

Use when benchmarking tandem MS/MS metabolite annotation: identical spectra splits and compound databases must be run through multiple tools and top-k retrieval rates compared side by side.

## When NOT to use

- The goal is to improve annotation accuracy rather than measure it — use ChemDistiller's own annotate.py pipeline instead
- Spectra lack known ground-truth structures, so correct-hit retrieval rates cannot be scored
- Only one tool is being evaluated and no cross-tool baseline comparison is required

## Inputs

- Tandem MS test spectra (train/test splits, e.g. ChemDistiller's MassBank test spectra set of 863 spectra)
- Compiled compound database in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns
- ChemDistiller ranked annotation result lists for the same test split
- Published baseline Table 1 retrieval-rate values for MetFrag, CFM-ID and CSI:FingerID

## Outputs

- Per-tool ranked candidate lists (molecular formula/structure hits) for each test spectrum
- Top-k retrieval rates (e.g. top-1 and top-3) for MetFrag, CFM-ID, SIRIUS/CSI:FingerID
- Side-by-side comparison table of baseline tools versus ChemDistiller scorers validating the 'comparable to or exceeding' claim

## How to apply

Load the same train/test tandem MS spectra splits and the compiled compound database (with pre-calculated fingerprints and fragmentation patterns) used for the ChemDistiller benchmark so all tools are evaluated under identical conditions. Run MetFrag from the command line on each test spectrum, ranking database candidates by in silico fragmentation against measured MS/MS peaks under the benchmark's precursor mass tolerance. Run CFM-ID in single-energy mode to generate predicted spectra and score database candidates, and run SIRIUS to compute fragmentation trees and query CSI:FingerID for fingerprint-based ranking. Parse each tool's ranked result lists, score correct molecular-formula/structure hits, and compute top-k retrieval rates on the test split; then place these rates side-by-side with ChemDistiller's per-scorer results. Validation requires the reproduced baseline rates to match the published Table 1 values within a small tolerance (e.g. MetFrag top-1 on the small database: 51.7%/45% vs ChemDistiller's 50%/44%).

## Related tools

- **ChemDistiller** (Reference annotation engine whose scorers define the comparison baseline; provides spectra splits, HDF5 compound databases and retrieval statistics) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **MetFrag** (Baseline run from the command line; ranks database candidates by in silico fragmentation against measured MS/MS peaks)
- **CFM-ID** (Baseline run in single-energy mode; generates predicted spectra and scores database candidates)
- **SIRIUS** (Baseline; computes fragmentation trees for each test spectrum before querying CSI:FingerID)
- **CSI:FingerID** (Baseline; ranks candidate compounds by fingerprint match given SIRIUS fragmentation trees)

## Examples

```
python annotate.py Input_Spectra_folder output_folder --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Reproduced baseline top-k retrieval rates match the published Table 1 values within a small tolerance
- Known-known sanity check: ChemDistiller test mode on the 863 MassBank spectra prints retrieval statistics for compounds with known structures
- Cross-tool comparison uses the identical train/test split, compound database and precursor mass tolerance (delta_mz in ppm) for every tool
- Side-by-side rates support the claim that ChemDistiller retrieval rates are comparable to or exceeding the state-of-the-art solutions
- Per-tool output parses cleanly into ranked result lists with correct molecular-formula/structure hit scoring

## Limitations

- Comparison requires identical data conditions; differences in precursor mass tolerance or candidate databases invalidate the comparison
- The finding is narrow: MetFrag can slightly outperform ChemDistiller's best scorer at the top-1 position on the small database (51.7%/45% versus 50%/44%)
- Large compound databases and stronger (Radial_weighted) SVMs must be downloaded separately due to size limitations; default installs ship only small databases and simple SVMs
- ChemDistiller's spectra must be converted into its internal hierarchical format; format conversions were not yet implemented at publication

## Evidence

- [abstract] retrieval-rate claim: "show retrievals rates comparable to or exceeding the ones obtainable by the current state-of-the-art solutions in the field while offering higher throughput, scalability and processing speed"
- [abstract] database description: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints'"
- [readme] test-mode retrieval statistics: [evidence span withheld — anchor status: external_doc]
- [readme] known-compound scoring: [evidence span withheld — anchor status: external_doc]
