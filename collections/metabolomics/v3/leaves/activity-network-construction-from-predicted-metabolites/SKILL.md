---
name: activity-network-construction-from-predicted-metabolites
description: Use when untargeted LC/MS metabolomics yields a significant m/z feature
  list (e.g. 601 features out of 7,995) and the goal is a predicted pathway/network
  activity, such as detecting nitric oxide synthase activation via arginine/citrulline
  shifts.
when_to_use_negative:
- The task is definitive metabolite identification — mummichog is not designed to
  replace tandem mass spectrometry for confirming metabolite identity.
- The input is already an identified, annotated metabolite abundance table rather
  than a raw m/z feature table.
- The sample matrix makes mass-flow balance impossible (e.g. serum or urine, where
  producing reactions occur in other tissues), so network activity claims relying
  on reaction mass balance are unwarranted.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: mummichog
  role: 'Core algorithm package: computes possibly matched metabolites from m/z features,
    performs module/pathway analysis, applies the confidence rating, and assembles
    the activity network'
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: cloud-mummichog
  role: Server and worker (RESTful) implementations for running the activity network
    construction remotely
- name: Azimuth DB
  role: Chemical database for biology supplying the metabolic models and adduct/isotope
    lookups used in network construction (planned rename to Mummichog DB)
- name: metDataModel
  role: Data models for metabolomics structuring the feature annotation table consumed
    by mummichog
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:dc23e716f3e3b5bf4f8b5b6bb7d8e00d0820a16177a25e3391228c33ce3276de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - cloud-mummichog
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# activity-network-construction-from-predicted-metabolites

## Summary

Constructs a metabolic activity network directly from a significant m/z feature list by matching features to candidate metabolites in a genome-scale human metabolic model, applying a confidence rating over adduct/isotope forms, and carrying over connections from the reference network — bypassing a priori metabolite identification.

## When to use

Use when untargeted LC/MS metabolomics yields a significant m/z feature list (e.g. 601 features out of 7,995) and the goal is a predicted pathway/network activity, such as detecting nitric oxide synthase activation via arginine/citrulline shifts.

## When NOT to use

- The task is definitive metabolite identification — mummichog is not designed to replace tandem mass spectrometry for confirming metabolite identity.
- The input is already an identified, annotated metabolite abundance table rather than a raw m/z feature table.
- The sample matrix makes mass-flow balance impossible (e.g. serum or urine, where producing reactions occur in other tissues), so network activity claims relying on reaction mass balance are unwarranted.

## Inputs

- Significant m/z feature list (e.g. 601 significant features from the moDC YF17D vs mock LC/MS experiment)
- Reference list of all detected spectral features (7,995 features after QC in the moDC experiment)
- Genome-scale metabolic model (KEGG / UCSD Recon1 / Edinburgh human metabolic network; alternative BioCyc human model), with compound neutral formulas and adduct/isotope mappings
- Optional annotation table on the features (from authentic standards and MS/MS), structured with metDataModel

## Outputs

- Predicted activity network of metabolites (e.g. 60 metabolites, 19 not in the original annotation), colored by log2 fold change with connecting enzymes as EC numbers
- Result tables and figures (result.html)
- JSON strings from pathway analysis and network module analysis
- Per-pathway enriched p-values with EASE score and adjusted p-values; module activity scores with Gamma-modeled permutation p-values

## How to apply

Run mummichog with two m/z feature lists: a significant list (e.g. the 601 features from the YF17D-activated moDC experiment selected by univariate statistics) and a reference list of all detected features (7,995). The core algorithm computes all possibly matched metabolites from m/z features using adduct/isotope-aware matching against a genome-scale human metabolic model built on KEGG, UCSD Recon1 and the Edinburgh human metabolic network. After module analysis (subnetworks of one to four reactions split via Newman's spectral algorithm, p-values against a Gamma-modeled null from random permutation of m/z features), collect the metabolites predicted from significant activities, look up all their isotopic derivatives and adducts, and filter through the confidence rating system — e.g. co-occurrence of the M+H[1+] and M(C13)+H[1+] forms confers high confidence; optionally enforce the M+H[+] (positive mode) or M−H[−] (negative mode) form. Finally, carry over connections from the reference network to the retained metabolites to form the activity network, and report it via result.html tables/figures plus JSON strings from pathway and network-module analysis. For the moDC YF17D case, the expected result is a network of 60 metabolites (19 absent from the original annotation) in which decreased arginine and increased citrulline indicate nitric oxide synthase pathway activation.

## Related tools

- **mummichog** (Core algorithm package: computes possibly matched metabolites from m/z features, performs module/pathway analysis, applies the confidence rating, and assembles the activity network) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **cloud-mummichog** (Server and worker (RESTful) implementations for running the activity network construction remotely)
- **Azimuth DB** (Chemical database for biology supplying the metabolic models and adduct/isotope lookups used in network construction (planned rename to Mummichog DB))
- **metDataModel** (Data models for metabolomics structuring the feature annotation table consumed by mummichog)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Network size and novelty match validation: for the moDC YF17D run, mummichog should predict a network of 60 metabolites of which 19 are not found in the original annotation (compare against Dataset S1).
- Biological consistency check: the network should show decreased arginine and increased citrulline consistent with nitric oxide synthase pathway activation, and the glutathione/GSSG subnetwork should show significant activity.
- Confidence rating invariant: retained metabolites should be qualified by the rating system, with high confidence assigned only when both M+H[1+] and M(C13)+H[1+] forms are present (or the enforced single-charge form in the chosen ionization mode).
- Structural integrity: every edge in the activity network must be carried over from the reference metabolic network, and candidate module graphs must be cleaned of protruding edges that do not connect input metabolites.
- Output format check: run must produce result.html tables/figures plus JSON strings from both pathway analysis and network module analysis.

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number and biasing enrichment.
- Features from high-resolution profiling experiments far exceed current annotations in metabolite databases, and some metabolites in databases are absent from any metabolic model, causing de facto filtering of features.
- The current mummichog version does not incorporate a statistical matrix from the feature selection step nor mass flow balance of metabolic reactions; quantification and feature selection upstream can still be problematic.
- The metabolic models need improvement on lipid annotation, physiological context and tissue specificity; an alternative human model (BioCyc) yields only comparable, not identical, results.

## Evidence

- [results] confidence rating filters qualified metabolites: "Mummichog collects these metabolites, and look up all their isotopic derivatives and adducts in . A confidence rating system is applied to filter for qualified metabolites."
- [results] high-confidence adduct/isotope co-occurrence rule: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [results] reference network sources for connection carry-over: "The genome-scale human metabolic network in mummichog is based on KEGG [38], UCSD Recon1 [15] and Edinburgh human metabolic network [16]."
- [supplementary] validated network size and novel predictions: "Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation."
- [readme] tool purpose stated in README: "It leverages the organization of metabolic networks to predict functional activity directly from feature tables, bypassing metabolite identification."
