---
name: pathway-enrichment-analysis
description: Use when an untargeted LC/MS feature table (m/z, retention time, p-value)
  plus a significant-vs-reference feature split needs pathway-level interpretation,
  and metabolite identification is not yet available.
when_to_use_negative:
- The goal is definitive metabolite identification — 'Mummichog is not designed to
  replace tandem mass spectrometry in metabolite identification.'
- The analysis requires mass flow balance of metabolic reactions in samples like serum
  or urine, where producing reactions occur in other tissues and mass balance is infeasible.
- The study requires validated multivariate (in vivo) activity calls that mummichog's
  single-feature significant-list input cannot directly support.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0203
tools:
- name: mummichog
  role: core algorithm package performing m/z-to-metabolite matching, module finding,
    and Fisher's exact test pathway enrichment with EASE correction and Gamma-modeled
    permutation null
  repo: https://github.com/metabolomics-cloud/mummichog
- name: metDataModel
  role: data structure for loading the metabolic model and structuring feature/compound
    annotations used by mummichog
- name: Azimuth DB
  role: chemical database supplying the metabolic model (e.g. BioCyc yeast model with
    compound neutral formulas) in metDataModel format
- name: cloud-mummichog
  role: RESTful server and worker implementation for running the pathway analysis
    remotely
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:f9521393c3b51c2ec3046bb7fe1cf38b6053aba60770fbfd9cafedcb4187966a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-cloud/mummichog
  tools:
  - Azimuth DB
  - cloud-mummichog
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# pathway-enrichment-analysis

## Summary

Pathway and network module enrichment for untargeted LC/MS metabolomics using mummichog, which maps significant m/z features to tentative metabolites against a genome-scale metabolic model and tests pathway activity without prior metabolite identification.

## When to use

Use when an untargeted LC/MS feature table (m/z, retention time, p-value) plus a significant-vs-reference feature split needs pathway-level interpretation, and metabolite identification is not yet available.

## When NOT to use

- The goal is definitive metabolite identification — 'Mummichog is not designed to replace tandem mass spectrometry in metabolite identification.'
- The analysis requires mass flow balance of metabolic reactions in samples like serum or urine, where producing reactions occur in other tissues and mass balance is infeasible.
- The study requires validated multivariate (in vivo) activity calls that mummichog's single-feature significant-list input cannot directly support.

## Inputs

- significant list of m/z features (with p-values from a statistical test, e.g. t-test)
- reference list of all m/z features detected in the MS experiment
- genome-scale metabolic model (list_of_pathways, list_of_reactions, list_of_compounds with neutral formulas, e.g. BioCyc yeast model via metDataModel/Azimuth DB)
- optional annotation table on features (JSON, metDataModel-structured)

## Outputs

- result.html report with result tables and figures (metabolites colored by log2 fold change, EC-number annotations)
- JSON strings from pathway analysis (FET with EASE correction, Gamma-modeled permutation null)
- JSON strings from network module analysis
- predicted metabolite network with confidence-rated metabolites

## How to apply

Prepare two lists of m/z features: a significant list (e.g. the 426-feature list from univariate t-test on the yeast mutant-vs-wild-type data) and a reference list of all detected features (5,707 features in the yeast case). Load a genome-scale metabolic model — BioCyc for yeast, or the KEGG/UCSD Recon1/Edinburgh human network — via the metDataModel structure, ensuring compounds carry neutral formulas (not salt formats) so adducts like M+H[1+] and M−H[−] can be computed. Run the mummichog core algorithm (python3 -m mummichog.main), optionally enforcing the M−H[−] form in negative ion mode; the yeast validation showed this enforcement had little effect. Pathway significance is assessed with Fisher's exact test using the EASE score for robustness, with the permutation-based null p-value distribution modeled as Gamma; structural network modules are identified within 1–4 reaction subnetworks using Newman's spectral split algorithm, with protruding edges that do not connect input metabolites removed before scoring. Results are written as result.html tables/figures plus JSON strings for programmatic consumption.

## Related tools

- **mummichog** (core algorithm package performing m/z-to-metabolite matching, module finding, and Fisher's exact test pathway enrichment with EASE correction and Gamma-modeled permutation null) — https://github.com/metabolomics-cloud/mummichog
- **metDataModel** (data structure for loading the metabolic model and structuring feature/compound annotations used by mummichog)
- **Azimuth DB** (chemical database supplying the metabolic model (e.g. BioCyc yeast model with compound neutral formulas) in metDataModel format)
- **cloud-mummichog** (RESTful server and worker implementation for running the pathway analysis remotely)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- On the yeast riboneogenesis dataset (5,707 features, 426 significant via t-test), mummichog's predictions should agree with the original targeted annotation of 101 metabolites on 24 of 28 comparable metabolites (~86%), with little change when M−H[−] is enforced.
- result.html exists and is a valid HTML report containing pathway and network result tables/figures.
- The JSON pathway-analysis and network-module outputs are parseable and include per-pathway EASE-adjusted p-values.
- The significant and reference feature lists are correctly sized (e.g. 426 of 5,707 features after QC and univariate selection).
- Predicted metabolites carry a confidence rating (e.g. high confidence when both M+H[1+] and M(C13)+H[1+] forms are present).

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number.
- Limited metabolite database annotations cause de facto filtering of features before analysis; features from high-resolution profiling far exceed current database annotations.
- Some metabolites in metabolomics databases are not present in any of the genome-scale metabolic models, and models need improvement on lipid annotation, physiological context and tissue specificity.
- Upstream quantification and feature selection steps (e.g. cutoff choice for the significant list) can still be problematic and are not incorporated as a statistical matrix by the current version.

## Evidence

- [results] The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis.: "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
- [results] mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites.: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites."
- [readme] It leverages the organization of metabolic networks to predict functional activity directly from feature tables, bypassing metabolite identification.: "It leverages the organization of metabolic networks to predict functional activity directly from feature tables, bypassing metabolite identification."
- [methods] mummichog(3): core algorithm package for pathway/network analysis: "mummichog(3): core algorithm package for pathway/network analysis"
- [discussion] Mummichog is not designed to replace tandem mass spectrometry in metabolite identification.: "Mummichog is not designed to replace tandem mass spectrometry in metabolite identification."
- [results] if both the single-charged form M+H[1+] and the form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence.: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
