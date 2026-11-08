---
name: compound-class-level-querying
description: Use when the goal is broad meta-analysis across many BinBase GC–TOF MS
  studies where sample or compound metadata should be aggregated by ontology adjacency
  (OGDA), e.g. asking which metabolites differ between 'digestive tract' vs bacteria
  or across 'rodents'.
when_to_use_negative:
- The question concerns a single pre-named compound or taxon rather than a category
  that ontology adjacency can expand — a plain differential query without OGDA is
  sufficient.
- The data are not from the BinBase/GC–TOF MS snapshot (e.g. LC–MS or external repositories
  such as Metabolomics Workbench or MetaboLights), because BinDiscover relies on BinBase's
  standardized, ontology-mapped metadata and the December 2021 static snapshot.
- A quantitative concentration comparison is needed for a metadata combination with
  very few samples or few studies — the article warns such comparisons are less robust
  than differential analyses.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: BinDiscover
  role: Webtool and PostgreSQL-backed database that performs ontologically grouped
    differential analysis (OGDA) over the BinBase GC–TOF MS snapshot; accessed via
    its query interface to explore compound or biological categories.
- name: BinBase
  role: Source database of standardized GC/MS metabolomics data (156,000+ samples,
    2000+ studies) whose metadata are mapped to ontologies that enable class-level
    querying in BinDiscover.
- name: Python
  role: Custom python scripts used for downstream analyses such as submitting compound
    lists from OGDA hits to ClassyFire for chemical-class annotation.
  repo: https://github.com/ucdavis/BinDiscover
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:71b5e06d44e6457835d72513a10bc83732f2052ddd29acd5069632200b39a209
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucdavis/BinDiscover
  tools:
  - BinBase
  - BinDiscover
  - Python
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# compound-class-level-querying

## Summary

Queries the BinDiscover webtool at the level of compound ontologies (e.g. amino acids) or biological ontologies (e.g. 'digestive tract') so that entire categories of metabolites or organisms can be explored simultaneously instead of single named compounds or taxa.

## When to use

Use when the goal is broad meta-analysis across many BinBase GC–TOF MS studies where sample or compound metadata should be aggregated by ontology adjacency (OGDA), e.g. asking which metabolites differ between 'digestive tract' vs bacteria or across 'rodents'.

## When NOT to use

- The question concerns a single pre-named compound or taxon rather than a category that ontology adjacency can expand — a plain differential query without OGDA is sufficient.
- The data are not from the BinBase/GC–TOF MS snapshot (e.g. LC–MS or external repositories such as Metabolomics Workbench or MetaboLights), because BinDiscover relies on BinBase's standardized, ontology-mapped metadata and the December 2021 static snapshot.
- A quantitative concentration comparison is needed for a metadata combination with very few samples or few studies — the article warns such comparisons are less robust than differential analyses.

## Inputs

- BinBase snapshot metadata mapped to established ontologies (biological and compound ontologies)
- OGDA query specification naming ontology categories (e.g. 'digestive tract', 'rodents', 'amino acids')
- List of significant metabolites from an OGDA query result (e.g. Additional file 3: Table S2, 15 human-digestive-tract-vs-bacteria metabolites)

## Outputs

- Differential-analysis graphs combining statistical results across ontology-defined subcategories
- Lists of significant metabolites grouped by compound class (vitamins, lipids, sterols, amino acid derivatives)
- Ontology-adjacent sample-group comparisons (e.g. entire biological categories vs each other)

## How to apply

Take the static December 2021 snapshot of BinBase (156,174 samples, 18,290 Bins, 773 identified metabolites) loaded into the BinDiscover PostgreSQL database, and formulate the question as an OGDA query: metadata are pre-assigned to ontologies, so the query transforms a broad question into sets of smaller categories (e.g. digestive tract vs bacteria) and combines statistical outputs into graphs. Run the query through the BinDiscover Flask/Plotly-Dash interface or its API; BinDiscover uses bucket-sort matching of peaks to Bins and supports imputed values (percentage-of-presence × minimum detected value) so missing-at-random peaks do not block the comparison. Interpret results at the level of ontology adjacency: compounds are grouped into classes such as vitamins, lipids, sterols, or amino acid derivatives, and sample groups are aggregated by proximity in biological ontologies such as rodents or digestive tract. Judge success by whether the output covers a whole category (not a single compound/taxon) and by whether the significant hit list is consistent with the expected class composition for the queried biological context.

## Related tools

- **BinDiscover** (Webtool and PostgreSQL-backed database that performs ontologically grouped differential analysis (OGDA) over the BinBase GC–TOF MS snapshot; accessed via its query interface to explore compound or biological categories.)
- **BinBase** (Source database of standardized GC/MS metabolomics data (156,000+ samples, 2000+ studies) whose metadata are mapped to ontologies that enable class-level querying in BinDiscover.)
- **Python** (Custom python scripts used for downstream analyses such as submitting compound lists from OGDA hits to ClassyFire for chemical-class annotation.) — https://github.com/ucdavis/BinDiscover

## Examples

```
python scripts/classyfire_lookup.py --input ogda_hits.csv --output classyfire_classification.csv
```

## Evaluation signals

- The output covers an entire ontology category (e.g. all compounds in the amino-acid class or all samples in 'digestive tract'), not just individual named entries.
- For a compound-class query, every metabolite in the hit list carries a non-empty superclass/class assignment consistent with the summary categories (vitamins, lipids, sterols, amino acid derivatives).
- Sample groups in the result are aggregated by metadata proximity within the ontology (adjacent classes), as the OGDA transformation of the broad question expects.
- Bins appearing in results reflect the underlying data: 773 identified metabolites, 39 known chemical artifacts, and 15,843 unannotated spectra were processed into 18,290 Bins.

## Limitations

- BinDiscover is built on a static December 2021 snapshot of BinBase; compounds added after that date will cause metadata incongruencies.
- Quantitative comparisons are less robust for biological metadata combinations with few samples and few studies.
- Abundances can be underestimated when detector saturation occurs during coelution of high- and low-abundance compounds.
- Different solvent extraction efficiencies across tissues or biofluids may introduce bias, and cross-organ comparisons may mix biofluids versus tissues (different units of biomass).

## Evidence

- [abstract] OGDA at compound ontology level: "we emphasize the use of ontologies to aggregate large groups of samples based on the proximity of their metadata within these ontologies"
- [intro] OGDA transformation of broad questions: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs"
- [intro] Snapshot size and ontology assignment: "we assigned all metadata into ontologies to empower broad comparisons such as phylo organizations or ontologically grouped differential analysis (OGDA)"
- [methods] Imputation handling of missing peaks: "we imputed the percentage of presence, multiplied by the minimum value of detected peaks (bins) for each metadata combination"
- [methods] Bucket-sort matching to Bins: "GC-Binbase uses a bucket sort approach, where new peaks from the chromatographic runs of samples are either matched to previously annotated groupings"
- [results] Class-level composition of hits: "vitamins, lipids, sterols, and amino acid derivatives"
- [discussion] Static snapshot caveat: "BinDiscover is built on top of a snapshot of BinBase data as they were in December 2021"
- [discussion] Robustness caveat for small groups: "for biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses"
