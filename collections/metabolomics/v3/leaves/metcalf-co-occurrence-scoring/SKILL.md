---
name: metcalf-co-occurrence-scoring
description: Use when generating an initial set of candidate gene cluster family (GCF)
  to MS/MS spectrum links from paired genome–metabolome data before applying class-based
  filtering such as NPClassScore.
when_to_use_negative:
- When the goal is already class-matched, prioritised links — apply NPClassScore filtering
  (cut-off 0.25) on top of the Metcalf results instead of stopping at co-occurrence
- When no strain-level mapping between genomes and metabolomics samples is available,
  since co-occurrence scoring requires paired occurrence profiles
- When validated links are known to have low standardised Metcalf scores (e.g. the
  Nocardia dataset, where the threshold had to be lowered from 2.5 to 2)
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: NPLinker
  role: Platform that computes the standardised Metcalf co-occurrence score between
    GCFs and spectra and exposes get_links for link retrieval
  repo: github.com/NPLinker/nplinker
- name: BiG-SCAPE
  role: Groups antiSMASH BGCs into gene cluster families (GCFs) that are the units
    scored against spectra
- name: GNPS
  role: Source of the molecular networking output (spectra and strain occurrence data)
    ingested by NPLinker for scoring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:ac896274e986f391ba36ef0861d7c3ca15e990a68dcd335540e219a70e3f5bbc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - GNPS
  - NPLinker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metcalf-co-occurrence-scoring

## Summary

Standardised Metcalf co-occurrence scoring within NPLinker generates the initial candidate BGC–MS/MS spectrum links by correlating GCF occurrence across strains with spectrum presence, using a cut-off of 2.5. It serves as the unfiltered baseline that chemical-class methods such as NPClassScore are designed to prune.

## When to use

Use when generating an initial set of candidate gene cluster family (GCF) to MS/MS spectrum links from paired genome–metabolome data before applying class-based filtering such as NPClassScore.

## When NOT to use

- When the goal is already class-matched, prioritised links — apply NPClassScore filtering (cut-off 0.25) on top of the Metcalf results instead of stopping at co-occurrence
- When no strain-level mapping between genomes and metabolomics samples is available, since co-occurrence scoring requires paired occurrence profiles
- When validated links are known to have low standardised Metcalf scores (e.g. the Nocardia dataset, where the threshold had to be lowered from 2.5 to 2)

## Inputs

- GNPS molecular networking data (mgf/mzXML and network files) for the paired dataset
- antiSMASH/BiG-SCAPE gene cluster family annotations for the strains (BiG-SCAPE cut-off 0.3)
- Strain mappings pairing genomes with MS/MS samples (e.g. from PoDP accessions)
- NPLinker configuration file (nplinker.toml)

## Outputs

- Link graph of candidate GCF–MS/MS spectrum links with standardised Metcalf scores ≥ 2.5
- Per-GCF candidate link counts for the pre-filtering histogram

## How to apply

Load the GNPS molecular networking data, antiSMASH/BiG-SCAPE GCF definitions, and strain mappings into NPLinker via load_config/load_data. Run standardised Metcalf scoring across all GCFs and spectra, retaining candidate links with a standardised Metcalf score above the cut-off of 2.5. Count retained links per GCF to build the pre-filtering histogram — for the Streptomyces/Salinispora dataset (Zenodo record 10822604) this yields 5 GCFs with ≤10 links and 42 GCFs with ≤25 links. This co-occurrence-only link set is the baseline against which NPClassScore filtering (cut-off 0.25) is compared; for datasets with low Metcalf scores for validated links, e.g. Nocardia, the threshold may be lowered to 2 to retain known links.

## Related tools

- **NPLinker** (Platform that computes the standardised Metcalf co-occurrence score between GCFs and spectra and exposes get_links for link retrieval) — github.com/NPLinker/nplinker
- **BiG-SCAPE** (Groups antiSMASH BGCs into gene cluster families (GCFs) that are the units scored against spectra)
- **GNPS** (Source of the molecular networking output (spectra and strain occurrence data) ingested by NPLinker for scoring)

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- On the Streptomyces/Salinispora dataset, the co-occurrence-only histogram should show 5 GCFs with ≤10 candidate links and 42 GCFs with ≤25 candidate links
- All retained candidate links must have standardised Metcalf scores above the cut-off of 2.5
- After subsequent NPClassScore filtering at 0.25 the counts should rise to 92 GCFs with ≤10 links and 270 GCFs with ≤25 links, i.e. the Metcalf-only set should be far larger
- The link graph returned by get_links should cover GCFs × spectra for the loaded dataset, e.g. npl.gcfs[:3]
- Average candidate links per GCF should decrease by roughly 63% once class matching is applied to the Metcalf baseline

## Limitations

- Co-occurrence-only scoring produces very long candidate lists — only 5 GCFs have ≤10 candidate links (42 with ≤25) before NPClassScore filtering
- For small datasets, validated links may fall below the 2.5 Metcalf cut-off, requiring threshold lowering (e.g. to 2 for Nocardia)
- Candidate links between GCFs and spectra of strains lacking RefSeq/GenBank identifiers cannot be fully reconstructed

## Evidence

- [discussion] Metcalf cut-off of 2.5 and NPClassScore cut-off of 0.25: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [methods] NPLinker API for retrieving Metcalf links: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [discussion] Lowering Metcalf threshold for Nocardia: "we changed the standardised Metcalf threshold from 2.5 to 2 for this dataset to include the validated links"
- [readme] Class matching builds on co-occurrence-based linking in NPLinker: [evidence span withheld — anchor status: external_doc]
