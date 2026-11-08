---
name: table-driven-parser-validation
description: Use when a lipidomics dataset built from mass-spectrometric experiments
  uses vendor shorthand lipid names (headgroup plus acyl/alkyl/alkenyl chain tokens,
  sphingoid bases, placeholders like 0:0) that must be converted before upload into
  LipidOne 2.0.
when_to_use_negative:
- Input lipid names are already in LipidOne 2.0-compatible molecular-species or sum-composition
  form, so no translation is needed before 'Load Data Matrix' upload
- 'The goal is downstream data mining within LipidOne 2.0 (Basic Protocol 3: data
  mining, biomarker identification, Lipid Pathway analysis) rather than name-format
  translation'
- The dataset is a polar-metabolomics feature table, not a lipidomic matrix organized
  around lipid classes, molecular species, and building blocks
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: LipidOne 2.0
  role: Target platform whose accepted lipid-name grammar (molecular-species and sum-composition
    forms) defines the translation targets; web tool at https://lipidone.eu
  repo: https://lipidone.eu
- name: R
  role: Implementation language for the parser, chosen because LipidOne 2.0's back-end
    interactions are coded with R scripts
- name: Lipid One
  role: Predecessor desktop tool; its README shows the original implementation is
    Perl-based and Windows-only, so the R parser targets LipidOne 2.0 rather than
    this version
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:e6d58eba44fb822dc3211ce7595cafeb6475e6cae027ae56649cfc77ed043afc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lipid One
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# table-driven-parser-validation

## Summary

A test-vector-driven approach to translating mass-spectrometry vendor lipid notations (e.g. PE(P-16:0/18:1), PE(P-34:1)) into the forms LipidOne 2.0 accepts — molecular-species (PE P-16:0_18:1) and sum-composition (PE O-34:2) — with acceptance gated on 100% reproduction of the article's Table 1/Table 2 reference mappings.

## When to use

Use when a lipidomics dataset built from mass-spectrometric experiments uses vendor shorthand lipid names (headgroup plus acyl/alkyl/alkenyl chain tokens, sphingoid bases, placeholders like 0:0) that must be converted before upload into LipidOne 2.0.

## When NOT to use

- Input lipid names are already in LipidOne 2.0-compatible molecular-species or sum-composition form, so no translation is needed before 'Load Data Matrix' upload
- The goal is downstream data mining within LipidOne 2.0 (Basic Protocol 3: data mining, biomarker identification, Lipid Pathway analysis) rather than name-format translation
- The dataset is a polar-metabolomics feature table, not a lipidomic matrix organized around lipid classes, molecular species, and building blocks

## Inputs

- Vendor/experimental lipid name strings using shorthand notation (e.g. PE(P-16:0/18:1), PE(P-34:1)), including sphingoid-base descriptors (e.g. d16:1) and placeholder chain positions (e.g. 0:0)
- Reference mapping table of vendor/notation form → LipidOne 2.0 target form drawn from the article's Table 1 and Table 2

## Outputs

- lipid_name_translations.csv containing each input name, the LipidOne 2.0 molecular-species form, the sum-composition form, and per-example match status
- Lipid name column in a form loadable into LipidOne 2.0 (Basic Protocol 2: Uploading a dataset into LipidOne 2.0)

## How to apply

First encode the article's Table 1 and Table 2 vendor-notation → LipidOne 2.0 target mappings (e.g. PE(P-16:0/18:1) → PE P-16:0_18:1; PE(P-34:1) → PE O-34:2) as a test-vector table, since these tables are the authoritative specification of acceptable names. Then, in R — the language of LipidOne 2.0's back-end scripts — implement a tokenizer that splits each input string into the headgroup/lipid class, chain descriptors (acyl, alkyl, or alkenyl chains and sphingoid bases such as d16:1), and placeholder positions (e.g. 0:0), and emit both the molecular-species form (individual chain identities retained) and the sum-composition form (chain detail aggregated into the class-level shorthand). Run every test vector through the parser and compare emitted strings against the Table 1/Table 2 targets, writing input names, both translated forms, and per-example match status to lipid_name_translations.csv. Acceptance requires 100% of examples to reproduce the exact expected strings; any mismatch is reported as a failed case, because the Support Protocol treats lipid nomenclature as a prerequisite for loading data into the tool.

## Related tools

- **LipidOne 2.0** (Target platform whose accepted lipid-name grammar (molecular-species and sum-composition forms) defines the translation targets; web tool at https://lipidone.eu) — https://lipidone.eu
- **R** (Implementation language for the parser, chosen because LipidOne 2.0's back-end interactions are coded with R scripts)
- **Lipid One** (Predecessor desktop tool; its README shows the original implementation is Perl-based and Windows-only, so the R parser targets LipidOne 2.0 rather than this version) — github.com/matteogiulietti/LipidOne

## Examples

```
Rscript lipid_name_parser.R --vectors table1_table2_test_vectors.csv --out lipid_name_translations.csv
```

## Evaluation signals

- 100% of the Table 1 and Table 2 mapping examples reproduce the exact expected molecular-species/sum-composition strings — any mismatch is reported as a failed case and blocks acceptance
- Both target forms are emitted for every input: a molecular-species string with individual chain identities (e.g. PE P-16:0_18:1) and a sum-composition string with aggregated shorthand (e.g. PE O-34:2)
- Tokenizer correctly separates headgroup from acyl, alkyl, and alkenyl chain descriptors, sphingoid bases (d16:1), and placeholder positions (0:0) across all test vectors
- lipid_name_translations.csv is produced with columns for input name, both translated forms, and per-example match status

## Limitations

- The article section does not state a software license for LipidOne 2.0 or whether its source code is publicly hosted; the linked GitHub README describes the older desktop Lipid One, a stand-alone Perl application for Microsoft Windows, not the R-based 2.0 back end
- Example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are accessible only through the web platform's 'START NOW' → 'Load Data Matrix' page rather than a persistent repository accession, complicating independent validation of name formats
- The context provides the PE(P-16:0/18:1)-family mapping examples in summary; the full Table 1/Table 2 vectors must be obtained from the article to execute the validation

## Evidence

- [abstract] Support Protocol: Lipid nomenclature from spectrometric experiments: "Support Protocol: Lipid nomenclature from spectrometric experiments"
- [methods] Vendor notations such as PE(P-16:0/18:1) are translated into the LipidOne 2.0-compatible molecular-species form PE P-16:0_18:1 or sum-composition form PE O-34:2.: [evidence span withheld — anchor status: not_found]
- [discussion] coding its back-end interactions with the R scripts: "coding its back-end interac- tions with the R scripts"
- [readme] LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system.: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
