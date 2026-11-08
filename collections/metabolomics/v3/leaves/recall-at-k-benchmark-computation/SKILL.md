---
name: recall-at-k-benchmark-computation
description: Use when benchmarking MS/MS lipid annotation performance of two or more
  tools (e.g., LipidIN EQ+LCI vs. MS-DIAL/LipidMatch) on identical spectra, and you
  need a top-k ranked-candidate recall metric stratified by lipid subclass.
when_to_use_negative:
- When the goal is plain spectral library searching or similarity scoring (e.g., flash
  entropy search itself) rather than evaluating a tool's ranked annotation output
  against ground truth.
- When spectra lack known ground-truth lipid identities — recall@k requires a true
  identity to count hits.
- When retention-time-aware rules (LCI's ECN/IUP/ESCN) cannot be applied because benchmark
  spectra were acquired without RT information.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: LipidIN
  role: Tool under test and benchmark driver; EQ+LCI querying the hierarchical library
    and EQ against MS-DIAL library produce ranked candidate lists
  repo: github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Comparator annotation tool run with recommended default settings; its public
    library also serves as the EQ-alone search target
- name: LipidSearch
  role: Comparator annotation tool run with recommended default settings
- name: Spectral Entropy
  role: Comparator similarity-based annotation approach
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: LipidMatch
  role: Comparator in-silico fragmentation annotation tool run with recommended default
    settings
  repo: https://github.com/GarrettLab-UF/LipidMatch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:7170f82598c275ccab3b06d1ae605d7238543d05021c09a6af2b5aa52c9c71e1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - LipidMatch
  - LipidSearch
  - MS-DIAL
  - Spectral Entropy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# recall-at-k-benchmark-computation

## Summary

Computes recall@Top-k annotation benchmarks for lipid annotation tools by counting how often the true lipid identity appears among the top-k ranked candidates, as used to compare LipidIN's EQ+LCI modules against MS-DIAL, LipidSearch, Spectral Entropy, and LipidMatch.

## When to use

Use when benchmarking MS/MS lipid annotation performance of two or more tools (e.g., LipidIN EQ+LCI vs. MS-DIAL/LipidMatch) on identical spectra, and you need a top-k ranked-candidate recall metric stratified by lipid subclass.

## When NOT to use

- When the goal is plain spectral library searching or similarity scoring (e.g., flash entropy search itself) rather than evaluating a tool's ranked annotation output against ground truth.
- When spectra lack known ground-truth lipid identities — recall@k requires a true identity to count hits.
- When retention-time-aware rules (LCI's ECN/IUP/ESCN) cannot be applied because benchmark spectra were acquired without RT information.

## Inputs

- Benchmark LC-MS/MS spectra of lipids with known ground-truth annotations (positive and negative ion mode)
- Annotated candidate lists from LipidIN EQ+LCI querying pos_ALL.rda / neg_ALL.rda hierarchical libraries
- Annotated candidate lists from LipidIN EQ alone querying the MS-DIAL public library
- Annotated candidate lists from MS-DIAL, LipidSearch, Spectral Entropy, and LipidMatch run with default settings
- Lipid subclass labels for stratification (e.g., glycerophospholipids, glycerolipids, sphingolipids, fatty acids)

## Outputs

- recall@Top-20 value per annotation tool
- Per-lipid-subclass breakdown of recall@Top-20
- Comparison table against reported benchmark values (EQ ~70%, EQ+LCI >90%)

## How to apply

Run every annotation tool (LipidIN EQ+LCI with the hierarchical 1st- to 4th-level library, EQ alone against the MS-DIAL public library, plus MS-DIAL, LipidSearch, Spectral Entropy, and LipidMatch) on the same benchmark MS/MS spectra, applying each comparator tool's recommended default settings. For each query spectrum, collect the ranked candidate list from each tool and count a hit when the true lipid identity appears among the top-20 ranked candidates; recall@Top-20 = number of hits / number of benchmark spectra. Apply LipidIN's LCI module with its three relative retention time rules (ECN, IUP, ESCN) to suppress false positives before scoring, since RT-informed scoring was hypothesized to improve annotation beyond MS/MS similarity alone. Stratify results by lipid subclass (glycerophospholipids, glycerolipids, sphingolipids, fatty acids, etc.) to reproduce per-subclass orderings. Validate against the paper's reference values: approximately 70% recall@Top-20 for EQ with the MS-DIAL library and above 90% for EQ+LCI.

## Related tools

- **LipidIN** (Tool under test and benchmark driver; EQ+LCI querying the hierarchical library and EQ against MS-DIAL library produce ranked candidate lists) — github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Comparator annotation tool run with recommended default settings; its public library also serves as the EQ-alone search target)
- **LipidSearch** (Comparator annotation tool run with recommended default settings)
- **Spectral Entropy** (Comparator similarity-based annotation approach) — https://github.com/YuanyueLi/SpectralEntropy
- **LipidMatch** (Comparator in-silico fragmentation annotation tool run with recommended default settings) — https://github.com/GarrettLab-UF/LipidMatch

## Examples

```
recall20 = sum(1 for r, cand in enumerate(ranked, 1) if r <= 20 and cand == true_id) / n_queries
```

## Evaluation signals

- recall@Top-20 for EQ+LCI exceeds 90%; EQ with MS-DIAL library is approximately 70%; values match reported benchmarks within tolerance
- Per-subclass ordering of recall values reproduces the published stratification pattern across glycerophospholipids, glycerolipids, sphingolipids, and fatty acids
- Every tool is scored on identical benchmark spectra with identical hit-counting logic (true identity within top-20 ranked candidates)
- EQ+LCI consistently outperforms all comparators when LCI's three relative RT rules (ECN, IUP, ESCN) are enabled

## Limitations

- Similarity-based comparators (MS-DIAL, LipidSearch, MS entropy) may inflate apparent recall through feature redundancy rather than correct unique annotation.
- LCI's RT-rule-based false-positive suppression depends on its estimated 5.7% false discovery rate across 8,923 lipids; recall gains assume retention order information is available for the benchmark samples.
- LipidMatch does not support Waters files, limiting comparability of benchmark data formats across instruments.

## Evidence

- [intro] and that combining MS/MS- and RT-based scores could significantly improve annotation performance in LC-MS/MS-based lipidomics analyses: "and that combining MS/MS- and RT-based scores could significantly improve annotation performance in LC-MS/MS-based lipidomics analyses"
- [discussion] Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy.: "Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy."
- [methods] Annotation — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions): "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)"
- [abstract] developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate"
- [readme] The software does not currently support Waters files.: [evidence span withheld — anchor status: external_doc]
