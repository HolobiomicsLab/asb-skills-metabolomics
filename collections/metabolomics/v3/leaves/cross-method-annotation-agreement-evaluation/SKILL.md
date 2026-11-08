---
name: cross-method-annotation-agreement-evaluation
description: Use when an untargeted metabolomics dataset has both a computational
  metabolite prediction (e.g., mummichog on a significant m/z feature list) and an
  independent targeted annotation, and you need to measure their agreement to validate
  the prediction approach.
when_to_use_negative:
- The input is already an annotated feature table with confirmed metabolite identities
  — no computational prediction needs benchmarking
- Only one annotation method exists (no independent targeted annotation or standard-based
  identification to compare against)
- The goal is de novo metabolite identification rather than benchmarking a prediction
  method — mummichog 'is not designed to replace tandem mass spectrometry in metabolite
  identification'
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: mummichog
  role: Produces the computationally predicted metabolite network/activity from the
    significant m/z feature list using a genome-scale metabolic model
  repo: https://github.com/shuzhao-li/mummichog
- name: LC/MS
  role: Data acquisition platform whose spectral feature tables are the input to both
    the prediction and the targeted annotation
- name: metDataModel
  role: Data-model structure used to load the BioCyc yeast metabolic model (with compound
    neutral formulas) into mummichog
- name: Azimuth DB
  role: Chemical database supplying the metabolic models referenced by the prediction
    pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:7b8409a8e419642d6a3ba349715df2c044af8c7b0a5c290dd0139082387dbd39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/mummichog
  tools:
  - Azimuth DB
  - LC/MS
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-method-annotation-agreement-evaluation

## Summary

Quantify the overlap between metabolites predicted computationally by mummichog from an untargeted feature table and those identified by an independent targeted annotation method (e.g., MAVEN with authentic standards), to benchmark the accuracy of network-based prediction without a priori metabolite identification.

## When to use

Use when an untargeted metabolomics dataset has both a computational metabolite prediction (e.g., mummichog on a significant m/z feature list) and an independent targeted annotation, and you need to measure their agreement to validate the prediction approach.

## When NOT to use

- The input is already an annotated feature table with confirmed metabolite identities — no computational prediction needs benchmarking
- Only one annotation method exists (no independent targeted annotation or standard-based identification to compare against)
- The goal is de novo metabolite identification rather than benchmarking a prediction method — mummichog 'is not designed to replace tandem mass spectrometry in metabolite identification'

## Inputs

- Predicted metabolite list from computational network analysis (e.g., mummichog output on a significant m/z feature list)
- Targeted metabolite annotation table (e.g., MAVEN annotation of 101 metabolites with authentic standards)
- Significant feature list selected by univariate t-test (e.g., 426 features from 5,707 total spectral features)
- Metabolic model with compound neutral formulas (e.g., BioCyc yeast model via metDataModel/Azimuth DB)

## Outputs

- Set of comparable metabolites between the two methods (e.g., 28 metabolites)
- Agreement count and fraction (e.g., 24/28 ≈ 0.857)
- List of predicted metabolites not covered by the targeted annotation
- Sensitivity check of agreement with vs. without enforced adduct form (M+H[+] or M−H[−])

## How to apply

Run the targeted annotation first (e.g., process deposited mzXML files with MAVEN for peak picking, quantification and alignment, then targeted annotation against known standards — in the yeast riboneogenesis study this yielded 101 annotated metabolites). Separately, run the computational prediction (e.g., mummichog with the BioCyc yeast metabolic model carrying compound neutral formulas, enforcing the M−H[−] adduct for negative ion mode) on the significant feature list selected by univariate t-test (426 of 5,707 features in the yeast case). Restrict the comparison to metabolites comparable between both methods (28 in the yeast case), then compute the agreement fraction of matched metabolite identities (24/28 ≈ 86%). Repeat the prediction with and without adduct enforcement to test sensitivity of the agreement to that option — in the yeast data enforcing M−H[−] had little effect. Treat the agreement fraction as the primary validation metric; note that the network approach may predict metabolites absent from the targeted annotation (in the moDC data, 19 of 60 predicted metabolites were not in the original annotation), so restrict agreement computation to the comparable subset.

## Related tools

- **mummichog** (Produces the computationally predicted metabolite network/activity from the significant m/z feature list using a genome-scale metabolic model) — https://github.com/shuzhao-li/mummichog
- **LC/MS** (Data acquisition platform whose spectral feature tables are the input to both the prediction and the targeted annotation)
- **metDataModel** (Data-model structure used to load the BioCyc yeast metabolic model (with compound neutral formulas) into mummichog)
- **Azimuth DB** (Chemical database supplying the metabolic models referenced by the prediction pipeline)

## Examples

```
python3 -m mummichog.main -i yeast_significant_426.tsv -j yeast_run -a empCpds_with_annotations.json -d .
```

## Evaluation signals

- The comparable-metabolite subset is well defined (e.g., 28 metabolites overlapping between prediction and targeted annotation of 101 metabolites)
- Agreement fraction matches the expected benchmark (24/28 ≈ 0.857 for the yeast cell-extract data)
- Agreement is stable with and without enforcing the adduct form (enforcing M−H[−] had little effect in the yeast data)
- Result tables and figures are written to result.html and JSON pathway/network outputs are parseable
- Predicted metabolites absent from the targeted annotation (e.g., 19 of 60 in the moDC data) are reported separately rather than counted as disagreements

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Mummichog is not designed to replace tandem mass spectrometry in metabolite identification
- Features from high-resolution profiling experiments far exceed current annotations in metabolite databases, causing de facto filtering
- Quantification and feature selection, the steps upstream of mummichog, can still be problematic
- The computationally predicted network can include metabolites not found in the original targeted annotation, so only the comparable subset measures agreement

## Evidence

- [supplementary] Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation.: "Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation."
- [discussion] Mummichog is not designed to replace tandem MS in metabolite identification.: "Mummichog is not designed to replace tandem mass spectrometry in metabolite identification."
- [results] There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction.: "There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction."
- [readme] Outpout are 1. Result tables and figures, result.html as ver 2. 2. JSON strings from pathway analysis and network module analysis for programmatic use.: "Outpout are 
1. Result tables and figures, result.html as ver 2.
2. JSON strings from pathway analysis and network module analysis for programmatic use."
- [discussion] This leads to a de facto filtering when data are run on mummichog (similiar situation in database searches).: "This leads to a de facto filtering when data are run on mummichog (similiar situation in database searches)."
