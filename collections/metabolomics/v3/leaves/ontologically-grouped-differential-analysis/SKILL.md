---
name: ontologically-grouped-differential-analysis
description: Use when a meta-analysis goal requires comparing broad biological categories
  (e.
when_to_use_negative:
- The comparison is a single, well-defined pairwise contrast between two concrete
  sample groups with no need for ontology-based term expansion
- Metadata combinations have few samples or few studies — the article notes quantitative
  comparisons are less robust for such combinations
- The dataset is not the curated BinBase snapshot (December 2021, 156,174 samples)
  or lacks ontology-mapped metadata; BinDiscover cannot incorporate later-added compounds
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: BinDiscover
  role: Public webtool that stores the BinBase-derived intensity dataset in PostgreSQL,
    expands ontology query terms into metadata combinations, and executes OGDA queries
- name: SciPy
  role: Statistics routines used to compute the pairwise Welch t-tests on log-transformed
    intensity distributions
- name: NetworkX
  role: Network analysis framework used to build and traverse the 'belongs to' ontology
    graph for term expansion
- name: Python
  role: Custom scripts implementing the OGDA aggregation (max p-value, min fold-change)
    and graph expansion
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:ce13bedfad44141f4c2aba55c99e689bf354cf346c19189662551d85374517cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinBase
  - BinDiscover
  - NetworkX
  - Python
  - SciPy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# ontologically-grouped-differential-analysis

## Summary

Ontologically grouped differential analysis (OGDA) as implemented in BinDiscover: a broad biological question (e.g., Human Digestive Tract vs. Bacteria) is expanded via metadata ontologies (MeSH, NCBI taxonomy) into sets of concrete metadata triads, pairwise Welch t-tests are run per compound, and results are conservatively aggregated into a single p-value and fold-change per metabolite.

## When to use

Use when a meta-analysis goal requires comparing broad biological categories (e.g., 'Digestive Tract', 'Bacteria', 'rodents', 'amino acids') across GC–MS metabolomics samples whose metadata are organized as organ/species/disease triads mapped to ontologies, rather than simple pairwise sample-group contrasts.

## When NOT to use

- The comparison is a single, well-defined pairwise contrast between two concrete sample groups with no need for ontology-based term expansion
- Metadata combinations have few samples or few studies — the article notes quantitative comparisons are less robust for such combinations
- The dataset is not the curated BinBase snapshot (December 2021, 156,174 samples) or lacks ontology-mapped metadata; BinDiscover cannot incorporate later-added compounds

## Inputs

- Metadata-annotated metabolite intensity matrix from BinBase GC/TOF MS samples, with each sample keyed by an organ/species/disease metadata triad
- Ontology hierarchies mapping metadata triads to terms (MeSH for organs/diseases, NCBI taxonomy for species)
- Two ontology triads defining the query groups (e.g., (Human, Digestive Tract, no Disease) vs. (Bacteria, Cells, no Disease))

## Outputs

- Aggregated per-compound p-value and fold-change table (one row per metabolite, e.g., ogda_results.csv) restricted to structurally identified compounds
- List of significant metabolites per comparison (e.g., 15 metabolites increased in human digestive system organs for the showcase query)

## How to apply

Load the relevant ontology hierarchy (MeSH for organs/diseases, NCBI taxonomy for species) and build a 'belongs to' graph so each generic query term resolves to all ontologically related metadata-combination nodes; exclude nodes mapping to different ontology branches (e.g., human stool maps to MeSH 'fluids and secretions', not 'Digestive System'). For each compound, run pairwise Welch t-tests on log-transformed intensity distributions for all n*m pairs of metadata combinations across the two query groups, using SciPy statistics routines and NetworkX for the ontology graph. Aggregate the per-compound n*m p-value and fold-change matrices with the paper's conservative rule: retain the maximum p-value and minimum fold-change, and set fold-change to 0 when significant pairs show both positive and negative fold-changes. Restrict the analysis to the 773 structurally identified BinBase metabolites (18,290 Bins from 156,174 samples), running only on metadata triads with at least 10 samples after QC filtering. The rationale is that aggregating many small ontology-grounded contrasts with the max-p/min-fold-change boundary avoids over-optimistic significance from any single heterogeneous pair.

## Related tools

- **BinDiscover** (Public webtool that stores the BinBase-derived intensity dataset in PostgreSQL, expands ontology query terms into metadata combinations, and executes OGDA queries)
- **SciPy** (Statistics routines used to compute the pairwise Welch t-tests on log-transformed intensity distributions)
- **NetworkX** (Network analysis framework used to build and traverse the 'belongs to' ontology graph for term expansion)
- **Python** (Custom scripts implementing the OGDA aggregation (max p-value, min fold-change) and graph expansion)

## Examples

```
from scipy.stats import ttest_ind; stat, p = ttest_ind(np.log(group_a_intensities), np.log(group_b_intensities), equal_var=False)  # Welch t-test per metadata-combination pair, per compound
```

## Evaluation signals

- The showcase query (Human Digestive Tract vs. Bacteria, no Disease on both sides) must yield exactly 15 significant metabolites reported at increased levels in human digestive system organs, matching Additional file 3: Table S2
- The aggregated output must contain exactly one p-value and one fold-change per compound, with p-values equal to the maximum over all n*m pair tests (conservative aggregation)
- Fold-change must be set to 0 for any compound whose significant pairs show both positive and negative fold changes
- Stool samples must be excluded from the 'Digestive Tract' expansion because they map to MeSH 'fluids and secretions', not digestive system
- Runtime is a sanity check: the full 773-compound showcase query completes in ~26 s (~1 s per metadata-combination query)

## Limitations

- BinDiscover is built on a static December 2021 snapshot of BinBase, so novel compounds added later cause metadata incongruencies
- Cross-organ comparisons may mix biofluids versus tissues (different units of biomass), and different solvent extraction efficiencies across tissues or biofluids may introduce bias
- For biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than for differential analyses
- Human stool maps to MeSH 'fluids and secretions' and is therefore excluded from digestive-tract OGDA queries, which may surprise users expecting stool inclusion

## Evidence

- [intro] OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs.: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [intro] we assigned all metadata into ontologies to empower broad comparisons such as phylo organizations or ontologically grouped differential analysis (OGDA): "we assigned all metadata into ontologies to empower broad comparisons such as phylo organizations or ontologically grouped differential analysis (OGDA)"
- [methods] We heavily employed statistics routines from SciPy and the network analysis framework from NetworkX: "We heavily employed statistics routines from SciPy and the network analysis framework from NetworkX"
- [abstract] we emphasize the use of ontologies to aggregate large groups of samples based on the proximity of their metadata within these ontologies.: "we emphasize the use of ontologies to aggregate large groups of samples based on the proximity of their metadata within these ontologies."
