---
name: classyfire-superclass-annotation-analysis
description: Use when evaluating formula-prediction accuracy across molecular strata
  (polarity/LogP, mass, ClassyFire superclass) on tandem MS test predictions to flag
  underperforming chemical classes.
when_to_use_negative:
- Input spectra are chimeric (co-eluting) — FIDDLE performance degrades with spectral
  mixing, so stratified accuracy would not reflect class-specific effects
- Molecules fall outside FIDDLE's training scope (e.g., atoms outside C, H, O, N,
  F, S, Cl, P, B, I, Br, Na, K, atom counts above 300, or m/z above 1500)
- The goal is de novo structure elucidation rather than benchmarking formula-identification
  accuracy against known ground truth
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: FIDDLE
  role: Deep learning model whose top-5 formula predictions on Q-TOF and Orbitrap
    NIST23 test spectra are stratified and scored
  repo: https://github.com/JosieHong/FIDDLE
- name: Q-TOF
  role: Instrument type for which accuracy is computed per stratum, with 10 ppm precursor
    mass tolerance
- name: Orbitrap
  role: Instrument type for which accuracy is computed per stratum, with 5 ppm precursor
    mass tolerance
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:426c6514253ecc9678f6d8e935c10bb033fa4267f404ea696bdec0a4286c4ad9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - FIDDLE
  - Orbitrap
  - Q-TOF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# classyfire-superclass-annotation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies FIDDLE formula-identification accuracy on held-out NIST23 test spectra by molecular properties — ClassyFire superclass, LogP polarity bins, and precursor mass bins — for Q-TOF and Orbitrap instruments, to reveal where the deep learning model underperforms.

## When to use

Use when evaluating formula-prediction accuracy across molecular strata (polarity/LogP, mass, ClassyFire superclass) on tandem MS test predictions to flag underperforming chemical classes.

## When NOT to use

- Input spectra are chimeric (co-eluting) — FIDDLE performance degrades with spectral mixing, so stratified accuracy would not reflect class-specific effects
- Molecules fall outside FIDDLE's training scope (e.g., atoms outside C, H, O, N, F, S, Cl, P, B, I, Br, Na, K, atom counts above 300, or m/z above 1500)
- The goal is de novo structure elucidation rather than benchmarking formula-identification accuracy against known ground truth

## Inputs

- Per-spectrum FIDDLE prediction CSV from the NIST23 test-set evaluation (top-5 formula candidates and confidence scores)
- Ground-truth molecular formulas for test spectra
- Canonical SMILES-derived LogP values and ClassyFire superclass annotations for test compounds
- Instrument-type labels (Q-TOF vs Orbitrap) and precursor m/z values per spectrum

## Outputs

- Stratified top-5 accuracy table per LogP bin, mass bin, and ClassyFire superclass
- Fig. 3c/d/f-style summary panels (inverse LogP trend, mass-bin profile, superclass comparison)
- List of underperforming superclasses (organohalogens, polymers, organosulfur, unknown)

## How to apply

Run the released FIDDLE TCN and rescore checkpoints (Q-TOF and Orbitrap variants) over held-out NIST23 test spectra via experiments_test_nist23.sh, writing per-spectrum top-5 predictions to a results CSV. Join each spectrum's ground-truth formula with molecular annotations (canonical SMILES-derived LogP and ClassyFire superclass). Bin spectra by LogP polarity group, by precursor mass (e.g., <200 Da vs larger bins), and by ClassyFire superclass, then compute top-5 accuracy per stratum for both Q-TOF and Orbitrap. Generate Fig. 3c/d/f-style panels showing the inverse LogP trend (polar 95.0% vs nonpolar 88.8%, a 6.2% gap) and the superclass comparison, flagging underperformers such as organohalogens, polymers, organosulfur, and unknown. Validate against reported reference values (e.g., 97.3% Q-TOF and 95.0% Orbitrap top-5 accuracy for <200 Da) before trusting the stratified tables.

## Related tools

- **FIDDLE** (Deep learning model whose top-5 formula predictions on Q-TOF and Orbitrap NIST23 test spectra are stratified and scored) — https://github.com/JosieHong/FIDDLE
- **Q-TOF** (Instrument type for which accuracy is computed per stratum, with 10 ppm precursor mass tolerance)
- **Orbitrap** (Instrument type for which accuracy is computed per stratum, with 5 ppm precursor mass tolerance)

## Examples

```
bash experiments_test_nist23.sh --instrument qtof --model tcn_rescore; python stratify_accuracy.py --predictions results.csv --annotations smiles_logp_classyfire.tsv --bins logp,mass,classyfire
```

## Evaluation signals

- Stratified table reproduces reported <200 Da top-5 accuracy: 97.3% (Q-TOF) and 95.0% (Orbitrap)
- LogP trend is monotonically inverse: polar compounds ~95.0%, moderate ~94.7%, nonpolar ~88.8% (a 6.2% gap), consistent across both instrument types
- Underperforming superclass list matches the paper's: organohalogens, polymers, organosulfur, unknown
- Per-spectrum prediction CSV row counts match the number of filtered NIST23 test spectra (≥5 peaks, m/z ≤1500, ≤300 atoms, common precursor adducts)
- Overall pooled top-5 accuracy recovers the paper's reported 93.6% evaluation-set value before stratification

## Limitations

- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases, which can confound superclass-level comparisons
- Experimental condition effect is not completely eliminated; accuracy varies across conditions, especially those with less training data
- Mass bins above <200 Da and superclasses with sparse test-set coverage will have high-variance accuracy estimates

## Evidence
