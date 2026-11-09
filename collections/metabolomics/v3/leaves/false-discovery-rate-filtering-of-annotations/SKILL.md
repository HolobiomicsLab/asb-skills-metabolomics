---
name: false-discovery-rate-filtering-of-annotations
description: Use when reporting counts of spectra or unique small molecules annotated
  by a mass spectral search method (e.g., molDiscovery vs Dereplicator+) and identifications
  must be filtered at a stated FDR threshold such as 0%.
when_to_use_negative:
- Exploratory annotation review where sensitivity matters more than specificity —
  0% FDR discards lower-scoring true matches, so an unfiltered or relaxed threshold
  may be preferable for hypothesis generation.
- When comparing methods at different effective thresholds — counts of spectra/compounds
  are only comparable if both methods are filtered at the same FDR level.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: molDiscovery
  role: Search tool whose scored molecule-spectrum identifications are filtered at
    the chosen FDR threshold; uses a learned probabilistic fragmentation model for
    scoring
  repo: github.com/mohimanilab/molDiscovery
- name: Dereplicator+
  role: Baseline search method whose identifications are filtered at the same FDR
    level to enable fair comparison of annotated spectra and unique compounds
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:93bc16fae2f186e3cbc16896023af707e8ff0e031367f516780a398d5d796bbf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dereplicator+
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# false-discovery-rate-filtering-of-annotations

## Summary

Filter mass spectral database search annotations (e.g., from molDiscovery or Dereplicator+) at a target false discovery rate — such as 0% FDR — before reporting matched spectra and unique small molecule counts, so reported identifications control the expected rate of spurious molecule-spectrum assignments.

## When to use

Use when reporting counts of spectra or unique small molecules annotated by a mass spectral search method (e.g., molDiscovery vs Dereplicator+) and identifications must be filtered at a stated FDR threshold such as 0%.

## When NOT to use

- Exploratory annotation review where sensitivity matters more than specificity — 0% FDR discards lower-scoring true matches, so an unfiltered or relaxed threshold may be preferable for hypothesis generation.
- When comparing methods at different effective thresholds — counts of spectra/compounds are only comparable if both methods are filtered at the same FDR level.

## Inputs

- Scored molecule-spectrum identification results from molDiscovery or Dereplicator+ (each with an FDR estimate)
- Tandem mass spectra from GNPS/MassIVE datasets (e.g., MSV000079450, MSV000084092, MSV000086427; ~8 million spectra)
- Molecular structure database used for the search (e.g., AllDB, 719,958 structures)

## Outputs

- FDR-filtered annotation list (identifications retained at the chosen threshold, e.g., 0% FDR)
- Tabulated counts of matched spectra and unique small molecules per method (e.g., moldiscovery_gnps_search_results.csv)

## How to apply

Run the database search tool (molDiscovery with its learned probabilistic fragmentation model, or a baseline such as Dereplicator+) over the tandem mass spectra against the chosen structure database (e.g., AllDB with 719,958 small-molecule structures), obtaining scored molecule-spectrum pairs with associated FDR estimates. Apply the FDR filter as a post-processing step on the score-sorted identifications — in the GNPS benchmark all identifications were retained only at 0% FDR, meaning no decoy-based false matches were tolerated. Then tabulate two metrics on the filtered set: number of matched tandem mass spectra and number of unique compounds. Because spectral search scores are probabilistic model outputs, the FDR threshold is the key decision point that directly determines the reported counts (e.g., molDiscovery: 56,971 spectra / 3,185 unique compounds at 0% FDR on ~8 million GNPS spectra vs AllDB).

## Related tools

- **molDiscovery** (Search tool whose scored molecule-spectrum identifications are filtered at the chosen FDR threshold; uses a learned probabilistic fragmentation model for scoring) — github.com/mohimanilab/molDiscovery
- **Dereplicator+** (Baseline search method whose identifications are filtered at the same FDR level to enable fair comparison of annotated spectra and unique compounds)

## Examples

```
python scripts/run_moldiscovery_search.py --spectra msv000079450.mgf --database AllDB --fdr 0.0 --output moldiscovery_gnps_search_results.csv
```

## Evaluation signals

- Reproduced filtered counts match published figures: 56,971 spectra annotated by molDiscovery vs 7,684 for Dereplicator+ (8x), and 3,185 unique compounds vs 542 (6x) at 0% FDR
- Identified unique molecules by molDiscovery at 0% FDR equal 3,185, an approximately six-fold increase over the baseline's 542
- Both compared methods are filtered at the identical FDR level (0%) before counting
- Output table contains both required metrics: matched-spectra count and unique-compound count per method
- The FDR estimates accompany each identification before filtering (decoy-based estimate available for all molecule-spectrum pairs)

## Limitations

- molDiscovery lacks p-value computation for general molecules; the MCMC p-value approach exists only for peptidic molecules and extension is challenging
- The 0% FDR operating point is strict; accuracy is lower in the low-mass range (<600 Da) partly because fewer fragments are generated for small molecules, which may affect FDR-calibrated scores there
- Fragmentation accuracy depends on experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled by the default model trained on combined CID/HCD positive-mode spectra, which may distort FDR calibration outside those conditions

## Evidence

- [intro] Searching 8 million GNPS tandem mass spectra, molDiscovery identified 3185 unique small molecules at 0% FDR, a six times increase compared to existing methods: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [abstract] molDiscovery correctly identifies six times more unique small molecules than previous methods in a search of over 8 million spectra: "molDiscovery correctly identify six times more unique small molecules than previous methods."
- [other] Filter both methods' identifications at 0% false discovery rate (FDR): [evidence span withheld — anchor status: not_found]
- [results] Reproduced benchmark counts for validation: [evidence span withheld — anchor status: not_found]
