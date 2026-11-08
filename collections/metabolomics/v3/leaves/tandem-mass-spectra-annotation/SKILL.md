---
name: tandem-mass-spectra-annotation
description: Use when you have HCD or CID tandem mass spectra of NRP/PK compounds
  and need dereplication or de novo annotation against custom building-block sequence
  databases rather than generic spectral libraries.
when_to_use_negative:
- Input spectra are for generic small molecules with no NRP/PK sequence database available,
  since Dereplicator+/NRPro-style generic tools or raw library search would be more
  appropriate
- You only need structure drawing or SMILES parsing without spectral annotation —
  SmilesDrawer or MassSpecBlocks alone suffice without CycloBranch
- Your compound set requires fully novel de novo sequencing where no candidate building
  blocks or sequence can be pre-defined; this skill is dereplication/annotation against
  curated sequences, not ab initio sequencing
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MassSpecBlocks
  role: Builds custom sequence and building-block databases from SMILES and exports
    them into the plain text format used by CycloBranch for annotation
  repo: https://github.com/privrja/MassSpecBlocks
- name: CycloBranch
  role: Performs the actual tandem mass spectra annotation (de novo analysis and dereplication)
    against the exported sequence/building-block databases
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: SmilesDrawer
  role: Draws chemical structures from SMILES format within the MassSpecBlocks interface
    during database construction
  repo: https://github.com/privrja/smilesDrawer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:c3b87c3295a6f6f850bbd9f364761285a97ad0401a188acb5cb99a57565f3736
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/privrja/MassSpecBlocks
  tools:
  - CycloBranch
  - MassSpecBlocks
  - SmilesDrawer
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tandem-mass-spectra-annotation

## Summary

Annotates experimental tandem mass spectra of nonribosomal peptides and polyketides by comparing them against theoretical spectra built from user-defined sequences of building blocks exported from MassSpecBlocks into CycloBranch.

## When to use

Use when you have HCD or CID tandem mass spectra of NRP/PK compounds and need dereplication or de novo annotation against custom building-block sequence databases rather than generic spectral libraries.

## When NOT to use

- Input spectra are for generic small molecules with no NRP/PK sequence database available, since Dereplicator+/NRPro-style generic tools or raw library search would be more appropriate
- You only need structure drawing or SMILES parsing without spectral annotation — SmilesDrawer or MassSpecBlocks alone suffice without CycloBranch
- Your compound set requires fully novel de novo sequencing where no candidate building blocks or sequence can be pre-defined; this skill is dereplication/annotation against curated sequences, not ab initio sequencing

## Inputs

- SMILES-format chemical structures of NRP/PK compounds
- CID tandem mass spectra of NRP/PK compounds
- HCD tandem mass spectra of NRP/PK compounds
- MassSpecBlocks-exported plain text sequence and building block container files

## Outputs

- CycloBranch annotation report with top-hit sequences and weighted ratio of matched peaks (e.g. 96.4% for desferrioxamine B)
- Per-condition annotation accuracy table (annotator_accuracy_by_condition.csv)

## How to apply

Build sequence and building-block databases in MassSpecBlocks from SMILES structures, then export them into the plain text format consumed by CycloBranch. Configure CycloBranch annotation with precursor and fragment ion m/z error tolerances of 5 ppm (0.005 Da for Dereplicator/Dereplicator+ comparison runs), minimum threshold of relative peak intensity set to zero, and restrict the search to b-ions and a-ions allowing at most one neutral loss of H2O or NH3 to unify configuration with NRPro. Run annotation separately over CID and HCD spectra sets against the exported database, then compare each returned annotation report against the reference compound identity to compute per-condition accuracy (e.g. desferrioxamine B scored as a single top hit with 96.4% weighted ratio of matched peaks). The rationale is that automated interpretation of NRP spectra is hard because PK siderophores contain uncommon building blocks that turn over peptide-bond orientation, so a curated block database plus permissive ion-type filtering (zero intensity threshold, 5 ppm tolerance) maximizes match recall without over-fragmenting the theoretical spectrum.

## Related tools

- **MassSpecBlocks** (Builds custom sequence and building-block databases from SMILES and exports them into the plain text format used by CycloBranch for annotation) — https://github.com/privrja/MassSpecBlocks
- **CycloBranch** (Performs the actual tandem mass spectra annotation (de novo analysis and dereplication) against the exported sequence/building-block databases) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **SmilesDrawer** (Draws chemical structures from SMILES format within the MassSpecBlocks interface during database construction) — https://github.com/privrja/smilesDrawer

## Evaluation signals

- Reproduced CID and HCD accuracy values agree with the Annotator accuracy values reported in the paper's results section for the 44 NRP/PK compound set
- The weighted ratio of matched peaks for a reference compound (e.g. desferrioxamine B) reaches ~96.4% when its true sequence is the single top hit
- Annotation reports list the correct compound as top hit with the expected ion series (b-ions and a-ions, at most one neutral loss of H2O or NH3)
- Precursor and fragment ion m/z errors in the report fall within the configured 5 ppm tolerance
- The exported plain text file is accepted by CycloBranch without errors and all building blocks carry valid SMILES, formulas, and sequences

## Limitations

- PK siderophores contain uncommon building blocks that turn over the orientation of peptide bonds, further complicating mass spectra interpretation and requiring custom block definitions
- The peptide bond connecting Ac to Ile on the branch was not automatically marked because the resulting Ac block would contain only three nonhydrogen atoms, so manual correction can be needed
- CycloBranch requires hand-built sequence and building block databases; without MassSpecBlocks the process requires a lot of manual labor
- Configuring ion-type filters too permissively (e.g. including y-ions or multiple neutral losses) breaks comparability with NRPro runs

## Evidence

- [results] Configure CycloBranch annotation with ion-type filters and ppm tolerances: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [abstract] Export sequence database into plain text format for CycloBranch: "The sequences and building blocks created in MassSpecBlocks can be easily exported into a plain text format used by CycloBranch"
- [results] Ion-type filter restricted to b- and a-ions with one neutral loss: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [results] Minimum relative peak intensity threshold set to zero: "we set up the minimum threshold of relative peak intensity to zero."
- [results] Desferrioxamine B top-hit weighted matched-peak ratio: [evidence span withheld — anchor status: not_found]
