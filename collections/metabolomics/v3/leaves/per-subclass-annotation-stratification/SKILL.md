---
name: per-subclass-annotation-stratification
description: Use when comparing MS2 annotation tools (LipidIN EQ+LCI, MS-DIAL, LipidSearch,
  Spectral Entropy, LipidMatch) on the same benchmark spectra and you need the reported
  ~70% vs >90% recall@Top-20 reproduced and broken down by lipid subclass.
when_to_use_negative:
- The task is producing annotations for a new biological sample rather than benchmarking
  annotation algorithms against known-identity spectra
- There are no ground-truth lipid identities available for the spectra, since recall@Top-20
  cannot be computed without true-identity labels
- You need coverage of features beyond lipid annotation scope, e.g. quantification,
  pathway enrichment or PB double-bond localization workflows, which are separate
  LipidIN steps
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: LipidIN
  role: Provides EQ annotation module, LCI module with ECN/IUP/ESCN relative RT rules,
    and the hierarchical .rda libraries against which recall@Top-20 is computed
  repo: github.com/LinShuhaiLAB/LipidIN
- name: MS-DIAL
  role: Comparator annotation tool run under default settings; its public library
    also serves as the reference library for the EQ-alone configuration
- name: LipidSearch
  role: Comparator annotation tool run on identical benchmark spectra under recommended
    default settings
- name: Spectral Entropy
  role: Comparator annotation tool whose MS/MS similarity scores rank candidate lipids
    for recall@Top-20
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: LipidMatch
  role: Comparator open-source lipid annotation tool matching experimental fragment
    m/z against in-silico fragmentation libraries
  repo: https://github.com/GarrettLab-UF/LipidMatch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:701d70baec8b659e7e9bfced36a2f6d1d9843e296f2cc30ee64acc06ee5ca624
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# per-subclass-annotation-stratification

## Summary

Benchmark lipid annotation tools by computing recall@Top-20 per lipid subclass, counting a hit when the true lipid identity appears among the top-20 ranked candidates, so that coverage differences (e.g. glycerophospholipids vs sphingolipids) are visible rather than hidden in a single aggregate score.

## When to use

Use when comparing MS2 annotation tools (LipidIN EQ+LCI, MS-DIAL, LipidSearch, Spectral Entropy, LipidMatch) on the same benchmark spectra and you need the reported ~70% vs >90% recall@Top-20 reproduced and broken down by lipid subclass.

## When NOT to use

- The task is producing annotations for a new biological sample rather than benchmarking annotation algorithms against known-identity spectra
- There are no ground-truth lipid identities available for the spectra, since recall@Top-20 cannot be computed without true-identity labels
- You need coverage of features beyond lipid annotation scope, e.g. quantification, pathway enrichment or PB double-bond localization workflows, which are separate LipidIN steps

## Inputs

- Benchmark MS/MS spectra (LC-MS/MS lipidomics peak lists)
- LipidIN R-native hierarchical library files pos_ALL.rda and neg_ALL.rda (Zenodo record 13350719)
- MS-DIAL library MS/MS spectra from Zenodo record 13350719
- Reference lipid identities for the benchmark spectra with subclass labels (glycerophospholipids, glycerolipids, sphingolipids, fatty acids, etc.)
- Candidate-ranked annotation outputs from each tool (LipidIN EQ+LCI, LipidIN EQ alone, MS-DIAL, LipidSearch, Spectral Entropy, LipidMatch)

## Outputs

- Per-tool aggregate recall@Top-20 values (~70% expected for EQ+MS-DIAL library, >90% for EQ+LCI)
- Per-subclass recall@Top-20 breakdown per tool
- Comparison table of the tested annotation tools with per-subclass ordering

## How to apply

Load the benchmark MS/MS spectra into the LipidIN for R framework together with the R-native hierarchical library (pos_ALL.rda, neg_ALL.rda) or the MS-DIAL library from Zenodo record 13350719. Annotate once with the EQ+LCI modules querying the 1st- to 4th-level hierarchical library — LCI applies its three relative retention time rules (ECN, IUP, ESCN) to suppress false positives — and once with EQ alone against the MS-DIAL public library. Run each comparator tool (MS-DIAL, LipidSearch, Spectral Entropy, LipidMatch) on the identical benchmark spectra under their recommended default settings. For every method compute recall@Top-20: score a candidate list as correct only if the true lipid identity is ranked within the top 20, and aggregate the per-hit outcomes separately within each lipid subclass (glycerophospholipids, glycerolipids, sphingolipids, fatty acids, etc.). Accept the reproduction when the aggregate values match the published benchmarks (~70% recall for EQ with the MS-DIAL library, >90% for EQ+LCI) within tolerance AND the per-subclass ordering reproduces the published pattern.

## Related tools

- **LipidIN** (Provides EQ annotation module, LCI module with ECN/IUP/ESCN relative RT rules, and the hierarchical .rda libraries against which recall@Top-20 is computed) — github.com/LinShuhaiLAB/LipidIN
- **MS-DIAL** (Comparator annotation tool run under default settings; its public library also serves as the reference library for the EQ-alone configuration)
- **LipidSearch** (Comparator annotation tool run on identical benchmark spectra under recommended default settings)
- **Spectral Entropy** (Comparator annotation tool whose MS/MS similarity scores rank candidate lipids for recall@Top-20) — https://github.com/YuanyueLi/SpectralEntropy
- **LipidMatch** (Comparator open-source lipid annotation tool matching experimental fragment m/z against in-silico fragmentation libraries) — https://github.com/GarrettLab-UF/LipidMatch

## Examples

```
Rscript -e 'load("pos_ALL.rda"); load("neg_ALL.rda");'  # load LipidIN R-native hierarchical libraries for recall@Top-20 stratified annotation benchmark
```

## Evaluation signals

- Aggregate recall@Top-20 for EQ+LCI on the 1st- to 4th-level hierarchical library exceeds 90%, matching the published value within tolerance
- Aggregate recall@Top-20 for EQ querying the MS-DIAL public library is approximately 70%, matching the reported benchmark
- Per-subclass recall ordering (glycerophospholipids, glycerolipids, sphingolipids, fatty acids, etc.) reproduces the published pattern
- A candidate is scored as a hit only when the true lipid identity appears within the top-20 ranked candidates for that spectrum
- All comparator tools were run on the identical benchmark spectra under their recommended default settings

## Limitations

- Similarity-based comparators can produce feature redundancy: 'Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy'
- Recall@Top-20 stratification requires ground-truth lipid identities for every benchmark spectrum; spectra lacking true-identity labels cannot be scored
- Subclass-level breakdown reduces per-cell sample size, so sparse subclasses may show unstable recall estimates

## Evidence

- [results] EQ combined with the LCI module, using the 1st- to 4th-level hierarchical library, achieved over 90% recall@Top-20, outperforming the other tested annotation tools by leveraging the three relative RT rules (ECN, IUP and ESCN).: [evidence span withheld — anchor status: not_found]
- [abstract] developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate, covering 8923 lipids cross: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate, covering 8923 lipids cross"
- [methods] R-native | `.rda` (`pos_ALL.rda`, `neg_ALL.rda`): "R-native | `.rda` (`pos_ALL.rda`, `neg_ALL.rda`)"
- [readme] Annotation — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions).: "Annotation — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)."
- [discussion] Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy.: "Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy."
