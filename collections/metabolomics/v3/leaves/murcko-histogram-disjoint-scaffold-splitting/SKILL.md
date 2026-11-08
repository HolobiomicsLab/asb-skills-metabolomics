---
name: murcko-histogram-disjoint-scaffold-splitting
description: Use when fine-tuning or evaluating spectral models (DreaMS, MIST, MS2DeepScore)
  on paired spectrum–structure data and you must guarantee no training molecule shares
  a Murcko scaffold with test molecules.
when_to_use_negative:
- When the dataset has so few distinct scaffolds that a fully disjoint split starves
  one partition (e.g. narrow homologous series where all molecules share one scaffold)
  — report scaffold scarcity rather than forcing the split.
- When evaluating de novo or pre-trained-only embeddings (e.g. zero-shot DreaMS cosine
  similarity) where structural leakage is not at issue for the comparison being made.
- When the input is unlabeled raw spectra (GeMS pre-training corpus) with no associated
  structures, since Murcko histograms cannot be computed without molecule identities.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: DreaMS
  role: Consumes the scaffold-disjoint splits when fine-tuning its 116M-parameter
    transformer for supervised tasks (similarity, fingerprints, chemical properties,
    fluorine detection); the split ensures reported metrics reflect unseen-scaffold
    generalization.
  repo: https://github.com/pluskal-lab/DreaMS
- name: MoNA
  role: One of the two standard training spectral libraries whose molecules are partitioned
    by the scaffold-disjoint split before DreaMS fine-tuning.
- name: NIST20
  role: The second standard training spectral library split alongside MoNA using Murcko-histogram
    disjointness.
- name: HDF5
  role: Binary storage format for spectra (GeMS and in-house test sets) from which
    spectra–structure pairs are loaded for split construction via dreams.utils.data.
- name: MassIVE (MSV000094528)
  role: Source of the 17,052-spectrum in-house fluorine test set whose spectra are
    held out under the same scaffold-disjoint discipline.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:0cadc308a0710f4aa1ddc24daafb8e27ea00bccfac3a8750e79fa13c63cdb278
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - HDF5
  - MassIVE (MSV000094528)
  - MoNA
  - NIST20
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Murcko-histogram-disjoint scaffold splitting

## Summary

Constructs train/val/test splits of spectra–molecule pairs so that Bemis–Murcko scaffold histograms are disjoint between splits, eliminating structural leakage when fine-tuning DreaMS or evaluating supervised MS/MS annotation heads (e.g. fluorine detection, fingerprints).

## When to use

Use when fine-tuning or evaluating spectral models (DreaMS, MIST, MS2DeepScore) on paired spectrum–structure data and you must guarantee no training molecule shares a Murcko scaffold with test molecules.

## When NOT to use

- When the dataset has so few distinct scaffolds that a fully disjoint split starves one partition (e.g. narrow homologous series where all molecules share one scaffold) — report scaffold scarcity rather than forcing the split.
- When evaluating de novo or pre-trained-only embeddings (e.g. zero-shot DreaMS cosine similarity) where structural leakage is not at issue for the comparison being made.
- When the input is unlabeled raw spectra (GeMS pre-training corpus) with no associated structures, since Murcko histograms cannot be computed without molecule identities.

## Inputs

- Paired spectrum–molecule records (spectra from MoNA, NIST20, GeMS, or MassIVE .mzML-derived HDF5; structures as SMILES/InChI)
- Bemis–Murcko scaffold definitions derived from each molecule's structure
- Test sets such as the 17,052-spectrum fluorine in-house set (MassIVE MSV000094528)

## Outputs

- Disjoint train/val/test split assignments keyed by scaffold
- Per-scaffold count histograms per split (proof of zero overlap)
- Split indices consumable by DreaMS fine-tuning scripts (e.g. for focal BCE training and checkpoint selection on validation)

## How to apply

Enumerate each molecule's Bemis–Murcko scaffold from its SMILES/InChI, aggregate per-scaffold spectral counts into a histogram, then allocate scaffolds to train/val/test partitions so that no scaffold histogram bin is shared between splits. For DreaMS fluorine detection, this split is applied over the MoNA and NIST20 training spectral libraries before fine-tuning the 116M-parameter transformer with focal binary cross-entropy (α=0.8, γ=0.5), and the same disjointness applies to the 17,052-spectrum in-house test set from MassIVE MSV000094528. The rationale is that if a scaffold appears in both splits, the model can interpolate from near-identical training spectra rather than learning spectrum-to-structure mapping, inflating precision/recall — this is why the 0.91 precision at 0.57 recall on 17,000 unreported spectra is trusted as generalization. Save the resulting split indices so that supervised heads (similarity, fingerprint, chemical property, fluorine) all train on the same disjoint partitions.

## Related tools

- **DreaMS** (Consumes the scaffold-disjoint splits when fine-tuning its 116M-parameter transformer for supervised tasks (similarity, fingerprints, chemical properties, fluorine detection); the split ensures reported metrics reflect unseen-scaffold generalization.) — https://github.com/pluskal-lab/DreaMS
- **MoNA** (One of the two standard training spectral libraries whose molecules are partitioned by the scaffold-disjoint split before DreaMS fine-tuning.)
- **NIST20** (The second standard training spectral library split alongside MoNA using Murcko-histogram disjointness.)
- **HDF5** (Binary storage format for spectra (GeMS and in-house test sets) from which spectra–structure pairs are loaded for split construction via dreams.utils.data.)
- **MassIVE (MSV000094528)** (Source of the 17,052-spectrum in-house fluorine test set whose spectra are held out under the same scaffold-disjoint discipline.)

## Examples

```
from dreams.utils.data import load_spectra; spectra = load_spectra('MSV000094528_test.hdf5'); splits = murcko_disjoint_split(spectra, train=0.8, val=0.1, test=0.1)
```

## Evaluation signals

- Verify no Bemis–Murcko scaffold is shared between train, val, and test: compute per-scaffold histograms for each split and assert zero intersection of scaffold keys.
- Confirm the test set is actually unseen: e.g. the 17,052-spectrum in-house fluorine set (MSV000094528) must not contribute any scaffold to training.
- Check split sizes preserve the paper's operating point — fine-tuned DreaMS on MoNA/NIST20 with focal BCE (α=0.8, γ=0.5) reaches 0.91 precision at 0.57 recall on the disjoint test set; a leaky split would show inflated precision well above this.
- Validate that each molecule appears in exactly one partition, and that spectra of molecules sharing a scaffold are never split across train/test.
- Cross-check against baselines: SIRIUS 5.6.3 precision ≤ 0.51 on the same test set provides an external sanity anchor.

## Limitations

- Disjointness is enforced at the Murcko-scaffold level; molecules with novel side-chain decoration on a shared scaffold are excluded from splits, which can reduce effective training diversity.
- The fine-tuning of DreaMS for fingerprint prediction should be understood as a proof of concept, not production-ready, so split-based metrics in that regime may not transfer.
- The method disregards MS1 isotopic patterns and adduct distributions and trains only on tandem mass spectra (positive ionization, singly charged precursors), so scaffold-disjoint splits do not mitigate leakage across ionization states or adducts.
- Only a subset of available mass spectra was used for training, and the full potential of training on raw unannotated mass spectra remains to be unlocked.

## Evidence

- [methods] Construct Murcko-histogram-disjoint train/val/test splits so that training and evaluation molecules are structurally disjoint (no shared Murcko scaffolds between splits).: [evidence span withheld — anchor status: not_found]
- [results] Whereas SIRIUS does not exceed a precision value of 0.51, DreaMS achieves a precision of 0.91 with a recall of 0.57: "Whereas SIRIUS does not exceed a precision value of 0.51, DreaMS achieves a precision of 0.91 with a recall of 0.57"
- [abstract] the standard training spectral libraries MoNA36 and NIST20 (ref. 37) cover only a limited subset of known natural molecules: "the standard training spectral libraries MoNA36 and NIST20 (ref. 37) cover only a limited subset of known natural molecules"
- [discussion] the full potential of this approach remains to be unlocked: "the full potential of this approach remains to be unlocked"
