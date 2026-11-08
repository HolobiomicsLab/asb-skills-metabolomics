---
name: cyclic-peptide-fragment-ion-series-interpretation
description: Use when experimental tandem mass spectra of NRP/PK compounds (e.g.,
  desferrioxamine B, pseudacyclin A) must be dereplicated against known building-block
  sequences using b-ion and a-ion fragment series with defined m/z tolerances.
when_to_use_negative:
- Input spectra are of non-peptidic natural products better served by Dereplicator+
  in silico fragmentation trees
- The user cannot construct custom building-block and sequence databases, which NRPro
  does not support
- A precursor mass filter must remain enabled when the expected sequence is a single
  top hit, as disabling it changed the desferrioxamine B report
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MassSpecBlocks
  role: Converts SMILES chemical structures into sequences of building blocks and
    exports them in plain text format for CycloBranch; builds custom NRP/PK databases
  repo: https://github.com/privrja/MassSpecBlocks
- name: CycloBranch
  role: Performs the actual annotation/dereplication of CID and HCD spectra against
    the exported sequence database under the specified ion-tolerance settings
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: NRPro
  role: Alternative tool used for comparison under the same 5 ppm tolerance and b/a-ion
    configuration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:274d876093038ed2b44113e1c7865276bc7612557a5fdefe53c91b98cabc9ece
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/privrja/MassSpecBlocks
  tools:
  - CycloBranch
  - MassSpecBlocks
  - NRPro
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cyclic-peptide-fragment-ion-series-interpretation

## Summary

A method for annotating experimental CID and HCD tandem mass spectra of nonribosomal peptides and polyketides by comparing them against theoretical fragment ion series derived from building-block sequences, with parameterization tuned for cyclic, branched, and branch-cyclic NRP/PK structures.

## When to use

Use when experimental tandem mass spectra of NRP/PK compounds (e.g., desferrioxamine B, pseudacyclin A) must be dereplicated against known building-block sequences using b-ion and a-ion fragment series with defined m/z tolerances.

## When NOT to use

- Input spectra are of non-peptidic natural products better served by Dereplicator+ in silico fragmentation trees
- The user cannot construct custom building-block and sequence databases, which NRPro does not support
- A precursor mass filter must remain enabled when the expected sequence is a single top hit, as disabling it changed the desferrioxamine B report

## Inputs

- 44 HCD and 44 CID tandem mass spectra of NRP/PK compounds deposited in the evaluation set
- MassSpecBlocks sequence and building block containers (lists of sequences, blocks, and modifications)
- Exported plain text sequence database accepted by CycloBranch

## Outputs

- Per-condition annotation accuracy table saved as annotator_accuracy_by_condition.csv
- CycloBranch annotation report per spectrum, e.g. desferrioxamine B sequence as single top hit with 96.4% weighted ratio of matched peaks

## How to apply

First, export the MassSpecBlocks sequence and building block containers into the plain text format accepted by CycloBranch, including lists of sequences, blocks, and modifications. Then configure CycloBranch annotation with precursor and fragment ion m/z error tolerances of 5 ppm, set the minimum threshold of relative peak intensity to zero, and restrict the search to b-ions and a-ions with at most one neutral loss of H2O or NH3. Run the annotation separately over the CID spectrum set and the HCD spectrum set against the exported sequence database. Compare each returned annotation report against the reference compound identity and compute annotation accuracy per fragmentation condition. Validation relies on reproducing per-condition accuracy values comparable to those reported in the paper's results section.

## Related tools

- **MassSpecBlocks** (Converts SMILES chemical structures into sequences of building blocks and exports them in plain text format for CycloBranch; builds custom NRP/PK databases) — https://github.com/privrja/MassSpecBlocks
- **CycloBranch** (Performs the actual annotation/dereplication of CID and HCD spectra against the exported sequence database under the specified ion-tolerance settings) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **NRPro** (Alternative tool used for comparison under the same 5 ppm tolerance and b/a-ion configuration)

## Evaluation signals

- CycloBranch reports the correct compound's sequence as the single top hit, e.g. desferrioxamine B with 96.4% weighted ratio of matched peaks
- Computed CID and HCD accuracy values agree with the per-condition values reported in the paper's results section
- Annotation output matches the reference compound identity from the 44-compound NRP/PK evaluation set
- Branch-cyclic structures such as pseudacyclin A are correctly decomposed (e.g. cyclo(Pro-Ile-Ile-Orn-Phe) ring plus Ac-Ile branch)
- Precursor and fragment ion m/z error tolerances remain at 5 ppm (or 0.005 Da for Dereplicator) with b- and a-ions and at most one neutral loss of H2O or NH3

## Limitations

- PK siderophores contain uncommon building blocks that reverse the orientation of peptide bonds, further complicating mass spectra interpretation
- The peptide bond connecting Ac to Ile on the branch was not automatically marked in pseudacyclin A because the resulting Ac block would contain only three nonhydrogen atoms
- It is not always easy to export flawless sequences of building blocks for tandem mass spectra analysis from Norine
- Automated interpretation of NRP mass spectra remains a challenging task due to the complex structures of the molecules

## Evidence

- [results] Configure CycloBranch annotation with 5 ppm tolerances and restricted ion types: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro."
- [results] Search only b- and a-ions with one neutral loss to unify configuration: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [results] Desferrioxamine B reproduced as single top hit at 96.4% weighted matched-peak ratio: [evidence span withheld — anchor status: not_found]
- [results] MassSpecBlocks correctly decomposed branch-cyclic pseudacyclin A into six building blocks: "MassSpecBlocks correctly determined the branch-cyclic structure of the molecule."
- [abstract] Export sequences and blocks into plain text for CycloBranch: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [intro] PK siderophores reverse peptide bond orientation complicating spectra: "PK siderophores contain uncommon building blocks that turn over the orientation of peptide bonds in molecules (e.g., desferrioxamines or ornibactins) and further complicate mass spectra interpretation"
