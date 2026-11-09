---
name: predicted-metabolite-set-stability-assessment
description: Use when you have mummichog's two input lists (significant m/z features
  selected by a cutoff and the full reference feature list) and need to know whether
  the predicted activity network is robust to the cutoff choice, e.g. for sensitivity
  analysis of the ~60-metabolite moDC prediction.
when_to_use_negative:
- You want to validate metabolite identity rather than prediction stability — mummichog
  is explicitly not designed to replace tandem mass spectrometry for identification
- The feature-selection cutoff is fixed by the experimental design and there is no
  need to test its sensitivity; a single mummichog run suffices
- Input is not an untargeted LC/MS feature table (e.g. targeted assays or pre-annotated
  compounds), where mummichog's pathway/module approach on unassigned m/z features
  does not apply
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
tools:
- name: mummichog
  role: Runs the pathway (Fisher exact test with EASE-score adjustment, Gamma-modeled
    permutation null) and network module (Newman spectral split on 1–4 reaction subnetworks)
    analysis that is rerun at each cutoff value
  repo: http://atcg.googlecode.com
- name: JMS
  role: Provides annotation data structures (empirical compounds with MS1 pseudo-spectra
    and ion_relation entries like M+H[1+]) used to structure optional feature annotation
    before/after the pipeline
  repo: https://github.com/shuzhao-li-lab/JMS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:dda0055c7a8b7fb0492b353ddae272158b15ae7d56d21639bd97981c51799a2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JMS
  - mummichog
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# predicted-metabolite-set-stability-assessment

## Summary

Sweeps the m/z feature-selection p-value cutoff across the range shown in Figure S7, rerunning mummichog's pathway and network module pipeline at each cutoff to quantify how stable the predicted metabolite set is on a given LC/MS dataset.

## When to use

Use when you have mummichog's two input lists (significant m/z features selected by a cutoff and the full reference feature list) and need to know whether the predicted activity network is robust to the cutoff choice, e.g. for sensitivity analysis of the ~60-metabolite moDC prediction.

## When NOT to use

- You want to validate metabolite identity rather than prediction stability — mummichog is explicitly not designed to replace tandem mass spectrometry for identification
- The feature-selection cutoff is fixed by the experimental design and there is no need to test its sensitivity; a single mummichog run suffices
- Input is not an untargeted LC/MS feature table (e.g. targeted assays or pre-annotated compounds), where mummichog's pathway/module approach on unassigned m/z features does not apply

## Inputs

- LC/MS feature table with m/z, rtime, and p-value columns (e.g. TSV accepted by mummichog; 7,995 spectral features in the moDC dataset)
- Significant-feature list selected by a univariate-statistics cutoff (601 features at the paper's primary cutoff)
- Reference feature list of all detected features in the MS experiment
- Range of p-value cutoffs to sweep (the range shown in Figure S7)
- Human genome-scale metabolic model built from KEGG, UCSD Recon1 and the Edinburgh human metabolic network (default in mummichog)

## Outputs

- Per-cutoff JSON strings from pathway analysis and network module analysis
- Per-cutoff result tables and figures (result.html as in mummichog v2)
- Predicted metabolite sets at each cutoff, with isotopic derivatives, adducts and confidence ratings
- Summary table of pairwise predicted-set overlap across all cutoffs, bracketing the reported ~60-metabolite network

## How to apply

Load the LC/MS feature table (e.g. the 7,995-feature moDC table from Dataset S1, of which 601 are significant at the primary cutoff) and split it into a significant list and a reference list as mummichog requires. Run the full pipeline at the paper's primary cutoff: pathway analysis via Fisher's exact test with EASE-score adjustment and a Gamma-modeled permutation null, plus network module analysis that extracts one-to-four reaction subnetworks, applies Newman's spectral split, and cleans protruding edges. Then sweep the cutoff across the range shown in Figure S7, rerunning the whole pathway/module pipeline at each value and applying the same confidence rating to predicted metabolites (e.g. co-occurrence of M+H[1+] and M(C13)+H[1+] forms gives high confidence). Compare predicted metabolite sets across cutoffs by pairwise overlap size, verifying that the network stays near the reported ~60 metabolites (19 of which were not in the original annotation).

## Related tools

- **mummichog** (Runs the pathway (Fisher exact test with EASE-score adjustment, Gamma-modeled permutation null) and network module (Newman spectral split on 1–4 reaction subnetworks) analysis that is rerun at each cutoff value) — http://atcg.googlecode.com
- **JMS** (Provides annotation data structures (empirical compounds with MS1 pseudo-spectra and ion_relation entries like M+H[1+]) used to structure optional feature annotation before/after the pipeline) — https://github.com/shuzhao-li-lab/JMS

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Per-cutoff predicted metabolite counts should bracket the reported value of 60 metabolites on the moDC data (Figure S7)
- Pairwise overlap sizes between predicted sets across cutoffs should be largely consistent, reproducing the reported stability
- The summary table of cross-cutoff overlaps must exist and be well-formed
- At each cutoff, the EASE-score-adjusted pathway p-values and Gamma-modeled permutation nulls should be recomputed (not carried over), so pathway p-values differ between runs
- Confidence-rated metabolites (e.g. M+H[1+] plus M(C13)+H[1+] co-occurrence) should be re-filtered at each cutoff, not held fixed

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating overlap counts between cutoffs
- Quantification and feature selection, the steps upstream of mummichog, can still be problematic, so cutoff sensitivity may reflect upstream noise rather than mummichog instability
- Features far exceed current metabolite database annotations, producing a de facto filtering that varies with the cutoff
- The current mummichog version does not incorporate a statistical matrix from the feature selection step, so the sweep only tests binary significance lists

## Evidence

- [results] mummichog requires two lists of m/z features: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [results] The basic test for pathway enrichment here is Fisher's exact test (FET): "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
- [results] Structural modules are identified within these subnetworks according to Newman's spectral split algorithm: "Structural modules are identified within these subnetworks according to Newman's spectral split algorithm [43]."
- [results] Mummichog collects these metabolites, and look up all their isotopic derivatives and adducts in . A confidence rating system is applied to filter for qualified metabolites.: "Mummichog collects these metabolites, and look up all their isotopic derivatives and adducts in . A confidence rating system is applied to filter for qualified metabolites."
- [results] yielded 7,995 spectral features (denoted as ) after quality control. Among them, 601 features were significantly different: "yielded 7,995 spectral features (denoted as ) after quality control. Among them, 601 features were significantly different between the infected samples"
- [results] The genome-scale human metabolic network in mummichog is based on KEGG [38], UCSD Recon1 [15] and Edinburgh human metabolic network [16].: "The genome-scale human metabolic network in mummichog is based on KEGG [38], UCSD Recon1 [15] and Edinburgh human metabolic network [16]."
- [discussion] In the moDC data, we chose  by a cutoff value .: "In the moDC data, we chose  by a cutoff value ."
- [supplementary] Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation.: "Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation."
- [readme] Input:
1. User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID.: "Input:
1. User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."
- [readme] Outpout are 
1. Result tables and figures, result.html as ver 2.
2. JSON strings from pathway analysis and network module analysis for programmatic use.: "Outpout are 
1. Result tables and figures, result.html as ver 2.
2. JSON strings from pathway analysis and network module analysis for programmatic use."
