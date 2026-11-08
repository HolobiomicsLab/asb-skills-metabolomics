---
name: cross-dataset-accuracy-comparison
description: Use when replicating or validating a published benchmark (e.g., FSA rank#1/top-5
  accuracy and runtime) across several MS/MS test datasets against reported numeric
  results.
when_to_use_negative:
- The goal is definitive metabolite identification or structural annotation — FSA's
  rank#1 accuracy was explicitly reported as not yet comparable to Buddy or SIRIUS
  since it does not use structural information
- The spectra originate from compounds beyond C, H, O, N, S, P plus halogens, or form
  diverse adduct types — FSA was not validated in those regimes
- There is no reported reference metric to compare against and only a single dataset,
  so cross-dataset accuracy replication is not meaningful
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FSA
  role: Formula ranking engine whose rank#1 accuracy, top-5 accuracy, and per-spectrum
    runtime are recomputed and compared across datasets
  repo: kslynn128171/FSA
- name: PubChem
  role: Source of the chemical formulas and monoisotopic molecular weights that build
    the expanded reference formula list used in the benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:84bf0ae3339dda6ad7c4581879098481134b919f7ba375db919ea2e6555bcd87
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - PubChem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cross-dataset-accuracy-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recomputes a paper's benchmark metrics (rank#1 accuracy, top-5 accuracy, per-spectrum runtime) across multiple test datasets and compares them against reported values within a stated tolerance, saved as a structured comparison table.

## When to use

Use when replicating or validating a published benchmark (e.g., FSA rank#1/top-5 accuracy and runtime) across several MS/MS test datasets against reported numeric results.

## When NOT to use

- The goal is definitive metabolite identification or structural annotation — FSA's rank#1 accuracy was explicitly reported as not yet comparable to Buddy or SIRIUS since it does not use structural information
- The spectra originate from compounds beyond C, H, O, N, S, P plus halogens, or form diverse adduct types — FSA was not validated in those regimes
- There is no reported reference metric to compare against and only a single dataset, so cross-dataset accuracy replication is not meaningful

## Inputs

- Six non-lipid MS/MS test datasets (spectra to be ranked)
- Halogen-extended PubChem reference formula list with monoisotopic masses (target: 2,613,203 formulas incl. F, Cl, Br, I)
- Paper-reported reference metrics: 77.84% rank#1, 92.60% top-5, 0.16 s per spectrum

## Outputs

- Per-spectrum ranked candidate formula lists from FSA
- Per-spectrum wall-clock runtime measurements
- Computed rank#1 accuracy, top-5 accuracy, and average runtime per spectrum
- Structured comparison table of recomputed vs. reported metrics

## How to apply

Run the tool under test (FSA) over the six non-lipid MS/MS test datasets using the configured element/formula reference list — e.g., the halogen-extended PubChem-derived list of 2,613,203 formulas covering F, Cl, Br, I in addition to C, H, O, N, S, P. For each spectrum, record the ranked candidate molecular formula list and the wall-clock processing time. Aggregate per-dataset and pooled rank#1 accuracy and top-5 accuracy, plus average runtime per spectrum. Compare recomputed values against the paper's reported halogen-inclusion results (77.84% rank#1, 92.60% top-5, 0.16 s/spectrum) within a stated tolerance, and verify structural invariants such as the reference formula count matching 2,613,203. Persist the accuracy and runtime comparison as a structured table for audit.

## Related tools

- **FSA** (Formula ranking engine whose rank#1 accuracy, top-5 accuracy, and per-spectrum runtime are recomputed and compared across datasets) — kslynn128171/FSA
- **PubChem** (Source of the chemical formulas and monoisotopic molecular weights that build the expanded reference formula list used in the benchmark)

## Examples

```
git clone https://github.com/kslynn128171/FSA && cd FSA && python run_fsa.py --reference pubchem_halogen_extended --datasets six_nonlipid_msms --time-per-spectrum
```

## Evaluation signals

- Recomputed rank#1 accuracy across the six non-lipid datasets falls within tolerance of the reported 77.84%
- Recomputed top-5 accuracy falls within tolerance of the reported 92.60%
- Reference formula list size matches the expected 2,613,203 entries after halogen expansion
- Average runtime is approximately 0.16 s per spectrum (6.67x slower than the CHONSP-only 0.024 s run)
- Final accuracy/runtime comparison is saved as a structured table with per-dataset and pooled values

## Limitations

- Expanding the reference list from CHONSP-only (928,369 formulas) to halogen-inclusive (2,613,203) makes FSA 6.67x slower (0.16 s vs 0.024 s per spectrum)
- FSA's rank#1 accuracy is not yet comparable to state-of-the-art tools such as Buddy and SIRIUS because it does not utilize structural information
- Few chemical rules are implemented to filter unfeasible formulas, and species/adduct-specific rules are not incorporated, so ranking accuracy can degrade for certain compound species
- FSA used an outdated PubChem snapshot (109,383,767 compounds vs 112,419,402 as of 2022/11/27), which affects reference-list comparability

## Evidence

- [abstract] Top-1/top-5 baseline reported in the paper: "FSA ranked the correct chemical formula as the top-1 candidate for a metabolite in 85.28% of the cases and in the top-5 candidates in 97.35% of the cases."
- [abstract] Per-spectrum runtime baseline: "The average processing time for each spectrum was 0.024 s."
- [discussion] Multi-dataset evaluation scope of the tool being benchmarked: "The FSA demonstrated effective and efficient performance in generating candidate molecular formulas across eight test datasets"
