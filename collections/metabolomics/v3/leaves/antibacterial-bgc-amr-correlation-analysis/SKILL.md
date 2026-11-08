---
name: antibacterial-bgc-amr-correlation-analysis
description: Use when the task requires benchmarking newly mined BGCs against known-function
  clusters in MIBiG 2.0, quantifying the fraction of BGCs lacking MIBiG homology,
  or correlating antibacterial BGC abundance with AMR gene counts across metagenomic
  soil samples.
when_to_use_negative:
- Input BGCs are already known-function entries stored in MIBiG — the skill benchmarks
  novel clusters against MIBiG, not MIBiG against itself
- The reference corpus should be candidate BGCs of unknown function such as IMG-ABC
  or antiSMASH-DB, which 'store many thousands' of clusters rather than BGCs 'associated
  with molecules of known chemical structure'
- The task is curating or depositing new BGC entries into MIBiG (accession request,
  extended entry form, Annotathon workflow) rather than consuming it as a homology
  baseline
edam_operation: http://edamontology.org/operation_2419
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MIBiG Repository
  role: Reference corpus of 2021 known-function BGCs used as the homology baseline
    for novelty and antibacterial-BGC identification
  repo: https://github.com/mibig-secmet
- name: antiSMASH
  role: Computational tool used to detect Biosynthetic Gene Clusters in microbial
    genomes before comparison against MIBiG
- name: ClusterCAD
  role: Computer-aided pathway design tool that sources BGC sequence and annotation
    data directly from MIBiG
- name: mibig-api
  role: REST-like web API providing programmatic access to the underlying PostgreSQL
    MIBiG database for bulk retrieval of reference entries
  repo: https://github.com/mibig-secmet/mibig-api/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:a2ca58a9823a7e0a6ee0a586730a437b35a5ea5b696b206aef76b6f753b4e3fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterCAD
  - GitHub
  - MIBiG Repository
  - PostgreSQL
  - antiSMASH
  - mibig-api
  edam_operation: http://edamontology.org/operation_2419
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# antibacterial-bgc-amr-correlation-analysis

## Summary

A downstream-analysis skill that uses the MIBiG 2.0 reference set (2021 curated BGCs of known function) as a homology baseline to assess BGC novelty (Crits-Cristoph et al. soil-bacteria analysis) and to correlate antibacterial BGC abundance with antimicrobial resistance genes across soil metagenomes (Bahram et al.).

## When to use

Use when the task requires benchmarking newly mined BGCs against known-function clusters in MIBiG 2.0, quantifying the fraction of BGCs lacking MIBiG homology, or correlating antibacterial BGC abundance with AMR gene counts across metagenomic soil samples.

## When NOT to use

- Input BGCs are already known-function entries stored in MIBiG — the skill benchmarks novel clusters against MIBiG, not MIBiG against itself
- The reference corpus should be candidate BGCs of unknown function such as IMG-ABC or antiSMASH-DB, which 'store many thousands' of clusters rather than BGCs 'associated with molecules of known chemical structure'
- The task is curating or depositing new BGC entries into MIBiG (accession request, extended entry form, Annotathon workflow) rather than consuming it as a homology baseline

## Inputs

- MIBiG 2.0 JSON-formatted BGC entries (851 new entries; 2021 total curated BGCs), obtained from https://github.com/mibig-secmet or the MIBiG REST API
- antiSMASH-detected BGC annotations from uncultivated soil-bacterial genomes
- Metagenomic fungal BGC calls from soil samples
- Antimicrobial resistance gene counts per soil sample

## Outputs

- Novelty fraction: proportion of uncultivated soil-bacterial BGCs lacking homology to any MIBiG cluster
- Correlation coefficient between antibacterial BGC abundance and AMR gene counts across soils
- Structured CSV table collating novelty fraction, correlation coefficient, and ClusterCAD sourcing confirmation

## How to apply

First obtain MIBiG 2.0 from the mibig-secmet GitHub repository or via the REST-like web API that fronts the underlying PostgreSQL database, and parse the JSON-formatted BGC entries to extract biosynthetic class, taxonomic identifier, and product annotation fields. For the novelty analysis, run homology comparisons of uncultivated soil-bacterial BGCs (typically mined with antiSMASH) against the MIBiG reference set and compute the fraction lacking homology to any known MIBiG cluster, reproducing the Crits-Cristoph et al. framing of 'exceptional novelty'. For the Bahram et al. analysis, perform homology searches of metagenomic fungal BGCs against MIBiG to identify those associated with antibacterial activity, then correlate antibacterial BGC abundance with antimicrobial resistance gene counts across soil samples. Optionally confirm that ClusterCAD sources BGC sequence and annotation data directly from MIBiG when the reference set feeds computer-aided pathway design. Collate novelty fractions, correlation coefficients, and sourcing confirmations into a structured CSV, then check values against the ranges reported in the corresponding publications.

## Related tools

- **MIBiG Repository** (Reference corpus of 2021 known-function BGCs used as the homology baseline for novelty and antibacterial-BGC identification) — https://github.com/mibig-secmet
- **antiSMASH** (Computational tool used to detect Biosynthetic Gene Clusters in microbial genomes before comparison against MIBiG)
- **ClusterCAD** (Computer-aided pathway design tool that sources BGC sequence and annotation data directly from MIBiG)
- **mibig-api** (REST-like web API providing programmatic access to the underlying PostgreSQL MIBiG database for bulk retrieval of reference entries) — https://github.com/mibig-secmet/mibig-api/

## Evaluation signals

- Reproduced novelty fraction (fraction of soil-bacterial BGCs lacking MIBiG homology) falls within the range reported by Crits-Christoph et al.
- Reproduced antibacterial-BGC vs AMR-gene correlation coefficient falls within the range reported by Bahram et al. when compared against the MIBiG 2.0 reference set
- Reference set size matches MIBiG 2.0 (2021 curated BGCs, including the 851 added over five years)
- Parsed entries conform to the MIBiG JSON schema (json-schema.org validation) with biosynthetic class, taxonomic identifier, and product fields populated
- Output CSV contains all three reproduction metrics (novelty fraction, correlation coefficient, ClusterCAD sourcing confirmation) with no missing rows

## Limitations

- Novelty estimates are baseline-dependent: MIBiG contains only BGCs 'associated with molecules of known chemical structure', so most uncultivated soil-bacterial BGCs naturally lack MIBiG homology
- Results are tied to MIBiG version 2.0; older or newer reference sets will shift homology fractions and correlations
- The skill depends on upstream BGC detection quality (e.g. antiSMASH calls) and on MIBiG's manual re-annotation quality; the 'no changelog found' status of the repo means version provenance must be tracked manually

## Evidence

- [intro] Crits-Cristoph et al. used MIBiG to assess and highlight the exceptional novelty of BGCs: "Crits-Cristoph et al. (7) recently used MIBiG to assess and highlight the exceptional novelty of BGCs"
- [intro] Bahram et al. used homology searches against MIBiG to identify fungal BGCs associated with antibacterial activity: "Bahram et al. (8) used homology searches against MIBiG to identify fungal BGCs associated with antibacterial activity"
- [intro] standardized deposition and extraction of BGCs associated with molecules of known chemical structure: "This calls for a standardized deposition and extraction of BGCs associated with molecules of known chemical structure"
- [intro] MIBiG repository became a central reference database for BGCs of known function: "the MIBiG repository has become a central reference database for BGCs of known function"
- [readme] The WebAPI server driving the MIBiG repository: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene
cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
