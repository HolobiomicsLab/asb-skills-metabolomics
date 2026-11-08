---
name: beam-search-molecular-generation-decoding
description: Use when evaluating a spectra-to-structure model (e.g., MS-Bart fine-tuned
  on MassSpecGym [M+H]+ data) on a test fold and you need Top-1/Top-10 exact-match
  (full InChIKey), minimum MCES, and maximum Tanimoto similarity scores per generated
  candidate.
when_to_use_negative:
- When evaluating diffusion-based competitors (e.g., FRIGID, DiffMS) that generate
  via masked-diffusion denoising rather than beam-search decoding — use their own
  sampling procedures.
- When evaluating on a training/val fold where [M+Na]+ adducts were filtered out,
  since the evaluation protocol explicitly retains all adducts on the full test fold.
- When ground-truth full InChIKeys are unavailable, since Top-1/Top-10 exact-match
  accuracy cannot be computed as specified.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: MS-Bart
  role: Aligned generative model decoded with beam-search multinomial sampling to
    produce candidate molecular structures from spectrum fingerprints
  repo: github.com/OpenDFM/MS-BART
- name: RDKit
  role: Computes Morgan fingerprints (radius 2, 2048 bits) and Tanimoto similarity
    between generated and ground-truth molecules during evaluation
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:e58fb4bfc3b9c11e57a290f3a4e5c79396a4067ec0eda39e62981d96e8d76b12
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BART-Base
  - MS-BART
  - MS-Bart
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9615384615384616
n_claims: 26
---

# beam-search-molecular-generation-decoding

## Summary

Decodes molecular structures (as SELFIES/SMILES) from a fine-tuned MS-Bart checkpoint using beam-search multinomial sampling, then scores candidates against ground-truth structures with Top-1/Top-10 InChIKey accuracy, MCES, and Tanimoto metrics.

## When to use

Use when evaluating a spectra-to-structure model (e.g., MS-Bart fine-tuned on MassSpecGym [M+H]+ data) on a test fold and you need Top-1/Top-10 exact-match (full InChIKey), minimum MCES, and maximum Tanimoto similarity scores per generated candidate.

## When NOT to use

- When evaluating diffusion-based competitors (e.g., FRIGID, DiffMS) that generate via masked-diffusion denoising rather than beam-search decoding — use their own sampling procedures.
- When evaluating on a training/val fold where [M+Na]+ adducts were filtered out, since the evaluation protocol explicitly retains all adducts on the full test fold.
- When ground-truth full InChIKeys are unavailable, since Top-1/Top-10 exact-match accuracy cannot be computed as specified.

## Inputs

- Fine-tuned and chemically aligned MS-Bart checkpoint (BART-Base backbone)
- Preprocessed MassSpecGym or NPLIB1 test fold (MIST 4096-bit Morgan fingerprint inputs, all adducts)
- Ground-truth molecular structures with full InChIKey identifiers

## Outputs

- Top-1/Top-10 exact-match accuracy (full InChIKey)
- Top-1/Top-10 minimum MCES distance
- Top-1/Top-10 maximum Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits, RDKit)
- Per-epsilon comparison table (epsilon_sensitivity_results.csv)

## How to apply

Run the aligned model on the full test fold with beam-search multinomial sampling via scripts/msg/eval.sh, retaining all adducts in the test set (unlike fine-tuning, which filters [M+Na]+ and keeps only [M+H]+). For each decoded candidate, compute exact-match accuracy using the full InChIKey, minimum MCES distance, and maximum Tanimoto similarity against the ground-truth structure. Tanimoto is computed from Morgan fingerprints with radius 2, 2048 bits, using RDKit, mirroring the fingerprint representation (4096-bit Morgan) used to encode spectra. Aggregate per-threshold or per-model metrics into a single comparison table (e.g., epsilon_sensitivity_results.csv) ordered by threshold value to expose sensitivity (MS-Bart was found insensitive to the fingerprint threshold epsilon, with Top-1 Tanimoto approximately 0.23 across epsilon in {0.10, 0.11, 0.15, 0.20}).

## Related tools

- **MS-Bart** (Aligned generative model decoded with beam-search multinomial sampling to produce candidate molecular structures from spectrum fingerprints) — github.com/OpenDFM/MS-BART
- **RDKit** (Computes Morgan fingerprints (radius 2, 2048 bits) and Tanimoto similarity between generated and ground-truth molecules during evaluation)

## Examples

```
bash scripts/msg/eval.sh
```

## Evaluation signals

- Top-1 Tanimoto similarity should be approximately 0.23 across tested epsilon values on MassSpecGym, with epsilon=0.11 giving Top-1 accuracy 1.07% and Top-1 MCES 16.47.
- Metrics table must contain Top-1/Top-10 exact-match accuracy (full InChIKey), minimum MCES, and maximum Tanimoto columns ordered by threshold value.
- Evaluation must cover the full MassSpecGym test fold with all adducts retained, matching the reported benchmark setup (SOTA on 5/12 key metrics).
- Tanimoto values must derive from Morgan fingerprints with radius 2 and 2048 bits computed via RDKit, consistent with the model's fingerprint representation.

## Limitations

- The accuracy of upstream MIST fingerprint predictions significantly impacts the overall performance of the decoded structures.
- Formula information is used only for re-ranking and not for training, so decoding does not exploit formula constraints directly.
- No software/tool versions are reported anywhere in the discussion/limitations text, so exact reproducibility of decoding behavior is uncertain.

## Evidence

- [other] Evaluate each aligned model on the full MassSpecGym test fold (all adducts retained) with beam-search multinomial sampling via scripts/msg/eval.sh: [evidence span withheld — anchor status: not_found]
- [abstract] MS-BART achieves SOTA performance across 5/12 key metrics on MassSpecGym and NPLIB1 and is faster by one order of magnitude than competing diffusion-based methods: "MS-BART achieves SOTA performance across 5/12 key metrics on MassSpecGym and NPLIB1 and is faster by one order of magnitude than competing diffusion-based methods"
- [other] maximum Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits, RDKit): [evidence span withheld — anchor status: not_found]
- [methods] we filter out [M+Na]+ adducts before fine-tuning and alignment, retaining only the dominant [M+H]+ data: [evidence span withheld — anchor status: not_found]
