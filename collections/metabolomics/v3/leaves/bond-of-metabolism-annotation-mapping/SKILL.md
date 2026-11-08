---
name: bond-of-metabolism-annotation-mapping
description: Use when starting from a SoM-annotated CYP450 substrate set (e.g. Zaretzki
  dataset for 679 substrates) and you need [compound, bond] BoMX−Y/non-BoMX−Y labels
  per isoform to train or evaluate reactive-bond predictors.
when_to_use_negative:
- Your labels are already bond-level (BoM) annotations such as EBoMD2 hold-out data
  — no SoM-to-BoM conversion is needed.
- Your query is de novo metabolite structure generation rather than reactive-bond
  labeling; use MetaboGen with SMIRKS rules instead.
- You lack substrate-metabolite pairs, since the conversion from SoMs to BoMs requires
  literature curation of those pairs.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0209
- http://edamontology.org/topic_3336
tools:
- name: CypBoM
  role: Downstream module trained on the BoM labels produced by this mapping; consumes
    the [compound, bond] instances to predict reactive bonds
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: Chemistry Development Kit (CDK)
  role: Computes the atom/bond descriptors (atomic number, ConnectedAtoms, AtomDescriptor)
    used in canonical bond-pair ordering
- name: CyProduct
  role: Suite whose EBoMD/EBoMD2 datasets and CypBoM module depend on the SoM-to-BoM
    annotation mapping
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:82efbf6c2bc98a9cf1834ed3ac6b1a3330da0eafd6a1d6801958e4936f51f789
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chemistry Development Kit (CDK)
  - CyProduct
  - CypBoM
  - WEKA
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0209
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bond-of-metabolism-annotation-mapping

## Summary

Converts site-of-metabolism (SoM) annotations from the Zaretzki CYP450 substrate dataset into bond-of-metabolism (BoM) labels over the EBoMD dataset, yielding 828 η-η, 1348 η-H and 86 η-SPN reactive bond instances across 679 substrates for training CypBoM classifiers.

## When to use

Use when starting from a SoM-annotated CYP450 substrate set (e.g. Zaretzki dataset for 679 substrates) and you need [compound, bond] BoMX−Y/non-BoMX−Y labels per isoform to train or evaluate reactive-bond predictors.

## When NOT to use

- Your labels are already bond-level (BoM) annotations such as EBoMD2 hold-out data — no SoM-to-BoM conversion is needed.
- Your query is de novo metabolite structure generation rather than reactive-bond labeling; use MetaboGen with SMIRKS rules instead.
- You lack substrate-metabolite pairs, since the conversion from SoMs to BoMs requires literature curation of those pairs.

## Inputs

- Zaretzki dataset (SoM annotations for 679 CYP450 substrates)
- Curated substrate-metabolite pairs from the literature
- Compounds checked against PubChem and DrugBank entries
- EBoMD.sdf (compound structures from Zaretzki's 679-compound substrate dataset)

## Outputs

- EBoMD BoM annotations: 828 η-η, 1348 η-H and 86 η-SPN bonds over 679 Zaretzki-derived substrates
- Binary BoMX−Y/non-BoMX−Y labels per [compound, bond] instance for each of the nine CYP450 isoforms (1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4)
- Canonically ordered η-η bond atom pairs ready for CDK feature generation

## How to apply

Begin with the public Zaretzki dataset providing SoMs for 679 substrates and curate the corresponding substrate-metabolite pairs from the literature, applying the paper's conversion rules to turn each SoM into the bond that is broken or formed. Write the resulting annotations into EBoMD.sdf with three bond-type classes: 828 η-η bonds, 1348 η-H bonds and 86 η-SPN bonds. For each [compound, bond] instance, assign the binary BoMX−Y/non-BoMX−Y label per isoform α ∈ {1A2, 2A6, 2B6, 2C8, 2C9, 2C19, 2D6, 2E1, 3A4}; note that η-η reactive bonds are strongly imbalanced (~5% reactive vs ~95% non-reactive), and that η+η-forming reactions are extremely rare (only 4 of 828 η-η reactions, all Cyclization), so treat them accordingly during labeling. Canonically order each η-η bond atom pair (smaller atomic number first, then ConnectedAtoms count CA(·), then lexicographic AtomDescriptor order) so bond instances are reproducibly represented. Keep a separate hold-out set (EBoMD2, 98 compounds) untouched by the mapping so model evaluation is not contaminated.

## Related tools

- **CypBoM** (Downstream module trained on the BoM labels produced by this mapping; consumes the [compound, bond] instances to predict reactive bonds) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **Chemistry Development Kit (CDK)** (Computes the atom/bond descriptors (atomic number, ConnectedAtoms, AtomDescriptor) used in canonical bond-pair ordering)
- **CyProduct** (Suite whose EBoMD/EBoMD2 datasets and CypBoM module depend on the SoM-to-BoM annotation mapping) — https://bitbucket.org/wishartlab/cyproduct/src/master/

## Examples

```
java -jar CyProduct.jar CC(=O)Nc1ccc(O)cc1 1A2,2A6,2B6,2C8,2C9,2C19,2D6,2E1,3A4 E:\Users\cyproduct\
```

## Evaluation signals

- Bond-type counts in the mapped EBoMD should be 828 η-η, 1348 η-H and 86 η-SPN over 679 substrates.
- Reactive η-η bonds should comprise roughly 5% of η-η instances (95% non-reactive), confirming label imbalance is preserved.
- Every labeled [compound, bond] instance carries a per-isoform binary BoMX−Y/non-BoMX−Y tag for all nine CYP450 enzymes.
- η-η bond atom pairs follow the canonical order: smaller atomic number first, then CA(·) connected-atoms count, then lexicographic AtomDescriptor order.
- Only 4 of 828 η-η reactions should involve η+η-forming bonds (all Cyclization type); anything else indicates a conversion error.

## Limitations

- The mapping depends on literature curation of substrate-metabolite pairs from the Zaretzki SoM dataset; curation errors propagate into BoM labels.
- η+η-forming reactions are nearly absent in EBoMD (4 of 828), so bond types that rare cannot be reliably learned or mapped.
- No software version numbers are stated for CDK or other tools used during annotation, limiting bit-for-bit reproducibility.
- The EBoMD/EBoMD2 sets are in the repository but the article text notes the repository location must be found via the publication link.

## Evidence

- [methods] Conversion of Zaretzki SoM dataset to BoM dataset by literature curation of substrate-metabolite pairs: "we converted Zaretzki’s SoM dataset to a corresponding BoM dataset, by applying the following rules"
- [methods] EBoMD composition: [evidence span withheld — anchor status: not_found]
- [readme] EBoMD.sdf source compounds: "EBoMD.sdf contains the compounds derived from the Zaretzki’s CYP450 substrate datasets of 679 compounds. It is used to train the CypBoM module."
- [methods] Class imbalance in η-η bonds: "the percentage of reactive η-η bonds (5%) is much less than the number of non-reactive η-η bonds (95%)"
- [methods] Rarity of η+η-forming reactions: [evidence span withheld — anchor status: not_found]
- [methods] Canonical bond ordering rule: "if the atomic numbers of atom i and atom j are different, then the pair is ordered to start with the atom having the smaller atomic number"
