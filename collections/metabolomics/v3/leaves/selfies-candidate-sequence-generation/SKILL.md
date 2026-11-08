---
name: selfies-candidate-sequence-generation
description: 'Use when training the alignment stage of MS-BART: after fingerprint->molecule
  finetuning (Pretrain-FT), you need a pool of sampled SELFIES candidates per fingerprint
  input to score preferences against the ground-truth structure.'
when_to_use_negative:
- No fine-tuned fingerprint->molecule checkpoint exists yet — candidates sampled from
  the raw pretrained model will not reflect experimental-spectrum adaptation; run
  the Pretrain-FT stage first
- No ground-truth structure or fingerprint prediction is available (e.g., purely de
  novo generation without MIST fingerprint conditioning) — preference scoring Ps(Si)
  requires a reference structure to compute Tanimoto similarity
- Input spectra contain [M+Na]+ adducts or collision-energy/adduct variability that
  was not thresholded and filtered as in finetuning — candidate quality will degrade
  on out-of-distribution conditioning tokens
edam_topics:
- http://edamontology.org/topic_3364
- http://edamontology.org/topic_3172
tools:
- name: MS-BART
  role: Fingerprint-conditioned seq2seq model (BART-Base backbone) whose decoder samples
    the candidate SELFIES sequences during the alignment stage
  repo: github.com/OpenDFM/MS-BART
- name: SELFIES
  role: String-based molecular representation in which candidates are generated, tokenized,
    and validated before decoding to SMILES
- name: RDKit
  role: Used to compute fingerprints from decoded candidate molecules and ground-truth
    structures for validity checks and Tanimoto preference scoring
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:5dae204bae0b8d07bc791cb4d6f210e21aa2c63e302685e5dd91f9ed4780503c
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MS-BART
  - RDKit
  - SELFIES
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3364
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# selfies-candidate-sequence-generation

## Summary

Generate n candidate molecular structures as SELFIES sequences from a fingerprint-conditioned seq2seq model (MS-BART on a BART-Base backbone), producing a candidate pool with sequence log-probabilities that feeds chemical-feedback alignment via Tanimoto/formula-distance preference scoring.

## When to use

Use when training the alignment stage of MS-BART: after fingerprint->molecule finetuning (Pretrain-FT), you need a pool of sampled SELFIES candidates per fingerprint input to score preferences against the ground-truth structure.

## When NOT to use

- No fine-tuned fingerprint->molecule checkpoint exists yet — candidates sampled from the raw pretrained model will not reflect experimental-spectrum adaptation; run the Pretrain-FT stage first
- No ground-truth structure or fingerprint prediction is available (e.g., purely de novo generation without MIST fingerprint conditioning) — preference scoring Ps(Si) requires a reference structure to compute Tanimoto similarity
- Input spectra contain [M+Na]+ adducts or collision-energy/adduct variability that was not thresholded and filtered as in finetuning — candidate quality will degrade on out-of-distribution conditioning tokens

## Inputs

- Thresholded fingerprint token sequences (active bits of MIST-predicted 4096-bit circular Morgan fingerprint FP) from the MassSpecGym/NPLIB1 fine-tuning splits
- Fine-tuned MS-BART checkpoint from the Pretrain-FT stage (BART-Base backbone)
- Ground-truth molecular structures (SELFIES/SMILES) of the finetuning examples, for candidate preference scoring
- Candidate pool size n and decoding hyperparameters (beam-search multinomial sampling settings)
- RDKit Morgan fingerprint parameters (radius 2) and rank-loss parameters (margin gamma, loss weight alpha)

## Outputs

- n candidate SELFIES sequences per fingerprint input, each with its sequence log-probability
- Validity-filtered candidate set decoded to RDKit Mol objects
- Ranked candidate table with per-candidate Tanimoto similarity (radius-2 Morgan, 2048-bit) to the ground-truth structure and molecular-formula distance
- Aligned checkpoint saved under logs/<run-name> after alignment training on this candidate pool

## How to apply

Load the fine-tuned MS-BART checkpoint (Pretrain-FT stage, BART-Base backbone) and the thresholded fingerprint–SELFIES finetuning data (bit threshold epsilon = 0.2 for MassSpecGym, [M+Na]+ adducts filtered out, only [M+H]+ retained). Represent each spectrum as the active-bit tokens of a 4096-bit circular Morgan fingerprint FP predicted by MIST (conditioned on metadata and chemical formula), then generate n candidate SELFIES sequences conditioned on fingerprint tokens FP using beam-search multinomial sampling from the decoder, recording each candidate's sequence log-probability. Decode every sampled SELFIES string with the SELFIES library and verify validity in RDKit, discarding candidates that fail sanitization. Score each valid candidate's preference Ps(Si) as Tanimoto similarity to the ground-truth structure using RDKit Morgan fingerprints (radius 2, 2048-bit hashing), and rank candidates by molecular-formula distance (sum of per-atom count differences) with log-probability as tie-breaker — this ranked pool is the input for the contrastive rank loss L_rank of the alignment objective L = L_ce + alpha*L_rank. Rationale: multiple candidates are needed because greedy decoding hallucinates molecules; alignment pushes probability mass toward higher-Tanimoto candidates via rank-scaled margins gamma_ij = (j-i)*gamma while freezing the encoder and updating only the decoder.

## Related tools

- **MS-BART** (Fingerprint-conditioned seq2seq model (BART-Base backbone) whose decoder samples the candidate SELFIES sequences during the alignment stage) — github.com/OpenDFM/MS-BART
- **SELFIES** (String-based molecular representation in which candidates are generated, tokenized, and validated before decoding to SMILES)
- **RDKit** (Used to compute fingerprints from decoded candidate molecules and ground-truth structures for validity checks and Tanimoto preference scoring)

## Evaluation signals

- Every sampled SELFIES string decodes without error and yields an RDKit-sanitizable molecule; invalid candidates are logged and dropped
- Candidate pool diversity: candidates differ in Tanimoto to the ground truth so that rank-scaled margins gamma_ij = (j-i)*gamma are well defined (non-degenerate ranking)
- Alignment training on these candidates raises validation Top-1/Top-10 accuracy, Top-1/Top-10 Tanimoto, and lowers MCES relative to the Pretrain-FT stage (Fig. 3 Stage 3)
- Sum of per-atom count differences (formula distance) between top-ranked candidate and ground truth decreases after alignment on validation split

## Limitations

- Candidate quality depends on the externally pretrained MIST fingerprint predictor; its prediction accuracy significantly impacts overall performance
- Formula information is used only for re-ranking candidates, not for training the model — incorporating it is deferred to future work
- Restricted benchmarks (e.g., NIST23, commercial license) and adduct filtering ([M+Na]+ removed) mean the candidate pool reflects only [M+H]+, thresholded finetuning conditions

## Evidence

- [methods] Generate SELFIES sequences conditioned on fingerprint tokens FP.: [evidence span withheld — anchor status: not_found]
- [methods] we represent each spectrum as a 4096-bit circular Morgan fingerprint FP∈{0,1}4096: [evidence span withheld — anchor status: not_found]
- [methods] we filter out [M+Na]+ adducts before fine-tuning and alignment, retaining only the dominant [M+H]+ data.: [evidence span withheld — anchor status: not_found]
