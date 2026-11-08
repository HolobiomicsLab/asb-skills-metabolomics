---
name: genome-class-to-pathway-cooccurrence-counting
description: Use when you have a set of MIBiG BGCs with SMILES and (sub)classes and
  need genome-class-to-NPClassifier-pathway match frequencies for scoring BGC-metabolite
  links.
when_to_use_negative:
- When the candidate BGCs have no MIBiG (sub)class annotation or no product SMILES,
  since both are required to tabulate genome class against NPClassifier pathway
- When the goal is a direct structure-based chemical classification from MS/MS alone
  rather than class matching between genome and metabolome ontologies
- When no reference class table exists for the organism or BGC type of interest, since
  the method relies on empirically derived match fractions
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: NPClassifier
  role: Assigns NPClassifier pathway classifications to the SMILES strings extracted
    from MIBiG entries via the GNPS API, providing the chemical-class side of the
    co-occurrence table
- name: MIBiG 2.0
  role: Source repository of experimentally validated BGCs and their products, supplying
    the genome-based BGC classes and SMILES used to build the match table
  repo: https://mibig.secondarymetabolites.org
- name: mibig_classifications
  role: Jupyter notebooks and scripts to retrieve chemical classes (ClassyFire/NPClassifier)
    and BGC classifications for MIBiG BGCs and their products, and to connect the
    two by counting interactions
  repo: github.com/louwenjjr/mibig_classifications
- name: NPLinker
  role: Consumes the resulting class matching tables as the NPClassScore scoring tables
    to rule out unreliable BGC-compound matches
  repo: github.com/NPLinker/nplinker
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:e0213700531dccf47ce1d577bf5ea45537a1c8004706f6f3cd236f490d266889
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIBiG 2.0
  - NPClassifier
  - NPLinker
  - mibig_classifications
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# genome-class-to-pathway-cooccurrence-counting

## Summary

Builds an empirical match table between genome-based BGC classes (MIBiG) and chemical compound classes (NPClassifier pathways) by counting how often each genome class co-occurs with each pathway term across validated BGC-product pairs.

## When to use

Use when you have a set of MIBiG BGCs with SMILES and (sub)classes and need genome-class-to-NPClassifier-pathway match frequencies for scoring BGC-metabolite links.

## When NOT to use

- When the candidate BGCs have no MIBiG (sub)class annotation or no product SMILES, since both are required to tabulate genome class against NPClassifier pathway
- When the goal is a direct structure-based chemical classification from MS/MS alone rather than class matching between genome and metabolome ontologies
- When no reference class table exists for the organism or BGC type of interest, since the method relies on empirically derived match fractions

## Inputs

- MIBiG 2.0 entries in json format (downloaded from mibig.secondarymetabolites.org)
- SMILES strings extracted from each MIBiG entry
- MIBiG (sub)class labels extracted from each MIBiG entry json

## Outputs

- Class-by-pathway co-occurrence count table (genome class x NPClassifier pathway term)
- Normalised match fractions per genome class per pathway term (e.g. 0.76 for polyketide-to-Polyketides)
- Scoring tables usable for NPClassScore filtering in NPLinker

## How to apply

Download all MIBiG 2.0 entries from mibig.secondarymetabolites.org in json format and extract each entry's SMILES string together with its MIBiG (sub)class fields. For each SMILES, retrieve the NPClassifier pathway classification through the GNPS API, producing a per-BGC pair of genome class and pathway term. Tabulate, for each MIBiG genome-based class, how many BGCs receive each NPClassifier pathway term, then normalise by class size to yield the fraction of BGCs in each genome class matching each pathway (e.g. 76% of polyketide BGCs match the 'Polyketides' pathway, most RiPPs and NRPs match 'Amino acids and Peptides', 65% of terpene BGCs match 'Terpenoids'). These relative counts form the scoring table that the NPClassScore method uses to rule out unreliable BGC-compound matches in the NPLinker framework. Validate by checking that recomputed fractions reproduce the reported co-occurrence pattern for the major classes.

## Related tools

- **NPClassifier** (Assigns NPClassifier pathway classifications to the SMILES strings extracted from MIBiG entries via the GNPS API, providing the chemical-class side of the co-occurrence table)
- **MIBiG 2.0** (Source repository of experimentally validated BGCs and their products, supplying the genome-based BGC classes and SMILES used to build the match table) — https://mibig.secondarymetabolites.org
- **mibig_classifications** (Jupyter notebooks and scripts to retrieve chemical classes (ClassyFire/NPClassifier) and BGC classifications for MIBiG BGCs and their products, and to connect the two by counting interactions) — github.com/louwenjjr/mibig_classifications
- **NPLinker** (Consumes the resulting class matching tables as the NPClassScore scoring tables to rule out unreliable BGC-compound matches) — github.com/NPLinker/nplinker

## Examples

```
git clone https://github.com/louwenjjr/mibig_classifications && conda activate myenv && jupyter notebook
```

## Evaluation signals

- Recomputed match fractions reproduce the reported pattern: 76% of polyketide BGCs match the NPClassifier 'Polyketides' pathway
- Most RiPPs and NRPs match the 'Amino acids and Peptides' pathway in the recomputed table
- 65% of terpene BGCs match the 'Terpenoids' pathway in the recomputed table
- Every MIBiG genome class row in the output table normalises to fractions summing to 1 (or 100%) across pathway terms
- The table contains no genome class or pathway term absent from the source MIBiG json classes or NPClassifier API responses

## Limitations

- Class matching relies on MIBiG entries having complete (sub)class and SMILES annotations; entries missing either cannot contribute to the table
- NPClassifier classification of SMILES via the GNPS API may leave some compounds unclassified, reducing effective counts per genome class
- The match fractions are empirical and dataset-dependent, so updating the MIBiG corpus changes the table and the NPClassScore thresholds

## Evidence

- [results] MIBiG-based class matching shows 76% of polyketide BGCs match the NPClassifier 'Polyketides' pathway, most RiPPs and NRPs match 'Amino acids and Peptides', and 65% of terpene BGCs match 'Terpenoids': [evidence span withheld — anchor status: not_found]
- [discussion] All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format: "All entries from MIBiG 2.0 were downloaded from mibig.secondarymetabolites.org in json format"
- [discussion] the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [readme] Series of jupyter notebooks and scripts to retrieve chemical classes (ClassyFire/NPClassifier), as well as BGC classifications for MIBiG BGCs and their products.: [evidence span withheld — anchor status: external_doc]
- [readme] These two types of classifications are then connected to eachother by counting their interactions, ideally matching certain chemical and BGC classes.: [evidence span withheld — anchor status: external_doc]
