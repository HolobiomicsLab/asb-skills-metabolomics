---
name: ms2-spectral-library-search
description: Use when an LC-MS/MS lipidomics feature has a measured MS2 peak list
  (m/z + intensity + precursor m/z) and the goal is library-based annotation of lipid
  species, chain composition, or C=C double-bond position.
when_to_use_negative:
- Input is an MS1-only feature table without MS2 peak lists — no fragment matching
  is possible
- The query is a general metabolite/small-molecule spectrum with no candidate lipid
  theoretical spectra in the library — library search cannot return a meaningful lipid
  annotation
- Quantification or QC (CV/PCA/PLS-DA) is the goal; this skill performs annotation
  only and precedes those steps in the workflow
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: LipidIN
  role: Provides the EQ expeditious-querying module (hash-table/bisection, greedy
    non-informative-prior search) and the 168.6-million-entry theoretical lipid fragmentation
    library used to compute Score_matched and Score_ratio
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: flash entropy search
  role: 'Alternative/baseline similarity engine: build an index over library spectra
    and search query MS2 spectra by entropy similarity'
  repo: https://github.com/YuanyueLi/FlashEntropySearch
- name: Spectral Entropy
  role: Entropy-similarity calculation package (ms-entropy) whose Flash search is
    benchmarked against the EQ module's linear-time lookup
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: LipidMatch
  role: Related open-source lipid annotation tool matching experimental fragment m/z
    to simulated in-silico library m/z values; used as a comparison method
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MS-DIAL
  role: Existing MS/MS-similarity/decision-tree annotation pipeline whose public database
    is combined with EQ for comparison; its similarity approach can cause feature
    redundancy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:86481070d1d6213d87e1d5c997cade63c721aede197ecf384e0b1a3be6c29505
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - LipidMatch
  - MS-DIAL
  - Spectral Entropy
  - flash entropy search
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ms2-spectral-library-search

## Summary

Annotates lipids by matching a measured MS/MS spectrum against a theoretical spectral library (LipidIN's 168.6-million-entry hierarchical library of chain compositions and C=C double-bond positions), scoring candidates with peak-match and intensity-similarity metrics at a stated ppm tolerance.

## When to use

Use when an LC-MS/MS lipidomics feature has a measured MS2 peak list (m/z + intensity + precursor m/z) and the goal is library-based annotation of lipid species, chain composition, or C=C double-bond position.

## When NOT to use

- Input is an MS1-only feature table without MS2 peak lists — no fragment matching is possible
- The query is a general metabolite/small-molecule spectrum with no candidate lipid theoretical spectra in the library — library search cannot return a meaningful lipid annotation
- Quantification or QC (CV/PCA/PLS-DA) is the goal; this skill performs annotation only and precedes those steps in the workflow

## Inputs

- Measured MS/MS spectrum (peak list of m/z and intensity pairs plus precursor m/z) from Thermo .raw, SCIEX .wiff, Bruker/Agilent .d, or .mzML files
- Theoretical spectral library for candidate lipids (LipidIN .pkl files such as pos_common.pkl / neg_common.pkl, R-native .rda files, or in-silico fragment lists)
- Stated mass tolerance in ppm for fragment matching
- Optionally, retention time information for combined MS/MS- + RT-based scoring

## Outputs

- Per-candidate Score_matched (fraction of matched peaks) and Score_ratio (intensity similarity), e.g., written to scores_table.csv
- Entropy similarity scores (identity, hybrid, neutral-loss, open search) per library spectrum
- Lipid annotation: species/subclass assignment including chain composition and C=C double-bond position

## How to apply

Load the theoretical fragment list for each candidate lipid from the library (e.g., LipidIN .pkl files such as pos_common.pkl/neg_common.pkl, or .rda/.msp formats) and the measured MS2 spectrum. Match theoretical fragment m/z values to measured peaks within the stated ppm mass tolerance using a fast lookup — the LipidIN EQ module applies hash tables and bisection methods for linear-time querying, outperforming squared-time flash entropy search. For each candidate compute Score_matched (fraction of matched peaks) and Score_ratio (ratio-based intensity similarity) per Equations 1-2, feeding the score pair into downstream decision trees; combine with RT-based scores to improve annotation. Alternatively, index the library with FlashEntropySearch.build_index() and call search(precursor_mz, peaks), which returns identity, hybrid, neutral-loss, and open-search entropy similarities in [0, 1]. Validate the scorer against the LipidIN reference implementation on deposited test spectra (Zenodo records 21421866/13350719/13735639) before production use.

## Related tools

- **LipidIN** (Provides the EQ expeditious-querying module (hash-table/bisection, greedy non-informative-prior search) and the 168.6-million-entry theoretical lipid fragmentation library used to compute Score_matched and Score_ratio) — https://github.com/LinShuhaiLAB/LipidIN
- **flash entropy search** (Alternative/baseline similarity engine: build an index over library spectra and search query MS2 spectra by entropy similarity) — https://github.com/YuanyueLi/FlashEntropySearch
- **Spectral Entropy** (Entropy-similarity calculation package (ms-entropy) whose Flash search is benchmarked against the EQ module's linear-time lookup) — https://github.com/YuanyueLi/SpectralEntropy
- **LipidMatch** (Related open-source lipid annotation tool matching experimental fragment m/z to simulated in-silico library m/z values; used as a comparison method) — https://github.com/GarrettLab-UF/LipidMatch
- **MS-DIAL** (Existing MS/MS-similarity/decision-tree annotation pipeline whose public database is combined with EQ for comparison; its similarity approach can cause feature redundancy)

## Examples

```
from ms_entropy import FlashEntropySearch; es = FlashEntropySearch(); es.build_index(library); sims = es.search(precursor_mz=prec, peaks=peaks)
```

## Evaluation signals

- Scorer output on the deposited test spectra agrees with the LipidIN reference implementation (expert-review on score agreement)
- Each candidate yields a complete Score_matched/Score_ratio pair in scores_table.csv; Score_matched is a fraction (0–1) bounded by the number of theoretical fragments
- All fragment matches satisfy the stated ppm tolerance — recompute |theoretical − observed|/theoretical × 1e6 and confirm no peak pair exceeds it
- FlashEntropySearch search() returns similarity arrays for all four search modes with values in [0, 1] as shown in the README example
- Query throughput check: EQ-module library search should complete tens of billions of comparisons in about a second on the sized library (e.g., 24,631,702 spectra) with ≥16 GB RAM

## Limitations

- Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy may cause feature redundancy; EQ was introduced partly to address this
- The PB (Paternò–Büchi) spectral library covers only 11 glycerophospholipid/lysophospholipid subclasses; other double-bond-localization queries are out of scope
- Requires ≥16 GB RAM to hold the general library; reverse-lipidomics fingerprint regeneration would benefit from pre-training on a larger, more diverse dataset

## Evidence

- [abstract] We introduce LipidIN, an advanced framework designed for flash platform-independent annotation.: "We introduce LipidIN, an advanced framework designed for flash platform-independent annotation."
- [methods] MS2 library search against the theoretical library: "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)"
- [discussion] EQ module hash-table/bisection linear-time lookup outperforms flash entropy search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [readme] FlashEntropySearch build_index then search usage: "# Step 1: Build the index from the library spectra
entropy_search.build_index(spectral_library)

# Step 2: Search the library"
- [discussion] Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy.: "Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy."
