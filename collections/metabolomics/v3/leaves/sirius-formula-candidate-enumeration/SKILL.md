---
name: sirius-formula-candidate-enumeration
description: Use when you have an exact precursor mass from MS/MS data and need a
  database-independent candidate formula set (RDBE-plausible, C/H/N/O/P/S/Cl/Br/F
  elemental space) to score and re-rank with a downstream model.
when_to_use_negative:
- When you already have a small candidate set from a spectral database match (e.g.
  NIST, GNPS, HMDB, PubChem) — enumeration is for de novo, database-independent settings.
- 'For negative-mode data: the article''s models were trained only on positive-mode
  spectra, so candidate scoring downstream is out of scope even if enumeration itself
  runs.'
- If you need SIRIUS's full fragmentation-tree scoring (CSI:FingerID, CANOPUS) rather
  than just the candidate list — this skill isolates only the `decomp` module.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: Provides the dynamic-programming `decomp` module used to enumerate all RDBE-permissible
    formula candidates for an observed MS1 mass
  repo: https://github.com/boecker-lab/sirius
- name: FastFilter
  role: Neural network that down-selects the enumerated candidates to a top-256 subset
    retaining the true formula ~99% of the time
  repo: https://github.com/samgoldman97/mist-cf
- name: MIST-CF
  role: Downstream energy-based model that re-ranks the enumerated (and FastFilter-filtered)
    candidate formulae against the MS/MS spectrum
  repo: https://github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_003
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:b556a7506943c8b5b7adc605f5c988fb36445abd1addb4fbfde733d8466a66a9
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - Chemical Formula Transformer
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# sirius-formula-candidate-enumeration

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Enumerates all plausible elemental formula candidates for an observed MS1 precursor mass using SIRIUS's dynamic-programming `decomp` module, providing the candidate pool that downstream learned scorers like MIST-CF and FastFilter rank.

## When to use

Use when you have an exact precursor mass from MS/MS data and need a database-independent candidate formula set (RDBE-plausible, C/H/N/O/P/S/Cl/Br/F elemental space) to score and re-rank with a downstream model.

## When NOT to use

- When you already have a small candidate set from a spectral database match (e.g. NIST, GNPS, HMDB, PubChem) — enumeration is for de novo, database-independent settings.
- For negative-mode data: the article's models were trained only on positive-mode spectra, so candidate scoring downstream is out of scope even if enumeration itself runs.
- If you need SIRIUS's full fragmentation-tree scoring (CSI:FingerID, CANOPUS) rather than just the candidate list — this skill isolates only the `decomp` module.

## Inputs

- Observed MS1 precursor mass (exact mass) from each tandem mass spectrum
- Elemental constraints/RDBE plausibility settings governing which formulae are permitted
- SIRIUS headless executable (v5.5.7 linux64) with SIRIUS_PATH exported

## Outputs

- Per-spectrum list of RDBE-permissive chemical formula candidates (often >5,000 entries for ~15% of training spectra)
- Candidate sets suitable for downstream FastFilter top-256 down-selection and MIST-CF energy scoring

## How to apply

Download the SIRIUS headless build (v5.5.7 linux64 per the MIST-CF instructions) and export the SIRIUS_PATH environment variable so its command-line `decomp` module is callable. For each spectrum, feed the observed MS1 mass into `SIRIUS decomp`, which applies a dynamic-programming decomposition over the elemental alphabet with relative-database-equivalent (RDBE) plausibility constraints — the article notes over 15% of NPLIB1/NIST20 training spectra yield more than 5,000 RDBE-permissive candidates, so the raw enumeration can be large. The candidate set is then handed to FastFilter (a lightweight neural energy model trained via `run_scripts/public_data_train/train_fast_filter.sh` with `configs/fast_filter.yaml`) which recovers the true formula within its top 256 candidates 99% of the time, keeping the pipeline tractable for MIST-CF scoring. Because MIST-CF still relies on SIRIUS for this step ('we still rely upon SIRIUS's algorithmic decomposition of exact masses into formula candidates due to their fast implementation'), the enumeration step is a prerequisite for all re-ranking skills in this article's workflow.

## Related tools

- **SIRIUS** (Provides the dynamic-programming `decomp` module used to enumerate all RDBE-permissible formula candidates for an observed MS1 mass) — https://github.com/boecker-lab/sirius
- **FastFilter** (Neural network that down-selects the enumerated candidates to a top-256 subset retaining the true formula ~99% of the time) — https://github.com/samgoldman97/mist-cf
- **MIST-CF** (Downstream energy-based model that re-ranks the enumerated (and FastFilter-filtered) candidate formulae against the MS/MS spectrum) — https://github.com/samgoldman97/mist-cf

## Examples

```
export SIRIUS_PATH=/path/to/sirius-5.5.7/bin/sirius; python src/mist_cf/fast_form_score/predict.py --input data/canopus_train --sirius-path $SIRIUS_PATH
```

## Evaluation signals

- FastFilter applied to the enumerated candidates retains the true formula within its top 256 for ~99% of training spectra (NPLIB1 and NIST20); a large drop indicates the enumeration or filter step was misconfigured.
- The pre-filter candidate-count distribution should show that >15% of training spectra yield >5,000 RDBE-permissive candidates; a near-empty or uniform-size candidate set suggests the RDBE/plausibility constraints were not applied as intended.
- Downstream MIST-CF top-1 accuracy on held-out NPLIB1 should reach ~74% (NPLIB1-only) or ~77% (NIST20-augmented); substantially lower values suggest the candidate pool was truncated before FastFilter down-selection.
- SIRIUS headless invocation succeeds when SIRIUS_PATH is set and the downloaded build matches the expected version (v5.5.7 headless linux64); failure to locate the binary or wrong version breaks all subsequent scoring steps.

## Limitations

- The enumeration still depends on SIRIUS's algorithmic decomposition — MIST-CF does not replace this step, so any SIRIUS availability/version issues propagate downstream.
- The article does not address negative-mode training, so candidate scoring for negative-mode spectra is unsupported by the downstream models in this workflow.
- The raw candidate lists can be very large (over 5,000 per spectrum in >15% of cases), making downstream scoring slow unless FastFilter is applied first.

## Evidence

- [discussion] SIRIUS decomp enumerates all RDBE-permissive formula candidates from exact precursor mass: [evidence span withheld — anchor status: not_found]
- [results] >15% of training spectra have more than 5,000 RDBE-permissive formula candidates: [evidence span withheld — anchor status: not_found]
- [methods] FastFilter down-selection to top 256 candidates is required before MIST-CF scoring: [evidence span withheld — anchor status: not_found]
