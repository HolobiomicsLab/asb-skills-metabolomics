---
name: taxonomic-lineage-aggregation
description: Use when microbeMASST batch-search outputs (_matches.tsv, _count_microbe.tsv)
  must be aggregated per NCBI lineage rank and rendered as an interactive D3 taxonomic
  tree (microbe_tree.html) showing which microbes may produce a queried metabolite.
when_to_use_negative:
- A rendered _microbe.html tree from the microbeMASST web app already exists — no
  need to re-derive the lineage aggregation.
- No batch search outputs exist yet (only raw .mgf spectra or USI lists) — run the
  jobs.py Fast Search API batch search first; aggregation cannot precede matching.
- The queried spectra match plant-, tissue-, or foodMASST domains rather than microbial
  taxa — use the corresponding domain lineage and _count_domain.tsv instead.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: microbeMASST
  role: Source of the JSON microbeMASST taxonomic tree, NCBI lineage data, and the
    reference _microbe.html interactive tree output to mirror
  repo: https://github.com/robinschmid/microbe_masst
- name: Python 3.10
  role: Runtime for the aggregation, lineage filtering, D3 JSON conversion, and HTML
    tree rendering scripts
- name: D3
  role: JavaScript library used to render the interactive collapsible taxonomic tree
    with pie-chart annotations in HTML
- name: MZmine
  role: Upstream tool that generates the single .mgf file of spectra submitted to
    the batch search whose matches feed the lineage aggregation
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:96b046c310a8f7071c36c254e3a4a475456bfb81086d86fd47b4af9dd9b909b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - D3
  - MZmine
  - Python 3.10
  - microbeMASST
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# taxonomic-lineage-aggregation

## Summary

Aggregates per-scan spectral search matches (_matches.tsv, _count_microbe.tsv) onto the NCBI lineage hierarchy of microbeMASST taxa (8 kingdoms through 542 strains) and renders the filtered taxonomy as an interactive D3 taxonomic tree in microbe_tree.html, mirroring the _microbe.html output of microbeMASST.

## When to use

Use when microbeMASST batch-search outputs (_matches.tsv, _count_microbe.tsv) must be aggregated per NCBI lineage rank and rendered as an interactive D3 taxonomic tree (microbe_tree.html) showing which microbes may produce a queried metabolite.

## When NOT to use

- A rendered _microbe.html tree from the microbeMASST web app already exists — no need to re-derive the lineage aggregation.
- No batch search outputs exist yet (only raw .mgf spectra or USI lists) — run the jobs.py Fast Search API batch search first; aggregation cannot precede matching.
- The queried spectra match plant-, tissue-, or foodMASST domains rather than microbial taxa — use the corresponding domain lineage and _count_domain.tsv instead.

## Inputs

- _matches.tsv — per-scan spectral matches (all scans matching the searched spectrum)
- _count_microbe.tsv — aggregated match counts for the microbeMASST domain
- microbeMASST NCBI taxonomy lineage table (taxon IDs plus kingdom/…/strain lineage)
- microbeMASST full taxonomy tree (Newick/JSON)

## Outputs

- microbe_tree.html — interactive D3 taxonomic tree with collapsible nodes, per-taxon match-proportion pie charts, and tooltips
- Filtered D3 JSON hierarchy with per-node lineage-propagated match counts and proportions

## How to apply

Load the per-scan results (_matches.tsv) and per-domain match counts (_count_microbe.tsv) from the jobs.py batch search together with the microbeMASST NCBI lineage table (8 kingdoms, 20 phyla, 48 classes, 124 orders, 278 families, 561 genera, 1379 species, 542 strains). Aggregate matches per taxon at every rank, then filter the full lineage tree to keep only taxa with at least one match, propagating counts upward through kingdom→phylum→class→order→family→genus→species→strain so parent nodes reflect all descendant matches. Convert the filtered hierarchy into a D3-consumable JSON, attaching per-node taxon name, NCBI ID, file counts, and match-proportion values that drive pie-chart annotations. Render as collapsible HTML nodes with hover/click tooltips and write microbe_tree.html. Because _matches.tsv also covers samples outside the curated domain MASSTs, only propagate counts for taxa present in the curated microbeMASST lineage set.

## Related tools

- **microbeMASST** (Source of the JSON microbeMASST taxonomic tree, NCBI lineage data, and the reference _microbe.html interactive tree output to mirror) — https://github.com/robinschmid/microbe_masst
- **Python 3.10** (Runtime for the aggregation, lineage filtering, D3 JSON conversion, and HTML tree rendering scripts)
- **D3** (JavaScript library used to render the interactive collapsible taxonomic tree with pie-chart annotations in HTML)
- **MZmine** (Upstream tool that generates the single .mgf file of spectra submitted to the batch search whose matches feed the lineage aggregation) — https://github.com/mzmine/mzmine

## Evaluation signals

- Per-taxon counts in microbe_tree.html reconcile with the totals in _count_microbe.tsv at every lineage rank (no count created or dropped during aggregation).
- microbe_tree.html exists, is non-empty, and renders as a valid interactive HTML tree.
- Rendered tree is lineage-consistent: counts propagate correctly from strain/species up through genus, family, order, class, phylum, and kingdom nodes.
- Taxa with zero matches are excluded from the filtered tree; retained taxa carry taxon name, NCBI ID, file counts, and a match-proportion pie chart.
- Depth/coverage of the tree matches the microbeMASST lineage scope (8 kingdoms, 20 phyla, 48 classes, 124 orders, 278 families, 561 genera, 1379 species, 542 strains).

## Limitations

- Fast Search API entries can fail intermittently — jobs.py must be re-run several times with skip_existing=True until no new output is generated, otherwise incomplete _matches.tsv data yields an under-counted tree.
- _matches.tsv includes samples not part of the curated domain-specific MASSTs; unfiltered propagation would inflate taxon counts.
- The pipeline targets Python 3.10 specifically and produces only the microbial (microbeMASST) lineage view — plantMASST/tissueMASST/foodMASST require their own lineage tables and _count_domain.tsv files.

## Evidence

- [methods] A series of interactive HTML trees files will be generated for each domain-specific MASST ending with _domain.html (e.g., _microbe.html): "A series of interactive HTML trees files will be generated for each domain-specific MASST ending with _domain.html (e.g., _microbe.html)"
- [methods] microbeMASST | 8 | 20 | 48 | 124 | 278 | 561 | 1379 | 542: "microbeMASST | 8 | 20 | 48 | 124 | 278 | 561 | 1379 | 542"
- [readme] Within the folder lineages you can find the complete lineage information of each NCBI taxonomy IDs used in microbeMASST and plantMASST.: "Within the folder lineages you can find the complete lineage information of each NCBI taxonomy IDs used in microbeMASST and plantMASST."
- [readme] A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST.: "A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST."
- [readme] This includes also samples that are not part of the curated domain-specific MASSTs.: "This includes also samples that are not part of the curated domain-specific MASSTs."
