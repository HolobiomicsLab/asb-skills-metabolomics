---
name: cooccurrence-metcalf-link-computation
description: Use when you have paired genome (antiSMASH/BiG-SCAPE GCFs) and metabolome
  (GNPS molecular networking) data and need to rank candidate BGC–MS/MS links per
  GCF by strain co-occurrence before applying class-based filtering.
when_to_use_negative:
- You only have metabolomics data without strain-resolved genome mining output (antiSMASH/BiG-SCAPE
  GCFs), since Metcalf scoring requires strain co-occurrence across both omics layers.
- You need chemically informed prioritisation rather than the co-occurrence baseline
  — NPClassScore class matching should complement, not replace, this step.
- The dataset lacks sufficient shared strains between genome and metabolome measurements
  for co-occurrence statistics to be meaningful.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0622
tools:
- name: NPLinker
  role: Platform that assembles paired omics data and computes the standardised Metcalf
    co-occurrence links via npl.get_links(npl.gcfs, "metcalf")
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Provides the molecular networking MS/MS data that candidate links are computed
    against
- name: BiG-SCAPE
  role: Groups antiSMASH BGCs into gene cluster families (GCFs) that are the units
    scored by Metcalf
- name: antiSMASH
  role: Genome mining source of the BGCs that feed into GCFs
- name: NPClassScore
  role: Downstream class-matching score (cut-off 0.25) applied to filter the Metcalf
    candidate links
  repo: https://github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:0adb3d700bf9e39a4ca695e8ac56bda859ca5cebbf426a8ea0cafc112e93ad3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - BiG-SCAPE
  - GNPS
  - NPClassScore
  - NPLinker
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cooccurrence-metcalf-link-computation

## Summary

Compute candidate BGC–MS/MS spectrum links using the standardised Metcalf co-occurrence score inside NPLinker, scoring each gene cluster family (GCF) against each GNPS molecular networking spectrum across paired omics datasets. This co-occurrence ranking is the baseline that chemical-class methods such as NPClassScore are designed to complement and filter.

## When to use

Use when you have paired genome (antiSMASH/BiG-SCAPE GCFs) and metabolome (GNPS molecular networking) data and need to rank candidate BGC–MS/MS links per GCF by strain co-occurrence before applying class-based filtering.

## When NOT to use

- You only have metabolomics data without strain-resolved genome mining output (antiSMASH/BiG-SCAPE GCFs), since Metcalf scoring requires strain co-occurrence across both omics layers.
- You need chemically informed prioritisation rather than the co-occurrence baseline — NPClassScore class matching should complement, not replace, this step.
- The dataset lacks sufficient shared strains between genome and metabolome measurements for co-occurrence statistics to be meaningful.

## Inputs

- nplinker.toml configuration file (podp mode with a PoDP dataset id)
- GNPS molecular networking output for the paired MS/MS dataset
- antiSMASH BGC annotations grouped into GCFs by BiG-SCAPE (cut-off 0.3)
- strain mappings linking genome and metabolome samples (e.g. from PoDP accessions)

## Outputs

- Ranked candidate BGC–MS/MS spectrum link list per GCF with standardised Metcalf scores
- Per-GCF candidate link counts before class-based filtering (baseline for reduction percentages)

## How to apply

Instantiate NPLinker with an nplinker.toml configuration in podp mode for the dataset's PoDP identifier (e.g. MSV000078836, MSV000078839, MSV000079284, MSV000084950, or MSV000084771) and call load_data() to assemble GNPS molecular networking, antiSMASH/BiG-SCAPE GCFs, and strain mappings. Compute candidate links with npl.get_links(npl.gcfs, "metcalf"), which scores each GCF–spectrum pair by standardised Metcalf co-occurrence across strains. Apply a Metcalf cut-off of 2.5 to retain candidate links; for small datasets where validated links score low (as in the 11-strain Nocardia dataset), lower the threshold to 2 so that known links are included. The resulting ranked candidate link lists per GCF are then the input to NPClassScore filtering (cut-off 0.25) or are reported directly as the co-occurrence-only baseline (e.g. 27 candidate links per GCF in the Cyanobacteria dataset, 206 in Nocardia).

## Related tools

- **NPLinker** (Platform that assembles paired omics data and computes the standardised Metcalf co-occurrence links via npl.get_links(npl.gcfs, "metcalf")) — https://github.com/NPLinker/nplinker
- **GNPS** (Provides the molecular networking MS/MS data that candidate links are computed against)
- **BiG-SCAPE** (Groups antiSMASH BGCs into gene cluster families (GCFs) that are the units scored by Metcalf)
- **antiSMASH** (Genome mining source of the BGCs that feed into GCFs)
- **NPClassScore** (Downstream class-matching score (cut-off 0.25) applied to filter the Metcalf candidate links) — https://github.com/louwenjjr/mibig_classifications

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs, "metcalf")
```

## Evaluation signals

- Per-dataset candidate link counts per GCF before NPClassScore filtering match the reported baseline (e.g. 27 per GCF for Cyanobacteria, 206 for Nocardia).
- Applying NPClassScore (cut-off 0.25) to the Metcalf links reproduces the reported reductions: 68% (Streptomyces/Salinispora), 53% (Cyanobacteria), 69% (Nocardia), averaging 63%.
- All experimentally validated links in the dataset are retained at the chosen Metcalf threshold; if validated links are missed (as with Nocardia at 2.5), lower the threshold to 2.
- Ranked positions of validated links are recorded before and after filtering so prioritisation changes can be verified.

## Limitations

- The Nocardia dataset exhibited quite low standardised Metcalf scores for the validated links, requiring the threshold to be lowered from 2.5 to 2, indicating performance issues on small datasets.
- Metcalf co-occurrence alone leaves long lists of candidate BGC–MS/MS links per GCF and relies on class-based filtering (NPClassScore) to reduce them.
- Strain mappings combined from multiple PoDP accessions are needed, and genomes are only downloaded from NCBI when RefSeq or GenBank identifiers are available.

## Evidence

- [discussion] Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [methods] get_links method evidence: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [abstract] the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [discussion] we changed the standardised Metcalf threshold from 2.5 to 2 for this dataset to include the validated links: "we changed the standardised Metcalf threshold from 2.5 to 2 for this dataset to include the validated links"
- [methods] load data step: "# load data
npl.load_data()"
