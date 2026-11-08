---
name: mibig-json-bgc-class-extraction
description: Use when building chemical-class-matching tables that bridge genome-based
  BGC classes (e.g. from MIBiG 2.0 JSON) to structure-based NPClassifier pathway terms
  for a BGC-metabolite linking analysis.
when_to_use_negative:
- When the goal is per-strain or per-dataset BGC classification rather than building
  the generic MIBiG-wide class-matching tables — antiSMASH or BiG-SCAPE outputs should
  be classified instead of re-deriving MIBiG co-occurrence counts.
- When candidate links already have NPClassScore scores and only filtering (e.g. cut-off
  of 0.25) is needed; no new MIBiG extraction is required.
- When metabolite-level classification alone is needed without a genome-based counterpart
  (e.g. pure CANOPUS/NPClassifier predictions on an mgf file) — the MIBiG co-occurrence
  step is unnecessary.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MIBiG 2.0
  role: Source of validated BGC entries; JSON download provides the SMILES strings
    and genome-based (sub)classes that this skill extracts.
- name: NPClassifier
  role: Receives each extracted SMILES via the GNPS API and returns the pathway classification
    used to build the co-occurrence table.
- name: mibig_classifications
  role: Repository of Jupyter notebooks and scripts implementing this extraction-and-counting
    workflow, including the class-matching tables used by NPClassScore.
  repo: github.com/louwenjjr/mibig_classifications
- name: NPLinker
  role: Consumes the matched class tables produced by this skill as the NPClassScore
    scoring component for BGC-MS/MS link prioritisation.
  repo: github.com/NPLinker/nplinker
- name: GNPS
  role: Hosts the API through which NPClassifier classifications for the extracted
    SMILES are retrieved.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:4b018a694922a3c748d149f2d7849c3d6d6d555be8cdf72ea90e8cf2f286df38
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MIBiG 2.0
  - NPClassifier
  - NPLinker
  - mibig_classifications
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mibig-json-bgc-class-extraction

## Summary

Extracts SMILES strings and MIBiG genome-based (sub)classes from MIBiG 2.0 JSON entries so that NPClassifier pathway classifications can be retrieved for each product and a genome-class-by-structure-class co-occurrence table can be built for NPClassScore scoring.

## When to use

Use when building chemical-class-matching tables that bridge genome-based BGC classes (e.g. from MIBiG 2.0 JSON) to structure-based NPClassifier pathway terms for a BGC-metabolite linking analysis.

## When NOT to use

- When the goal is per-strain or per-dataset BGC classification rather than building the generic MIBiG-wide class-matching tables — antiSMASH or BiG-SCAPE outputs should be classified instead of re-deriving MIBiG co-occurrence counts.
- When candidate links already have NPClassScore scores and only filtering (e.g. cut-off of 0.25) is needed; no new MIBiG extraction is required.
- When metabolite-level classification alone is needed without a genome-based counterpart (e.g. pure CANOPUS/NPClassifier predictions on an mgf file) — the MIBiG co-occurrence step is unnecessary.

## Inputs

- MIBiG 2.0 JSON entries downloaded from mibig.secondarymetabolites.org
- SMILES strings extracted from each MIBiG entry's JSON fields
- MIBiG (sub)class annotations extracted from each entry's JSON fields

## Outputs

- Class-by-class co-occurrence table counting MIBiG genome-based classes against NPClassifier pathway/superclass/class terms
- Per-class match-fraction table (e.g. fraction of polyketide BGCs matching 'Polyketides', RiPP/NRP BGCs matching 'Amino acids and Peptides', terpene BGCs matching 'Terpenoids')
- Matched class tables usable as NPClassScore scoring tables in NPLinker

## How to apply

Download all MIBiG 2.0 entries from mibig.secondarymetabolites.org in JSON format. From each entry's JSON fields, extract the SMILES string of the annotated product together with the MIBiG (sub)class annotation. Send each SMILES to the NPClassifier service via the GNPS API to obtain the pathway, superclass, and class terms. Then count, for each MIBiG genome-based class, how many BGCs receive each NPClassifier pathway term, and compute the fraction of BGCs in each genome class that match each pathway term — the article reports e.g. 76% of polyketide BGCs matching the 'Polyketides' pathway and 65% of terpene BGCs matching 'Terpenoids'. These relative counts form the matched class tables used by NPClassScore in NPLinker to rule out unreliable BGC-compound matches; the code and scoring tables are maintained in the louwenjjr/mibig_classifications repository as Jupyter notebooks and scripts.

## Related tools

- **MIBiG 2.0** (Source of validated BGC entries; JSON download provides the SMILES strings and genome-based (sub)classes that this skill extracts.)
- **NPClassifier** (Receives each extracted SMILES via the GNPS API and returns the pathway classification used to build the co-occurrence table.)
- **mibig_classifications** (Repository of Jupyter notebooks and scripts implementing this extraction-and-counting workflow, including the class-matching tables used by NPClassScore.) — github.com/louwenjjr/mibig_classifications
- **NPLinker** (Consumes the matched class tables produced by this skill as the NPClassScore scoring component for BGC-MS/MS link prioritisation.) — github.com/NPLinker/nplinker
- **GNPS** (Hosts the API through which NPClassifier classifications for the extracted SMILES are retrieved.)

## Evaluation signals

- Reproduce the reported co-occurrence pattern: 76% of polyketide BGCs match the NPClassifier 'Polyketides' pathway, most RiPP and NRP BGCs match 'Amino acids and Peptides', and 65% of terpene BGCs match 'Terpenoids'.
- Verify that every MIBiG 2.0 entry with a SMILES field yields both a SMILES string and a MIBiG (sub)class annotation — entries missing either field should be excluded from the co-occurrence counts.
- Check that the match fractions in the co-occurrence table reproduce the article's stated percentages within the precision given in the text.
- Confirm the resulting matched class tables load successfully as NPClassScore scoring tables in NPLinker (the scoring tables are versioned in the mibig_classifications repo).
- Ensure SMILES sent to NPClassifier are chemically valid so that the GNPS API returns pathway/superclass/class terms rather than errors.

## Limitations

- MIBiG entries that lack an annotated SMILES string cannot be classified via NPClassifier and are dropped from the co-occurrence table.
- The co-occurrence fractions are empirical and depend on MIBiG 2.0 coverage; they must be updated when new MIBiG entries are added (the article notes that the scoring tables can be further updated).
- The skill only bridges genome-based BGC classes to structure-based chemical classes; it does not itself validate individual BGC-metabolite links, and the lack of verified genome-metabolome links remains a bottleneck for validating downstream linking methods.

## Evidence

- [discussion] Download MIBiG entries and extract SMILES + classes: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format. The SMILES and MIBiG (sub)classes were extracted"
- [discussion] Retrieve NPClassifier classifications via GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [discussion] Scoring tables are updated and versioned in the mibig_classifications repo: "Code and scoring tables are available at github.com/louwenjjr/mibig_classifications"
- [readme] README describes counting interactions between the two classification types: [evidence span withheld — anchor status: external_doc]
- [readme] README states repo retrieves both chemical classes and BGC classifications for MIBiG BGCs: [evidence span withheld — anchor status: external_doc]
