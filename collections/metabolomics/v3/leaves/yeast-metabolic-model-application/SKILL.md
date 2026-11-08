---
name: yeast-metabolic-model-application
description: Use when analyzing yeast LC/MS untargeted metabolomics data with a feature
  table (e.g. MAVEN-extracted features from mzXML) and you want pathway/network activity
  prediction without prior metabolite identification.
when_to_use_negative:
- The goal is definitive metabolite identification — mummichog predicts network activity
  and 'is not designed to replace tandem mass spectrometry in metabolite identification'
- The organism is not yeast (or no BioCyc yeast model applies) — use the appropriate
  species-specific metabolic model instead, since compound identifiers and pathway
  definitions must align with the supplied model
- Input is already a list of identified metabolites rather than an m/z feature table
  — targeted pathway over-representation methods are then more appropriate
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: mummichog
  role: Core algorithm that computes possibly matched metabolites from m/z features,
    performs pathway enrichment (FET with EASE, Gamma null) and network module analysis
    using the BioCyc yeast model
  repo: https://github.com/shuzhao-li/mummichog
- name: metDataModel
  role: Data model used to structure the BioCyc yeast metabolic model and annotation
    tables for loading into mummichog
- name: Azimuth DB
  role: Chemical database for biology, including metabolic models, supplying the yeast
    model with compound neutral formulas
- name: LC/MS
  role: Analytical platform producing the spectral features from yeast cell extracts
    processed into the feature table
- name: MAVEN
  role: Used for untargeted feature extraction, peak picking, quantification and alignment
    of yeast mzXML files, and for targeted annotation used as the comparison benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:5f1a8113a9d058e93083926859785cf3f5ba48ba889fb198a3ce587ff25407f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/mummichog
  tools:
  - Azimuth DB
  - LC/MS
  - MAVEN
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# yeast-metabolic-model-application

## Summary

Apply the BioCyc yeast metabolic model (with compound neutral formulas) inside mummichog via the metDataModel structure to predict metabolite and pathway activity from untargeted LC/MS feature tables, and benchmark predictions against targeted metabolite annotation.

## When to use

Use when analyzing yeast LC/MS untargeted metabolomics data with a feature table (e.g. MAVEN-extracted features from mzXML) and you want pathway/network activity prediction without prior metabolite identification.

## When NOT to use

- The goal is definitive metabolite identification — mummichog predicts network activity and 'is not designed to replace tandem mass spectrometry in metabolite identification'
- The organism is not yeast (or no BioCyc yeast model applies) — use the appropriate species-specific metabolic model instead, since compound identifiers and pathway definitions must align with the supplied model
- Input is already a list of identified metabolites rather than an m/z feature table — targeted pathway over-representation methods are then more appropriate

## Inputs

- Deposited yeast mzXML raw LC/MS files (e.g. from the riboneogenesis study)
- Feature table of m/z, retention time and p-values (5,707 features from MAVEN untargeted extraction)
- Significant feature list selected by univariate t-test (426 features between mutant and wild-type)
- Reference list of all detected m/z features
- BioCyc yeast metabolic model with compound neutral formulas, structured via metDataModel from Azimuth DB

## Outputs

- Predicted metabolite set from significant network activity (28 metabolites comparable to targeted annotation)
- Pathway enrichment results (Fisher's exact test with EASE correction, Gamma-modeled permutation null)
- Network module results with metabolites colored by log2 fold change and connecting EC numbers
- result.html report plus JSON strings from pathway and network module analysis for programmatic use

## How to apply

Process deposited yeast mzXML files with MAVEN for untargeted feature extraction, peak picking, quantification and alignment to obtain the full feature table (5,707 spectral features in the riboneogenesis study). Select significantly altered features between mutant and prototrophic wild-type groups by univariate t-test to yield the significant list (426 features) and keep all detected features as the reference list. Load the BioCyc yeast metabolic model — ensuring compound neutral formulas, not salt formats — into mummichog via the metDataModel structure from Azimuth DB. Run mummichog with the significant and reference lists, optionally enforcing the M−H[−] adduct form for negative ion mode (this had little effect on the yeast data). The tool performs Fisher's exact test pathway enrichment with EASE correction and a Gamma-modeled permutation null, plus network module analysis via Newman's spectral split algorithm, writing result.html and JSON outputs. Validate by comparing predicted metabolites against targeted annotation (expected 24/28 ≈ 86% agreement with MAVEN's targeted annotation of 101 metabolites).

## Related tools

- **mummichog** (Core algorithm that computes possibly matched metabolites from m/z features, performs pathway enrichment (FET with EASE, Gamma null) and network module analysis using the BioCyc yeast model) — https://github.com/shuzhao-li/mummichog
- **metDataModel** (Data model used to structure the BioCyc yeast metabolic model and annotation tables for loading into mummichog)
- **Azimuth DB** (Chemical database for biology, including metabolic models, supplying the yeast model with compound neutral formulas)
- **LC/MS** (Analytical platform producing the spectral features from yeast cell extracts processed into the feature table)
- **MAVEN** (Used for untargeted feature extraction, peak picking, quantification and alignment of yeast mzXML files, and for targeted annotation used as the comparison benchmark)

## Examples

```
python3 -m mummichog.main -i yeast_ttest_426.tsv -j yeast_biocyc -o yeast_results --model biocyc_yeast --mode negative --enforce M-H[-]
```

## Evaluation signals

- Predicted-vs-targeted metabolite agreement equals 24 of 28 comparable metabolites (≈0.857) against MAVEN's targeted annotation of 101 metabolites
- result.html exists and is a valid HTML report
- JSON pathway and network module outputs are parseable
- Feature table contains 5,707 spectral features and the t-test significant list contains 426 features
- Enforcing the M−H[−] adduct form in negative mode should have little effect on the yeast prediction outcome

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Quantification and feature selection upstream of mummichog can still be problematic
- Some metabolites in metabolomics databases are absent from metabolic models, and features from high-resolution profiling far exceed current database annotations, causing de facto filtering
- Compound formulas may be missing from a GSMM; if present they must be correct neutral masses and not salt formats, otherwise adduct calculation in mass spec is wrong

## Evidence

- [results] BioCyc yeast model adoption: "We adopted the yeast metabolic model from BioCyc database [11] for the yeast data [64]"
- [results] mummichog input requirement: "mummichog requires two lists of m/z features, the significant list (e.g. selected by univariate statistics) and the reference list (all features detected in the MS experiment)."
- [results] adduct enforcement option: "There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction."
- [results] pathway analysis method: "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
- [other] yeast validation finding: [evidence span withheld — anchor status: not_found]
- [readme] model formula requirements: "If compound formulas are provided in a model, be sure to have correct neutral mass and not a salt format. Salts are formed in both biological systems and in mass spectrometry."
- [readme] output formats: "Outpout are 1. Result tables and figures, result.html as ver 2. 2. JSON strings from pathway analysis and network module analysis for programmatic use."
