---
name: d3-interactive-tree-visualization
description: Use when spectral search matches (per-scan _matches.tsv and per-domain
  _count_microbe.tsv) need to be turned into a lineage-consistent interactive taxonomic
  tree HTML for interpreting which microbes may produce the queried metabolite.
when_to_use_negative:
- The goal is to generate or tune the raw spectral matches themselves (cosine score,
  mz tolerance, minimum matching peaks) rather than visualize existing microbeMASST
  output
- The input is a domain-MASST output other than microbeMASST taxonomy (e.g. plantMASST
  lineages) without an appropriately scoped lineage table
- A static summary table of taxon match counts suffices and no interactive, downloadable
  HTML visualization is needed
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: microbeMASST
  role: Domain-specific MASST whose batch search outputs (_matches.tsv, _count_microbe.tsv)
    and NCBI lineage data feed the tree rendering
  repo: https://github.com/robinschmid/microbe_masst
- name: jobs.py
  role: Batch search driver that generates the interactive HTML tree files for each
    domain-specific MASST as part of its output series
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: GNPS_MASST
  role: Standalone web application codebase for one-spectrum-at-a-time search, hosting
    the interactive tree rendering context
  repo: https://github.com/mwang87/GNPS_MASST
- name: Python 3.10
  role: Required runtime environment for the microbeMASST batch and visualization
    workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:438ec11e87fce2c1569b682917fe442ff7040990fd829cfcbd59b58f90d55a3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - GNPS_MASST
  - Python 3.10
  - jobs.py
  - microbeMASST
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# d3-interactive-tree-visualization

## Summary

Renders taxonomic search results as an interactive collapsible HTML tree with D3, propagating per-taxon match counts and proportions from microbeMASST output so a queried metabolite's candidate microbial producers can be visually explored and downloaded.

## When to use

Use when spectral search matches (per-scan _matches.tsv and per-domain _count_microbe.tsv) need to be turned into a lineage-consistent interactive taxonomic tree HTML for interpreting which microbes may produce the queried metabolite.

## When NOT to use

- The goal is to generate or tune the raw spectral matches themselves (cosine score, mz tolerance, minimum matching peaks) rather than visualize existing microbeMASST output
- The input is a domain-MASST output other than microbeMASST taxonomy (e.g. plantMASST lineages) without an appropriately scoped lineage table
- A static summary table of taxon match counts suffices and no interactive, downloadable HTML visualization is needed

## Inputs

- Per-scan match results from microbeMASST batch search (_matches.tsv)
- Per-domain match counts (_count_microbe.tsv)
- NCBI taxonomy lineage table for microbeMASST taxa (8 kingdoms through 542 strains)
- Full NCBI lineage JSON/Newick tree for microbeMASST taxa

## Outputs

- Interactive HTML taxonomic tree (e.g. _microbe.html) with collapsible nodes, per-taxon match-proportion pie charts, and hover/click tooltips
- D3-consumable JSON tree hierarchy with per-node match counts and proportions (e.g. _microbe.json)

## How to apply

Load the per-scan match results (_matches.tsv) and the per-domain match counts (_count_microbe.tsv) produced by the microbeMASST batch search, together with the NCBI taxonomy lineage table covering 8 kingdoms, 20 phyla, 48 classes, 124 orders, 278 families, 561 genera, 1379 species, and 542 strains. Aggregate matches per taxon at every lineage rank to compute the number and proportion of matched spectra or samples attributable to each taxon, then filter the full lineage JSON/Newick tree to taxa with at least one match, propagating counts upward through each rank. Convert the filtered taxonomy into a D3-consumable JSON hierarchy and render it in HTML with collapsible nodes, per-taxon match-proportion pie charts, and hover/click tooltips. The resulting file mirrors the interactive tree outputs microbeMASST generates for each domain-specific MASST (files ending with _domain.html, e.g. _microbe.html), and validation requires per-taxon counts to reconcile with the _count_microbe.tsv totals. Python 3.10 is the required environment for the surrounding microbeMASST workflow.

## Related tools

- **microbeMASST** (Domain-specific MASST whose batch search outputs (_matches.tsv, _count_microbe.tsv) and NCBI lineage data feed the tree rendering) — https://github.com/robinschmid/microbe_masst
- **jobs.py** (Batch search driver that generates the interactive HTML tree files for each domain-specific MASST as part of its output series) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **GNPS_MASST** (Standalone web application codebase for one-spectrum-at-a-time search, hosting the interactive tree rendering context) — https://github.com/mwang87/GNPS_MASST
- **Python 3.10** (Required runtime environment for the microbeMASST batch and visualization workflow)

## Examples

```
python code/jobs.py  # generates _microbe.html, _microbe.json, _count_microbe.tsv via Fast Search API
```

## Evaluation signals

- The rendered HTML must contain a complete lineage-consistent tree whose per-taxon counts reconcile with the input _count_microbe.tsv totals at every rank
- File existence and non-empty, renderable HTML content must be verified for the output _microbe.html
- Counts propagated up the lineage (kingdom through strain) must be consistent, i.e. child totals must not exceed parent totals
- Each taxon node must display taxon name, NCBI ID, file counts, and match proportion as a pie chart, matching the microbeMASST interactive tree spec
- The JSON hierarchy produced for D3 must be parseable and structurally consistent with the filtered tree

## Limitations

- The Fast Search API can cause some entries to fail, so jobs.py must be re-run a couple of times with skip_existing=True until no new output is generated; incomplete search results will yield incomplete trees
- _matches.tsv includes samples not part of the curated domain-specific MASSTs, so tree counts can reflect non-domain matches unless reconciled against _count_microbe.tsv
- Only Python 3.10 is supported for the workflow

## Evidence

- [readme] HTML tree outputs: "A series of interactive HTML trees files will be generated for each domain-specific MASST ending with _domain.html (e.g., _microbe.html)"
- [readme] per-domain count outputs: "A series of _count_domain.tsv files will be generated, containing information on matches found for each specific domain MASST."
- [readme] taxonomy coverage: "microbeMASST | 8 | 20 | 48 | 124 | 278 | 561 | 1379 | 542"
- [readme] lineage data location: "Within the folder lineages you can find the complete lineage information of each NCBI taxonomy IDs used in microbeMASST and plantMASST."
- [readme] rerun requirement: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`."
- [readme] Python version constraint: "Please make user to use **_Python 3.10_**"
