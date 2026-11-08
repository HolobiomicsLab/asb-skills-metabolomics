---
name: archaeplastida-class-coverage-computation
description: Use when you need family-level natural product census numbers — unique
  NPs, genera, species counts, and chemical-class coverage relative to Archaeplastida
  — from LOTUS, e.g. to benchmark literature-reported chemical space of a taxon.
when_to_use_negative:
- Your input is untargeted MS data (MZmine quantification tables, GNPS FBMN jobs)
  rather than a literature-reported structure–organism database — use Inventa's extract-prioritization
  workflow instead
- You need experimentally observed metabolites of specific extracts rather than literature-reported
  compounds per taxon (LOTUS counts are reported compounds, not MS features)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0621
tools:
- name: LOTUS
  role: Source database dump providing reported natural products with organism taxonomy
    and NPClassifier chemical ontology
  repo: https://lotus.naturalproducts.net/
- name: Open Tree of Life
  role: Cleaning taxon names to currently recognized species/genera before counting
  repo: https://opentree.readthedocs.io/en/latest/readme.html
- name: Inventa
  role: Prior proof-of-concept framework in which the Celastraceae chemotaxonomic
    context was established; the Literature Component uses reported compound counts
    per species/genus/family
  repo: https://github.com/luigiquiros/inventa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:d575589db21c1f735f546af0de60c70a02b1788a2974c7b74e8bf5e7decfc724
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Inventa
  - LOTUS
  - Open Tree of Life
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# archaeplastida-class-coverage-computation

## Summary

Computes a chemotaxonomic census for a plant family (e.g. Celastraceae) from the LOTUS database dump: counts of unique natural products (deduplicated by canonical SMILES/INCHI), genera, species, and NPClassifier chemical classes, plus the fraction of Archaeplastida-wide classes covered by the family.

## When to use

Use when you need family-level natural product census numbers — unique NPs, genera, species counts, and chemical-class coverage relative to Archaeplastida — from LOTUS, e.g. to benchmark literature-reported chemical space of a taxon.

## When NOT to use

- Your input is untargeted MS data (MZmine quantification tables, GNPS FBMN jobs) rather than a literature-reported structure–organism database — use Inventa's extract-prioritization workflow instead
- You need experimentally observed metabolites of specific extracts rather than literature-reported compounds per taxon (LOTUS counts are reported compounds, not MS features)

## Inputs

- LOTUS database dump (lotus.naturalproducts.net) with compound–organism pairs, taxonomy columns, and NPClassifier chemical ontology columns
- Taxon name reference for name cleaning (e.g. Open Tree of Life taxonomy)

## Outputs

- lotus_celastraceae_census.csv with unique NPs, genera, species, NPClassifier classes, and fraction of Archaeplastida classes
- Filtered compound–organism table restricted to the target family

## How to apply

Download the LOTUS database dump from lotus.naturalproducts.net, which reports natural products together with organism taxonomy and the NPClassifier chemical ontology. Load the dump with pandas and filter reported compound–organism pairs to entries whose family-level taxonomy equals the target family (e.g. Celastraceae). Count unique natural products using canonical structure identifiers (SMILES/INCHI) so that duplicate structure–organism records collapse into single compounds. Count distinct genera and species within the family, cleaning taxon names against currently recognized names via the Open Tree of Life, as recommended in the Inventa metadata workflow. Map each reported compound to its NPClassifier chemical class column, count distinct classes, and compute the fraction of Archaeplastida-wide classes covered by the family, then write the aggregated census to a CSV such as lotus_celastraceae_census.csv.

## Related tools

- **LOTUS** (Source database dump providing reported natural products with organism taxonomy and NPClassifier chemical ontology) — https://lotus.naturalproducts.net/
- **Open Tree of Life** (Cleaning taxon names to currently recognized species/genera before counting) — https://opentree.readthedocs.io/en/latest/readme.html
- **Inventa** (Prior proof-of-concept framework in which the Celastraceae chemotaxonomic context was established; the Literature Component uses reported compound counts per species/genus/family) — https://github.com/luigiquiros/inventa

## Examples

```
import pandas as pd; df = pd.read_csv('lotus_dump.tsv', sep='\t'); cel = df[df['organism_family'] == 'Celastraceae']; census = {'unique_nps': cel['structure_smiles'].nunique(), 'genera': cel['organism_genus'].nunique(), 'species': cel['organism_species'].nunique(), 'classes': cel['class_chemont'].nunique()}
```

## Evaluation signals

- Unique NP count for Celastraceae reproduces ~4,800 (0.98% of Archaeplastida entries) after canonical-SMILES/INCHI deduplication
- Species count reproduces 168 and genus count reproduces ~38 (any deviation beyond the 'approximately' qualifier on the genus count indicates an incorrect filter or identifier-deduplication step)
- NPClassifier class count reproduces 130 distinct classes (~20% of Archaeplastida classes)
- Filtered table contains only rows whose family-level taxonomy equals the target family (spot-check via Open Tree of Life-cleaned names)
- Output CSV schema contains the aggregate census fields (unique NPs, genera, species, classes, fraction) with non-negative, integer counts

## Limitations

- Counts are of literature-reported compounds only; genera count is approximate and sensitive to taxonomic name cleaning
- Results depend on the LOTUS dump version and on NPClassifier ontology columns; incorrect structure-identifier deduplication inflates unique-NP counts

## Evidence

- [results] LOTUS and DNP report 4,800 unique NPs for Celastraceae: [evidence span withheld — anchor status: not_found]
- [methods] LOTUS dump filter to family: [evidence span withheld — anchor status: not_found]
- [readme] Open Tree of Life for name cleaning: [evidence span withheld — anchor status: external_doc]
- [methods] LOTUS uses NPClassifier ontology: "given that the [Lotus Dabase](https://lotus.naturalproducts.net/) uses the NPClassifyre ontology"
