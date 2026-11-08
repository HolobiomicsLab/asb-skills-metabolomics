---
name: metabolite-mass-filtering-by-exact-mass-tolerance
description: Use when you have only an unknown metabolite's exact mass (no MS/MS spectrum)
  and need to prioritize which DeepMet-sampled candidate structure most likely accounts
  for the observed mass spectrometric signal.
when_to_use_negative:
- You already have MS/MS spectral data — the paper states that in that regime spectral
  matching and meta-learning integration should be used instead, and mass-only filtering
  alone is less informative.
- The unknown's mass was measured at accuracy far worse than ~10 ppm — the ±10 ppm
  window is the paper's operating point and a looser window will flood the candidate
  set.
- You are annotating metabolites from evolutionarily distant organisms (plants, bacteria)
  — the model generating the candidate database was trained only on human metabolite
  structures, so the candidate pool itself is out of domain.
- The candidate structure is not in the sampled database at all — DeepMet has inherent
  limits anticipating metabolites from divergent, undiscovered biosynthetic routes,
  and mass filtering cannot recover an unsampled structure.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0219
tools:
- name: DeepMet
  role: Generates the 34.4-million-structure database with per-structure sampling
    frequencies that is the pool being mass-filtered
  repo: https://github.com/skinniderlab/CLM
- name: RDKit
  role: Computes exact monoisotopic mass for both the unknown and each candidate structure
    to enforce the ±10 ppm window
- name: Snakemake
  role: Runs the collect_tabulated_molecules / process_tabulated_molecules workflow
    that produces the tabulated mass/formula/frequency input file
  repo: https://github.com/skinniderlab/CLM
- name: AddCarbon
  role: Baseline generator whose database is filtered with the same ±10 ppm window
    to benchmark DeepMet's uplift
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:982132f05f6655c679e2d69a1e44736714da7ea2f1585afb8a816e6e3190652a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - AddCarbon
  - DeepMet
  - PyTorch
  - Python
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# metabolite-mass-filtering-by-exact-mass-tolerance

## Summary

Filters a chemical language model's generated-structure database by exact monoisotopic mass within a narrow ppm tolerance of an unidentified metabolite's mass, then ranks surviving candidates by sampling frequency so that the most frequently sampled structure becomes the top-1 annotation prior.

## When to use

Use when you have only an unknown metabolite's exact mass (no MS/MS spectrum) and need to prioritize which DeepMet-sampled candidate structure most likely accounts for the observed mass spectrometric signal.

## When NOT to use

- You already have MS/MS spectral data — the paper states that in that regime spectral matching and meta-learning integration should be used instead, and mass-only filtering alone is less informative.
- The unknown's mass was measured at accuracy far worse than ~10 ppm — the ±10 ppm window is the paper's operating point and a looser window will flood the candidate set.
- You are annotating metabolites from evolutionarily distant organisms (plants, bacteria) — the model generating the candidate database was trained only on human metabolite structures, so the candidate pool itself is out of domain.
- The candidate structure is not in the sampled database at all — DeepMet has inherent limits anticipating metabolites from divergent, undiscovered biosynthetic routes, and mass filtering cannot recover an unsampled structure.

## Inputs

- Unknown metabolite's exact monoisotopic mass (single value or m/z list)
- Generated-structure database with per-structure molecular mass, molecular formula, and sampling frequency (from collect_tabulated_molecules / process_tabulated_molecules)
- AddCarbon-generated structure database with identical fields, for baseline comparison
- RDKit-validated SMILES or InChIKey identifiers for each candidate structure

## Outputs

- Ranked list of mass-matched candidate structures per unknown, sorted by decreasing sampling frequency
- Per-annotation confidence score derived from sampling frequency
- Top-1 / top-3 / top-10 accuracy metrics (reported: ~29% / ~51% / ~72% cross-validation; ~22.9% / ~61.7% prospective HMDB v5.0)
- Calibration-bin statistics for correct vs incorrect annotations
- evaluation_results.csv with accuracies, rank distributions, and AddCarbon-vs-DeepMet comparisons

## How to apply

Use RDKit to compute the exact monoisotopic mass of the target unknown and compute the same mass for every structure in the generated-structure database produced by the clm Snakemake pipeline's collect_tabulated_molecules / process_tabulated_molecules steps, which tabulate molecular mass, formula, and sampling frequency per structure. Retain only candidates whose exact mass falls within a ±10 ppm tolerance of the target mass — mass-to-charge-based filtering is the sole analytical constraint in this mode, so the ppm window is the key threshold. Sort the retained candidates in decreasing order of sampling frequency and record the rank at which the true structure appears; the sampling frequency doubles as a confidence score, which the paper shows is well-calibrated when binned for correct vs incorrect annotations. For prospective unknowns, the same procedure without a known answer yields a structural prior that can be fed to downstream MS/MS meta-learning. Compare against the AddCarbon baseline (carbon-atom insertion database) using identical mass windows to gauge uplift.

## Related tools

- **DeepMet** (Generates the 34.4-million-structure database with per-structure sampling frequencies that is the pool being mass-filtered) — https://github.com/skinniderlab/CLM
- **RDKit** (Computes exact monoisotopic mass for both the unknown and each candidate structure to enforce the ±10 ppm window)
- **Snakemake** (Runs the collect_tabulated_molecules / process_tabulated_molecules workflow that produces the tabulated mass/formula/frequency input file) — https://github.com/skinniderlab/CLM
- **AddCarbon** (Baseline generator whose database is filtered with the same ±10 ppm window to benchmark DeepMet's uplift)

## Examples

```
python -c "from rdkit import Chem; from rdkit.Chem import Descriptors; m=Chem.MolFromSmiles('OC(=O)CC'); mws=[(s,f) for s,f in candidates.items() if abs(Descriptors.ExactMolWt(Chem.MolFromSmiles(s))-target)/target<1e-5]; mws.sort(key=lambda x:-x[1])"
```

## Evaluation signals

- Cross-validation top-1 accuracy lands at approximately 29%, top-3 at ~51%, and top-10 at ~72% when the known answer is hidden from the ranked list.
- Prospective HMDB v5.0 set yields top-1 ≈ 22.9% and top-10 ≈ 61.7%.
- For each unknown, the exact monoisotopic mass of every retained candidate lies within ±10 ppm of the target mass (verify via RDKit re-computation).
- Binning sampling-frequency-derived confidence for correct vs incorrect annotations yields monotonic, well-calibrated bins.
- DeepMet's rank distribution for the correct structure outperforms the AddCarbon baseline under identical mass windows and frequency ranking.

## Limitations

- Only a small fraction of peaks detected in metabolomics can typically be identified, and structure elucidation of novel metabolites remains a low-throughput endeavor.
- Model trained and evaluated only on human metabolite structures; evolutionarily distant applications (plant, bacterial metabolism) will likely require bespoke models.
- Inherent limitations for anticipating metabolites originating from divergent, as-of-yet undiscovered biosynthetic routes.
- Predicting MS/MS spectra or retention times from chemical structures remain open problems, so mass-only prioritization cannot be refined without external spectral models.
- Only a small fraction of peaks can be identified without additional analytical data; opportunity to more tightly integrate DeepMet with analytical data not yet realized.

## Evidence
