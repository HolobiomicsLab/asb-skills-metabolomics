---
name: genomics-metabolomics-link-scoring
description: Use when pairing genome-mined BGCs (e.g. antiSMASH GCFs from BiG-SCAPE)
  with GNPS molecular networking MS/MS spectra and you need to prioritise or prune
  candidate BGC–spectrum links without labelled training data.
when_to_use_negative:
- The dataset's metabolites are large (>~850 Da) where CANOPUS coverage is limited
  — Spec2Vec/MS2DeepScore-style methods may be better suited
- You need validated links for compounds without any MS/MS spectrum (as with retimycin
  and nocardimicin in the Nocardia dataset)
- MolNetEnhancer classes are required but you cannot run external GNPS jobs, since
  MolNetEnhancer has to be run outside NPLinker
edam_operation: http://edamontology.org/operation_3754
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: NPLinker
  role: Platform running standardised Metcalf scoring, NPClassScore filtering, and
    the get_links API over GCFs and spectra
  repo: https://github.com/NPLinker/nplinker
- name: NPClassScore
  role: Empirical chemical-class matching score that filters co-occurrence candidate
    links below the 0.25 cut-off
  repo: https://github.com/NPLinker/nplinker
- name: antiSMASH
  role: Provides BGC annotations (v3 and v6 outputs merged) that BiG-SCAPE groups
    into GCFs for scoring
- name: CANOPUS
  role: Structure-based class prediction via SIRIUS on the GNPS mgf file, supplying
    spectrum-side classes for NPClassScore
- name: mibig_classifications
  role: Source of the NPClassScore scoring tables linking MIBiG BGC classes to compound
    classes
  repo: https://github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:e995325f42209250182582fa844292a908e195e1601744a4c10ac265e0c597ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - CANOPUS
  - NPClassScore
  - NPLinker
  - Python
  - antiSMASH
  - mibig_classifications
  edam_operation: http://edamontology.org/operation_3754
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- genomic-metabolomic-link-analysis
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# genomics-metabolomics-link-scoring

## Summary

Two-stage linking of biosynthetic gene clusters (BGCs) to MS/MS spectra in NPLinker: standardised Metcalf co-occurrence scoring followed by NPClassScore chemical-class matching, which reduces candidate BGC–MS/MS links by on average 63%.

## When to use

Use when pairing genome-mined BGCs (e.g. antiSMASH GCFs from BiG-SCAPE) with GNPS molecular networking MS/MS spectra and you need to prioritise or prune candidate BGC–spectrum links without labelled training data.

## When NOT to use

- The dataset's metabolites are large (>~850 Da) where CANOPUS coverage is limited — Spec2Vec/MS2DeepScore-style methods may be better suited
- You need validated links for compounds without any MS/MS spectrum (as with retimycin and nocardimicin in the Nocardia dataset)
- MolNetEnhancer classes are required but you cannot run external GNPS jobs, since MolNetEnhancer has to be run outside NPLinker

## Inputs

- GNPS molecular networking output (mgf and network files) for a paired genome-metabolome dataset
- antiSMASH BGC annotations grouped into GCFs (BiG-SCAPE cut-off 0.3)
- Strain mappings linking genomes to MS/MS sample filenames (e.g. from PoDP accessions MSV000078836/MSV000078839/MSV000079284)
- Structure-based chemical classes for spectra (CANOPUS or MolNetEnhancer predictions)
- MIBiG 2.0 BGC product classifications and NPClassScore scoring tables (github.com/louwenjjr/mibig_classifications)

## Outputs

- Ranked candidate BGC–MS/MS spectrum links with standardised Metcalf scores and NPClassScore values
- Filtered link set retaining only NPClassScore >= 0.25 links with structure-based classes
- Per-GCF link-count distributions before and after NPClassScore filtering (e.g. histograms of GCFs with ≤10 / ≤25 candidate links)

## How to apply

Load the paired dataset (e.g. the Streptomyces/Salinispora Zenodo record 10822604) into NPLinker via load_config/load_data, then run standardised Metcalf co-occurrence scoring with a cut-off of 2.5 over all GCF–spectrum pairs to obtain the unfiltered candidate link set. Structure-based compound classes come from CANOPUS (run via SIRIUS v4.9.3 on the GNPS mgf) or MolNetEnhancer (run externally on the GNPS platform), while BGC classes derive from MIBiG 2.0 (sub)class mappings; candidate links lacking structure-based classes are discarded (filter_missing_scores). Retain only links with NPClassScore >= 0.25 and compare per-GCF link counts before versus after filtering — for the Streptomyces/Salinispora dataset this turns 5 GCFs with ≤10 links into 92, and 42 GCFs with ≤25 links into 270. Rationale: matching NP ontologies across genomics and metabolomics removes false-positive links while retaining 96% of experimentally validated links in the PoDP use cases.

## Related tools

- **NPLinker** (Platform running standardised Metcalf scoring, NPClassScore filtering, and the get_links API over GCFs and spectra) — https://github.com/NPLinker/nplinker
- **NPClassScore** (Empirical chemical-class matching score that filters co-occurrence candidate links below the 0.25 cut-off) — https://github.com/NPLinker/nplinker
- **antiSMASH** (Provides BGC annotations (v3 and v6 outputs merged) that BiG-SCAPE groups into GCFs for scoring)
- **CANOPUS** (Structure-based class prediction via SIRIUS on the GNPS mgf file, supplying spectrum-side classes for NPClassScore)
- **mibig_classifications** (Source of the NPClassScore scoring tables linking MIBiG BGC classes to compound classes) — https://github.com/louwenjjr/mibig_classifications

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- Post-filtering histogram for the Streptomyces/Salinispora dataset shows 92 GCFs with ≤10 candidate links and 270 GCFs with ≤25 links, versus 5 and 42 pre-filtering
- Number of retained candidate links is reduced by roughly 63% on average relative to co-occurrence-only scoring
- 96% of experimentally validated links in the three PoDP datasets are retained and prioritised after NPClassScore filtering
- Candidate links missing CANOPUS or MolNetEnhancer predictions (1988 spectra across datasets) are excluded when filter_missing_scores is set
- Validated-link ranked positions after filtering should not exceed their pre-filtering ranks (prioritisation check)

## Limitations

- 1988 MS/MS spectra across the datasets lacked CANOPUS or MolNetEnhancer predictions and their links were discarded; not filtering them decreases average candidate links per GCF by 49%
- CANOPUS coverage is limited by an 850 Da mass cut-off, so larger metabolites are poorly handled
- The Nocardia dataset required lowering the standardised Metcalf threshold from 2.5 to 2 to retain validated links, indicating weaker performance on the smallest dataset
- Merged antiSMASH 3 and antiSMASH 6 outputs introduce version inconsistency across the 50 strains
- No MS/MS spectrum could be found for retimycin and nocardimicin, so those validated links cannot be scored

## Evidence

- [discussion] metcalf then npclassscore: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [abstract] reduction of links: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [abstract] validated link retention: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [discussion] missing classes discarded: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [readme] scoring tables repo: [evidence span withheld — anchor status: external_doc]
