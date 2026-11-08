---
name: hierarchical-library-querying
description: Use when benchmarking or performing lipid annotation of raw/benchmark
  MS2 spectra where recall@Top-20, coverage of chain compositions and C=C positional
  isomers, and false-positive control via relative RT rules are the goals.
when_to_use_negative:
- Input is already an annotated feature table or quantification matrix rather than
  raw/benchmark MS2 spectra — annotation querying does not apply.
- Analysis targets non-lipid small molecules outside the ~121 lipid subclasses covered
  by LipidIN's hierarchical library.
- The goal is C=C double-bond localization of Paternò–Büchi (PB) derivatized data
  via the dedicated PB module — that requires the PB-specific .pkl libraries, not
  the general hierarchical query.
tools:
- name: LipidIN
  role: Provides the EQ querying module, the hierarchical theoretical library (pos_ALL.rda
    / neg_ALL.rda), and the LCI relative-RT rule module used for annotation.
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Comparator annotation tool and source of the public spectral library used
    as the ~70% recall@Top-20 baseline for EQ-only querying.
- name: Flash Entropy Search
  role: Baseline similarity-search algorithm whose squared time complexity LipidIN's
    EQ module (hash tables + bisection) is designed to outperform.
  repo: https://github.com/YuanyueLi/FlashEntropySearch
- name: Spectral Entropy
  role: Comparator similarity-algorithm tool run on identical benchmark spectra under
    recommended default settings.
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: LipidMatch
  role: Comparator lipid annotation tool run on identical benchmark spectra under
    recommended default settings.
  repo: https://github.com/GarrettLab-UF/LipidMatch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:a1d9e6479f4d0c78476678cc725676ddc9eacf7722d8964189efb2a61af5503e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - Flash Entropy Search
  - LipidIN
  - LipidMatch
  - LipidSearch
  - MS-DIAL
  - Spectral Entropy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# hierarchical-library-querying

## Summary

Annotates LC-MS/MS lipid spectra by querying experimental MS2 against LipidIN's hierarchical theoretical fragmentation library (168.6 million entries covering all chain compositions and C=C double-bond positions) via the EQ module, optionally combined with LCI relative-RT rules to suppress false positives.

## When to use

Use when benchmarking or performing lipid annotation of raw/benchmark MS2 spectra where recall@Top-20, coverage of chain compositions and C=C positional isomers, and false-positive control via relative RT rules are the goals.

## When NOT to use

- Input is already an annotated feature table or quantification matrix rather than raw/benchmark MS2 spectra — annotation querying does not apply.
- Analysis targets non-lipid small molecules outside the ~121 lipid subclasses covered by LipidIN's hierarchical library.
- The goal is C=C double-bond localization of Paternò–Büchi (PB) derivatized data via the dedicated PB module — that requires the PB-specific .pkl libraries, not the general hierarchical query.

## Inputs

- Benchmark MS/MS spectra (LC-MS/MS lipidomics, e.g. from Orbitrap/Q-TOF acquisitions)
- LipidIN R-native hierarchical library files pos_ALL.rda / neg_ALL.rda (Zenodo 13350719)
- LipidIN Python .pkl libraries (pos_common.pkl, neg_common.pkl, PB_PX_H.pkl etc.) from Zenodo 21421866
- MS-DIAL-format public spectral library (Zenodo 13350719) for baseline comparison

## Outputs

- Ranked lipid annotation candidates (top-20) per input MS2 spectrum with chain compositions and C=C double-bond positions
- recall@Top-20 values per annotation method
- Per-lipid-subclass stratified recall breakdown (glycerophospholipids, glycerolipids, sphingolipids, fatty acids, etc.)
- Estimated annotation false discovery rate (~5.7% with EQ+LCI)

## How to apply

Download the R-native hierarchical library (pos_ALL.rda, neg_ALL.rda) or the MS-DIAL-format libraries from Zenodo record 13350719 (general/PB Python .pkl libraries are at Zenodo 21421866). Load the benchmark MS/MS spectra into the LipidIN for R framework and run the EQ querying module — an MS2 library search against the theoretical library — walking the 1st- to 4th-level hierarchy of the library (all chain compositions + C=C double-bond positions). For false-positive suppression, pair EQ with the LCI module, which applies the three relative retention time rules (ECN, IUP, ESCN) via best-first-search (BFS) node-expansion ordering. Compute recall@Top-20 per spectrum, counting a hit when the true lipid identity appears in the top-20 ranked candidates, and stratify results by lipid subclass (glycerophospholipids, glycerolipids, sphingolipids, fatty acids). Rationale: EQ+LCI on the 1st–4th-level hierarchical library achieved >90% recall@Top-20 versus ~70% for EQ against the MS-DIAL public library alone, and the EQ hash-table/bisection design outperforms squared-time-complexity flash entropy search.

## Related tools

- **LipidIN** (Provides the EQ querying module, the hierarchical theoretical library (pos_ALL.rda / neg_ALL.rda), and the LCI relative-RT rule module used for annotation.) — https://github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Comparator annotation tool and source of the public spectral library used as the ~70% recall@Top-20 baseline for EQ-only querying.)
- **Flash Entropy Search** (Baseline similarity-search algorithm whose squared time complexity LipidIN's EQ module (hash tables + bisection) is designed to outperform.) — https://github.com/YuanyueLi/FlashEntropySearch
- **Spectral Entropy** (Comparator similarity-algorithm tool run on identical benchmark spectra under recommended default settings.) — https://github.com/YuanyueLi/SpectralEntropy
- **LipidMatch** (Comparator lipid annotation tool run on identical benchmark spectra under recommended default settings.) — https://github.com/GarrettLab-UF/LipidMatch

## Examples

```
load("pos_ALL.rda"); load("neg_ALL.rda")  # R-native hierarchical libraries (Zenodo 13350719) loaded into the LipidIN for R framework before EQ benchmark querying
```

## Evaluation signals

- EQ+LCI with the 1st- to 4th-level hierarchical library should yield recall@Top-20 above 90% on the benchmark spectra.
- EQ against the MS-DIAL public library alone should yield approximately 70% recall@Top-20 — a large drop from this value suggests misconfiguration of the library level or querying module.
- The per-subclass recall ordering (glycerophospholipids vs. glycerolipids vs. sphingolipids etc.) should reproduce the published benchmark pattern.
- Estimated annotation false discovery rate should be near 5.7% when LCI's three relative RT rules are active.
- Query throughput sanity check: EQ should handle roughly 70 billion spectral comparisons in under 1 second.

## Limitations

- Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy may cause feature redundancy; EQ+LCI's advantage depends on the hierarchical library plus RT rules, not similarity scoring alone.
- The R implementation (pos_ALL.rda / neg_ALL.rda track) is kept for reproducibility only and is no longer updated; new work should use the Python LipidIN × Agent track.
- Requires a minimum of 16 GB RAM; querying ~10^8-entry libraries is memory-intensive.
- The deeper relationship between metabolites and relative RT remains uninvestigated, so LCI's three-rule RT filtering is an empirical heuristic with estimated (not measured) FDR.

## Evidence

- [abstract] LipidIN hierarchical library coverage: "LipidIN features a 168.5-million lipid fragmentation hierarchical library that encompasses all potential chain compositions and carbon-carbon double bond locations."
- [methods] Annotation workflow definition: "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)"
- [methods] Original library download location: "the original hierarchical / MS-DIAL libraries are at https://zenodo.org/records/13350719"
- [discussion] Benchmark recall values: "the combination of EQ and MS-DIAL public database also showed superior performance to MS-DIAL program"
- [readme] Library sizes and RAM requirements from README: "Theoretical library size | 168.6 million fragmentation entries (all chain compositions + C=C double-bond positions)"
- [discussion] EQ efficiency rationale: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
