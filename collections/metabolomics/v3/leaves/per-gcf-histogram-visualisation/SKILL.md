---
name: per-gcf-histogram-visualisation
description: Use when you need to quantify and visualise how a link-scoring or filtering
  step (e.g. NPClassScore at a 0.25 cut-off) changes the per-GCF distribution of candidate
  BGC–MS/MS links, e.g. counting GCFs with ≤10 or ≤25 candidate links.
when_to_use_negative:
- When the analysis goal is ranking or prioritising individual validated links rather
  than summarising per-GCF candidate-link burden across the whole dataset
- When candidate links have not been scored (e.g. no Metcalf scores computed in NPLinker),
  so there is no link set to count per GCF
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Platform used to score co-occurrence links (standardised Metcalf), apply NPClassScore
    filtering, and expose per-GCF link sets via get_links for counting
  repo: https://github.com/NPLinker/nplinker
- name: NPClassScore
  role: Class-matching score whose 0.25 cut-off defines the post-filtering link set
    being histogrammed
  repo: https://github.com/NPLinker/nplinker
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:c094652f38b1dd9e391e340d243b8225958301a78efe2c030b31dd292ca7e00c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - NPClassScore
  - NPLinker
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# per-gcf-histogram-visualisation

## Summary

Builds and plots side-by-side histograms of the number of candidate BGC–MS/MS spectrum links per gene cluster family (GCF), before and after NPClassScore filtering, to quantify how class matching reduces candidate link lists to manageable sizes.

## When to use

Use when you need to quantify and visualise how a link-scoring or filtering step (e.g. NPClassScore at a 0.25 cut-off) changes the per-GCF distribution of candidate BGC–MS/MS links, e.g. counting GCFs with ≤10 or ≤25 candidate links.

## When NOT to use

- When the analysis goal is ranking or prioritising individual validated links rather than summarising per-GCF candidate-link burden across the whole dataset
- When candidate links have not been scored (e.g. no Metcalf scores computed in NPLinker), so there is no link set to count per GCF

## Inputs

- Candidate BGC–MS/MS spectrum link set from NPLinker after standardised Metcalf scoring (cut-off 2.5)
- Filtered candidate link set after NPClassScore filtering (cut-off 0.25, links without structure-based classes discarded)
- GCF objects from NPLinker (BiG-SCAPE-derived gene cluster families)

## Outputs

- Per-GCF link-count histogram before filtering (matplotlib)
- Per-GCF link-count histogram after NPClassScore filtering (matplotlib)
- Counts of GCFs with manageable numbers of candidate links (≤10 and ≤25 links) for both conditions

## How to apply

After running standardised Metcalf co-occurrence scoring (cut-off 2.5) over all GCFs and spectra in NPLinker, count candidate links per GCF to build the pre-filtering histogram. Then apply NPClassScore chemical-class matching — using CANOPUS/MolNetEnhancer structure-based classes for spectra and MIBiG-derived classes for BGCs — discard candidate links without structure-based classes, retain only links with NPClassScore ≥ 0.25, and re-count links per GCF for the post-filtering histogram. Plot both per-GCF link-count histograms side by side with matplotlib, highlighting the number of GCFs falling within 'manageable' link-count bins such as ≤10 and ≤25 links. For the Streptomyces/Salinispora dataset this comparison is the paper's headline result: 5 vs 92 GCFs at ≤10 links and 42 vs 270 GCFs at ≤25 links.

## Related tools

- **NPLinker** (Platform used to score co-occurrence links (standardised Metcalf), apply NPClassScore filtering, and expose per-GCF link sets via get_links for counting) — https://github.com/NPLinker/nplinker
- **NPClassScore** (Class-matching score whose 0.25 cut-off defines the post-filtering link set being histogrammed) — https://github.com/NPLinker/nplinker

## Examples

```
npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Post-filtering histogram for the Streptomyces/Salinispora dataset shows 92 GCFs with ≤10 candidate links and 270 GCFs with ≤25 candidate links
- Pre-filtering histogram for the same dataset shows 5 GCFs with ≤10 links and 42 GCFs with ≤25 links
- The filtering step should reduce per-GCF link counts (on average ~63% fewer links across datasets), so the post-filtering histogram must shift towards lower counts
- Every GCF in the histograms must map to a BiG-SCAPE-derived gene cluster family present in the loaded NPLinker session

## Limitations

- Discarding candidate links without structure-based classes (1988 MS/MS spectra lacked CANOPUS or MolNetEnhancer predictions) reduces average candidate links per GCF by 49% even without NPClassScore, so the histogram partly reflects class-annotation coverage rather than the score itself
- CANOPUS class predictions are limited by an 850 Da mass cut-off, so larger metabolites are under-represented in the post-filtering histogram

## Evidence

- [discussion] Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [discussion] filter missing classes: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [methods] NPLinker get_links API: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [abstract] average link reduction: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [readme] NPClassScore in NPLinker: [evidence span withheld — anchor status: external_doc]
