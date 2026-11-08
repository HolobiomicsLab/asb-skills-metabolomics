---
name: identification-count-benchmark-reproduction
description: Reference for reproducing large-scale in silico small molecule identification
  benchmarks where reported counts (e.g., 56,971 spectra / 3,185 unique molecules
  at 0% FDR) must be validated against a baseline method.
when_to_use_negative:
- Input spectra are not tandem MS/MS spectra from GNPS-style metabolomics experiments,
  or the target database differs substantially from AllDB, so the reported 3,185 /
  542 count targets are not comparable.
- The goal is de novo structure prediction or spectral library search against reference
  spectra, which the article treats as distinct identification strategies with different
  limitations.
- A non-zero FDR threshold is required; the benchmark's headline counts are defined
  specifically at 0% FDR.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: molDiscovery
  role: 'Primary search method: scores molecule-spectrum pairs against AllDB using
    a learned probabilistic fragmentation model over the ~8 million GNPS spectra'
  repo: github.com/mohimanilab/molDiscovery
- name: Dereplicator+
  role: Baseline in silico database search method run on the same spectra against
    AllDB for count comparison at 0% FDR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:3dadba2f61cde3a6bbe1b7b8967408fd39112bb64c29164e192cc515769b4851
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dereplicator+
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# identification-count-benchmark-reproduction

## Summary

Reproduces the MolDiscovery Nature Communications benchmark that searches ~8 million GNPS tandem mass spectra from 46 MassIVE datasets against the AllDB database (719,958 structures) and tabulates spectrum and unique-compound identification counts at 0% FDR against the Dereplicator+ baseline.

## When to use

Reference for reproducing large-scale in silico small molecule identification benchmarks where reported counts (e.g., 56,971 spectra / 3,185 unique molecules at 0% FDR) must be validated against a baseline method.

## When NOT to use

- Input spectra are not tandem MS/MS spectra from GNPS-style metabolomics experiments, or the target database differs substantially from AllDB, so the reported 3,185 / 542 count targets are not comparable.
- The goal is de novo structure prediction or spectral library search against reference spectra, which the article treats as distinct identification strategies with different limitations.
- A non-zero FDR threshold is required; the benchmark's headline counts are defined specifically at 0% FDR.

## Inputs

- 46 GNPS MassIVE dataset files (e.g., MSV000079450, MSV000084092, MSV000086427) containing ~8 million tandem mass spectra
- AllDB molecular structure database of 719,958 small-molecule structures

## Outputs

- moldiscovery_gnps_search_results.csv tabulating matched spectra and unique compound counts per method at 0% FDR
- molDiscovery identification counts: 56,971 spectra / 3,185 unique compounds
- Dereplicator+ baseline identification counts: 542 unique compounds

## How to apply

Download the 46 GNPS MassIVE dataset files (e.g., MSV000079450, MSV000084092, MSV000086427) totaling ~8 million tandem mass spectra from the Global Natural Product Social molecular networking infrastructure, and prepare the AllDB database of 719,958 small-molecule structures. Run molDiscovery over all spectra against AllDB so each molecule-spectrum pair is scored with its learned probabilistic fragmentation model, then run Dereplicator+ on the identical spectra/database as baseline. Filter both methods' identifications at 0% false discovery rate (FDR) — this cutoff is the decisive threshold of the benchmark. Tabulate matched spectra and unique compounds per method (molDiscovery: 56,971 spectra / 3,185 unique compounds; Dereplicator+: 542 unique compounds, i.e., ~8x and ~6x improvements) and write results to moldiscovery_gnps_search_results.csv. Validate by confirming the reproduced unique-molecule count matches 3,185 at 0% FDR, approximately six times the baseline's 542.

## Related tools

- **molDiscovery** (Primary search method: scores molecule-spectrum pairs against AllDB using a learned probabilistic fragmentation model over the ~8 million GNPS spectra) — github.com/mohimanilab/molDiscovery
- **Dereplicator+** (Baseline in silico database search method run on the same spectra against AllDB for count comparison at 0% FDR)

## Examples

```
molDiscovery --spectra gnps_8M.mgf --database AllDB.tsv --fdr 0.0 && python tabulate_counts.py --out moldiscovery_gnps_search_results.csv
```

## Evaluation signals

- molDiscovery at 0% FDR identifies 3,185 unique small molecules on the 8-million-spectrum GNPS search — the reproduced count must match this figure.
- molDiscovery annotates 56,971 spectra at 0% FDR versus 7,684 for Dereplicator+ (approximately 8-fold).
- Dereplicator+ baseline yields 542 unique compounds; the reproduced ratio (~6x) must hold.
- FDR filtering is applied identically to both methods' identification lists before counting.
- Output CSV moldiscovery_gnps_search_results.csv contains per-method spectra and unique compound counts for the same spectra/database pair.

## Limitations

- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules; benzene rings and large resonant structures are naively treated as alternating single/double bonds.
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra.
- Lower identification accuracy in low mass ranges (<600 Da), partly because fewer fragments are generated for small molecules.
- No p-value computation is available for general molecules; the MCMC p-value approach exists only for peptidic molecules.

## Evidence

- [intro] 8 million spectra GNPS search counts: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [abstract] probabilistic model rationale: "learning a probabilistic model to match small molecules with their mass spectra"
- [abstract] scale of search: "A search of over 8 million spectra from the Global Natural Product Social molecular networking infrastructure"
- [abstract] six-fold claim: "molDiscovery correctly identify six times more unique small molecules than previous methods."
- [readme] tool access: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
