---
name: in-silico-annotation-benchmarking
description: Use when evaluating or reproducing mass spectral search method performance
  on a benchmark set with known true molecules, such as scoring spectra against a
  structure database and computing top-k identification accuracy.
when_to_use_negative:
- Input spectra lack known ground-truth molecule assignments, so top-k accuracy cannot
  be computed against a reference
- The goal is de novo structure prediction or spectral library search rather than
  scoring spectra against a structure database
- Molecules are outside the method's supported mass/fragmentation range, e.g., super
  small molecules (<400 Da) or heavy molecules (>1000 Da), where the method is reported
  to perform poorly or be computationally insufficient
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molDiscovery
  role: Mass spectral database search method used to score each benchmark spectrum
    against all candidate structures in the reference database and rank them by matching
    score
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:0e7173fcb663a77da25af17d8a3bf09eb3b31abb5b41e538421856ab53407e32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# in-silico-annotation-benchmarking

## Summary

Benchmarks in-silico mass spectral database search methods (e.g., molDiscovery) on annotated spectral libraries against structure databases by ranking candidate molecules and computing top-k identification accuracies, validating reproduction of reported benchmark values.

## When to use

Use when evaluating or reproducing mass spectral search method performance on a benchmark set with known true molecules, such as scoring spectra against a structure database and computing top-k identification accuracy.

## When NOT to use

- Input spectra lack known ground-truth molecule assignments, so top-k accuracy cannot be computed against a reference
- The goal is de novo structure prediction or spectral library search rather than scoring spectra against a structure database
- Molecules are outside the method's supported mass/fragmentation range, e.g., super small molecules (<400 Da) or heavy molecules (>1000 Da), where the method is reported to perform poorly or be computationally insufficient

## Inputs

- 15,917 positive-mode [M+H]+ spectra from the PNNL lipid library benchmark set
- 39,126-compound LMSD (Lipid Maps Structure Database) structure database
- molDiscovery pretrained fragmentation model

## Outputs

- Top-1/3/5/10 identification accuracies per benchmark run
- lipid_benchmark_accuracies.csv containing the four computed accuracy values

## How to apply

Load a benchmark spectral library with known ground-truth molecule assignments (e.g., 15,917 positive-mode [M+H]+ spectra from the PNNL lipid library). Load a reference structure database to search against (e.g., 39,126-compound LMSD/Lipid Maps). Run the search tool in pretrained-model database-search mode to score each spectrum against all candidate structures, rank candidates by matching score, then compute top-1/3/5/10 accuracies where a hit counts as correct when a top-ranked structure matches the known true molecule. Validate that computed accuracies fall within the paper's reported values before declaring the benchmark reproduced.

## Related tools

- **molDiscovery** (Mass spectral database search method used to score each benchmark spectrum against all candidate structures in the reference database and rank them by matching score) — github.com/mohimanilab/molDiscovery

## Examples

```
molDiscovery --mode pretrained-search --spectra pnnl_lipid_library.mgf --database LMSD.tsv --output lipid_benchmark_accuracies.csv
```

## Evaluation signals

- Computed top-1 accuracy matches reported 16.9% on PNNL lipid library vs LMSD
- Computed top-3 accuracy matches reported 37.4%
- Computed top-5 accuracy matches reported 47.6%
- Computed top-10 accuracy matches reported 63.0%
- All four accuracy values written correctly to lipid_benchmark_accuracies.csv

## Limitations

- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; default model trained on combined CID/HCD positive-mode spectra
- Lower accuracy in low mass ranges (<600 Da), partly because fewer fragments are generated for small molecules
- No p-value computation for general molecules; MCMC p-value approach exists only for peptidic molecules

## Evidence

- [abstract] Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [results] In the PNNL lipid library search against LMSD, molDiscovery achieved a top 1 accuracy of 16.9%, a top 3 accuracy of 37.4%, a top 5 accuracy of 47.6%, and a top 10 accuracy of 63.0%.: [evidence span withheld — anchor status: not_found]
- [intro] the existing methods do not perform well for super small molecules (<400 Da), and are computationally insufficient for heavy small molecules (>1000 Da): "the existing methods do not perform well for super small molecules (<400 Da), and are computationally insufficient for heavy small molecules (>1000 Da)"
- [discussion] Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc.: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
- [readme] For more information, please contact <maintainer e-mail, see the repository README>: "For more information, please contact <maintainer e-mail, see the repository README>"
