---
name: metabolic-network-module-activity-analysis
description: Reference for reproducing mummichog network module activity predictions
  (e.g. the moDC YF17D activation network of 60 metabolites, 19 un-annotated) from
  a significant-m/z list plus a full reference feature list.
when_to_use_negative:
- Input is a single m/z feature needing definitive identification — mummichog is not
  designed to replace tandem mass spectrometry in metabolite identification
- Serum or urine samples where mass balance of reactions producing the metabolites
  is not expected within the sample itself
- The metabolites of interest are not present in any of the metabolic models (many
  metabolomics database entries are not in any model)
edam_operation: http://edamontology.org/operation_3722
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3172
tools:
- name: mummichog
  role: Core algorithm package for pathway/network module activity analysis on feature
    tables
  repo: http://atcg.googlecode.com
- name: metDataModel
  role: Data models for metabolomics used to structure annotation tables consumed
    by mummichog
- name: Azimuth DB
  role: Chemical database for biology including metabolic models feeding mummichog
- name: JMS
  role: Performs annotation on a dataset to supply the optional feature-annotation
    table
  repo: https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:c5f8fe2b800997c5c469757a50b860d222a2d97d45cd70adb18a247128b25b68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - JMS
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3722
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolic-network-module-activity-analysis

## Summary

Module-level activity analysis in mummichog: rather than testing whole pathways, it extracts subnetworks of one to four reactions from a genome-scale metabolic model, identifies structural modules via Newman's spectral split algorithm, and computes module activity p-values against a Gamma-modeled permutation null — predicting functional activity directly from spectral feature tables without metabolite identification.

## When to use

Reference for reproducing mummichog network module activity predictions (e.g. the moDC YF17D activation network of 60 metabolites, 19 un-annotated) from a significant-m/z list plus a full reference feature list.

## When NOT to use

- Input is a single m/z feature needing definitive identification — mummichog is not designed to replace tandem mass spectrometry in metabolite identification
- Serum or urine samples where mass balance of reactions producing the metabolites is not expected within the sample itself
- The metabolites of interest are not present in any of the metabolic models (many metabolomics database entries are not in any model)

## Inputs

- Significant list of m/z features (e.g. selected by univariate statistics; tsv with m/z, rtime, p-value from a statistical test)
- Reference list of all spectral features detected in the MS experiment
- Optional annotation table on the features (metDataModel format)
- Optional metabolic model JSON (default human model based on KEGG, UCSD Recon1 and Edinburgh human metabolic network; BioCyc alternative for yeast/urine)

## Outputs

- Activity network with predicted metabolites colored by log2 fold change and connecting enzymes shown as EC numbers (result.html tables and figures)
- JSON strings from pathway analysis and network module analysis for programmatic use
- Predicted metabolite set with confidence ratings (e.g. a network of 60 metabolites, 19 not found in the original annotation)
- Per-module p-values against the Gamma-modeled permutation null

## How to apply

Run mummichog with two inputs: the significant m/z list (e.g. the 601 features differentiating YF17D-infected vs mock moDC samples) and the reference list of all detected features (7,995 spectral features after QC). The algorithm computes all possibly matched metabolites from the m/z features using adduct/isotope-aware matching, searches the genome-scale human metabolic model (KEGG, UCSD Recon1, Edinburgh network) for subnetworks within one to four reactions, identifies structural modules via Newman's spectral split algorithm, scores activity, and estimates null module-activity distributions by random permutations of m/z features (modeled as a Gamma distribution). Clean each candidate module graph of protruding edges that do not connect input metabolites since they do not contribute to the activity score. Collect predicted metabolites from significant activities, look up their isotopic derivatives and adducts, and filter with the confidence rating system — e.g. co-occurrence of the M+H[1+] and M(C13)+H[1+] forms confers high confidence — carrying over connections from the reference network to build the activity network. Rationale: the collective power of the metabolic network resolves the ambiguity of a single m/z feature matching multiple metabolites.

## Related tools

- **mummichog** (Core algorithm package for pathway/network module activity analysis on feature tables) — http://atcg.googlecode.com
- **metDataModel** (Data models for metabolomics used to structure annotation tables consumed by mummichog)
- **Azimuth DB** (Chemical database for biology including metabolic models feeding mummichog)
- **JMS** (Performs annotation on a dataset to supply the optional feature-annotation table) — https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks

## Examples

```
python3 -m mummichog.main -f mummichog/tests/testdata0710.txt -o t3
```

## Evaluation signals

- Compare per-experiment activity network against the reference prediction: mummichog predicts a network of 60 metabolites on the moDC data, of which 19 are not in the original annotation
- Confirm the arginine/citrulline subnetwork shows significant activity consistent with activation of the nitric oxide synthase pathway
- Verify the predicted metabolite sets remain largely consistent when the feature-selection p-value cutoff is swept over the range in Figure S7
- Check that output result.html contains tables and figures plus JSON strings from pathway analysis and network module analysis
- Validate that input table contains 7,995 reference features with 601 significant features for the moDC experiment

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Quantification and feature selection, the steps upstream of mummichog, can still be problematic
- Current mummichog version does not incorporate a statistical matrix from the feature selection step nor mass flow balance of metabolic reactions
- Features from high-resolution profiling experiments far exceed the current annotations in metabolite databases, causing de facto filtering
- Metabolic models need improvement on lipid annotation, physiological context and tissue specificity

## Evidence

- [results] Module analysis identifies network activity beyond pathway annotation: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites."
- [results] Subnetwork size and module detection algorithm: "Structural modules are identified within these subnetworks according to Newman's spectral split algorithm [43]."
- [results] Confidence rating filters qualified metabolites: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [supplementary] Validation target on moDC data: "Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation."
- [abstract] mummichog's design goal and rationale: "We present a set of computational algorithms which, by leveraging the collective power of metabolic pathways and networks, predict functional activity directly from spectral feature tables without a"
