---
name: negative-mode-adduct-enforcement
description: Use when running mummichog on negative-ion-mode LC/MS feature tables
  (e.g. mzXML-derived yeast mutant vs wild-type data) and the predicted metabolites
  must be constrained to deprotonated M−H[−] species.
when_to_use_negative:
- Positive-ion-mode data — use the M+H[+] enforcement option instead of M−H[−]
- The goal is definitive metabolite identification — mummichog is not designed to
  replace tandem mass spectrometry in metabolite identification
- The metabolic model compounds lack correct neutral masses (salt formats present)
  — fix formulas first or look them up via compound identifiers before running
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0057
tools:
- name: mummichog
  role: Core package whose metabolite-prediction option enforces the M−H[−] (or M+H[+])
    adduct form during network/pathway analysis
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: metDataModel
  role: Data structure used to load the BioCyc yeast metabolic model (with compound
    neutral formulas) into mummichog
- name: MAVEN
  role: Upstream untargeted feature extraction, peak picking, quantification and alignment
    of yeast mzXML files producing the feature table the adduct rule is applied to
- name: JMS
  role: Identifier mapping and mass/empirical-compound search used to look up neutral
    formulas when model compounds lack them
  repo: https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:9cf721e486f2f1972fe73e1de85d175596295cda1c8e7b6c413d79d96a77cb7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - JMS
  - MAVEN
  - metDataModel
  - mummichog
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0057
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# negative-mode-adduct-enforcement

## Summary

Enforces the M−H[−] adduct form in mummichog's metabolite prediction when the input LC/MS feature table comes from negative-ion-mode acquisition, so that m/z-to-compound matching is restricted to chemically consistent deprotonated forms built from correct neutral formulas.

## When to use

Use when running mummichog on negative-ion-mode LC/MS feature tables (e.g. mzXML-derived yeast mutant vs wild-type data) and the predicted metabolites must be constrained to deprotonated M−H[−] species.

## When NOT to use

- Positive-ion-mode data — use the M+H[+] enforcement option instead of M−H[−]
- The goal is definitive metabolite identification — mummichog is not designed to replace tandem mass spectrometry in metabolite identification
- The metabolic model compounds lack correct neutral masses (salt formats present) — fix formulas first or look them up via compound identifiers before running

## Inputs

- Negative-ion-mode LC/MS m/z feature table with significant list and reference list (e.g. 426 significant of 5,707 features from MAVEN processing of yeast mzXML files)
- Metabolic model with list_of_pathways, list_of_reactions and list_of_compounds carrying neutral formulas (BioCyc yeast metabolic model via metDataModel / Azimuth DB)
- Optional annotation table on the features (structured with metDataModel, e.g. from JMS)

## Outputs

- Predicted metabolite list with per-metabolite confidence ratings and adduct/isotope assignments
- Pathway enrichment (Fisher's exact test with EASE correction, Gamma-modeled permutation null) and network module results as JSON strings
- result.html report with result tables and figures

## How to apply

Feed mummichog two lists of m/z features — the significant list (e.g. the 426 features selected by univariate t-test on the yeast mutant vs prototrophic wild-type comparison) and the reference list of all detected features — together with a metabolic model such as the BioCyc yeast model loaded via metDataModel from Azimuth DB. Enable the built-in option to enforce the M−H[−] form (the negative-mode analogue of the M+H[+] enforcement for positive mode) so that only features interpretable as deprotonated molecules contribute to metabolite prediction. The model's compounds must carry correct neutral formulas, not salt formats, because mummichog computes adducts (including salts) from the neutral formula mass; if formulas are absent, look them up via compound identifiers (e.g. with JMS/mass2chem utilities). Confidence ratings then filter predictions — e.g. co-occurrence of M+H[1+] and its M(C13)+H[1+] isotope form confers high confidence — so verify the adduct/isotope table rather than assuming enforcement alone guarantees correctness. On the published yeast dataset, enforcing M−H[−] had little effect: 24 of 28 predicted metabolites still agreed with the original targeted annotation (~86%), so treat this step as a chemical-consistency guard rather than a performance-changing parameter.

## Related tools

- **mummichog** (Core package whose metabolite-prediction option enforces the M−H[−] (or M+H[+]) adduct form during network/pathway analysis) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **metDataModel** (Data structure used to load the BioCyc yeast metabolic model (with compound neutral formulas) into mummichog)
- **MAVEN** (Upstream untargeted feature extraction, peak picking, quantification and alignment of yeast mzXML files producing the feature table the adduct rule is applied to)
- **JMS** (Identifier mapping and mass/empirical-compound search used to look up neutral formulas when model compounds lack them) — https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Predicted-vs-targeted agreement on the yeast data equals 24 of 28 comparable metabolites (≈0.857), with enforcing M−H[−] changing little relative to unenforced prediction
- The adduct/isotope table is built from correct neutral formula masses, not salt formats, for every compound in the loaded model
- High-confidence predictions are backed by co-observed isotope/adduct forms (e.g. M+H[1+] together with M(C13)+H[1+])
- result.html exists and is valid HTML; the pathway and network JSON outputs parse without error

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating overlap counts regardless of adduct enforcement
- Enforcement is optional and may have little effect on prediction outcomes (as observed on the yeast dataset)
- Many genome-scale metabolic models lack compound formulas, requiring identifier-based lookup before adduct calculation
- Features from high-resolution profiling experiments far exceed current metabolite database annotations, causing de facto filtering

## Evidence

- [results] Option to enforce M−H[−] in negative mode: "There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction."
- [results] Confidence rating via isotope/adduct co-occurrence: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [readme] Neutral formulas, not salt formats, drive adduct calculation: "If compound formulas are provided in a model, be sure to have correct neutral mass and not a salt format."
- [readme] Adducts computed from neutral formulas including salts: "We use neutral formula to calculate adducts in mass spec, which includes salts."
- [discussion] mummichog not a replacement for MS/MS identification: "Mummichog is not designed to replace tandem mass spectrometry in metabolite identification."
