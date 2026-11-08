---
name: modular-metabolite-class-annotation
description: Use when predicting ESI-MS/MS spectra or identifying compounds belonging
  to modular chemical classes (acylcarnitines, acylcholines, flavonoids, lipids) where
  class-specific hand-written SMIRKS fragmentation rules should be routed via ClassyFire
  annotations.
when_to_use_negative:
- Compound belongs to none of the 11 covered modular classes — it will fall through
  to the generic machine-learned CFM predictor rather than benefiting from MSRB rules
- Spectra were collected on non-QTOF instruments; the rule-derived relative intensities
  are scaled from QTOF ESI-MS/MS data at 10, 20, and 40 eV only
- Input is already a predicted-spectrum or feature table without a resolved chemical
  structure, since ClassyFire routing requires the structure itself
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 4.0
  role: Applies ClassyFire-routed SMIRKS-based (MSRB) rules plus machine-learned CFM
    prediction to generate class-aware ESI-MS/MS spectra
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Assigns the chemical class label that routes structures to the suitable MSRB
    fragmentation rules
- name: msrb-fragmenter
  role: Standalone source for the SMIRKS-based rule library; rules also listed in
    article Supplemental Table S1
  repo: https://bitbucket.org/wishartlab/msrb-fragmenter
- name: Docker
  role: Deployment container for running CFM-ID 4.0 locally
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:e9b5cf8c7b857120e589ba7f1e1174c8d25a9d8fac41d0932b1c0cedac6eded3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - ClassyFire
  - Docker
  - msrb-fragmenter
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# modular-metabolite-class-annotation

## Summary

Annotates metabolite classes using CFM-ID 4.0's ClassyFire-driven routing to select hand-written SMIRKS-based (MSRB) fragmentation rules — 313 rules spanning 11 chemical classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, lipids) and 3 adduct types — for class-aware ESI-MS/MS spectral prediction.

## When to use

Use when predicting ESI-MS/MS spectra or identifying compounds belonging to modular chemical classes (acylcarnitines, acylcholines, flavonoids, lipids) where class-specific hand-written SMIRKS fragmentation rules should be routed via ClassyFire annotations.

## When NOT to use

- Compound belongs to none of the 11 covered modular classes — it will fall through to the generic machine-learned CFM predictor rather than benefiting from MSRB rules
- Spectra were collected on non-QTOF instruments; the rule-derived relative intensities are scaled from QTOF ESI-MS/MS data at 10, 20, and 40 eV only
- Input is already a predicted-spectrum or feature table without a resolved chemical structure, since ClassyFire routing requires the structure itself

## Inputs

- Candidate compound structure (SMILES/InChI) for ClassyFire class assignment
- Target adduct type / precursor ion specification
- SMIRKS-based rule library from the msrb-fragmenter repository or Supplemental Table S1 (313 rules, 11 classes, 3 adduct types)

## Outputs

- Class-routed MSRB predicted ESI-MS/MS spectrum with peaks scaled by rule-derived average relative intensities (at 10/20/40 eV collision energies)
- Verified rule table mapping each SMIRKS pattern to chemical class, adduct type, and relative intensity (e.g. msrb_rules_table.csv)

## How to apply

First run ClassyFire on the candidate structure to assign its chemical class, since CFM-ID 4.0 uses ClassyFire to identify the specific classes suitable for MSRB (rule-based) fragmentation versus the learned CFM model. If the class is one of the 11 covered modular classes, retrieve the corresponding SMIRKS fragmentation rules from the msrb-fragmenter source or Supplemental Table S1, each rule encoding a fragmentation reaction pattern plus average scaled relative intensities derived from QTOF spectra at 10, 20, and 40 eV collision energies. Route the rule selection by chemical class and adduct type, then apply the matched SMIRKS reactions to generate predicted fragment peaks with rule-derived scaled intensities. Verify the assembled rule set totals 313 hand-made MSRB rules and that class routing matches the 11 chemical classes and 3 adduct types reported in the paper.

## Related tools

- **CFM-ID 4.0** (Applies ClassyFire-routed SMIRKS-based (MSRB) rules plus machine-learned CFM prediction to generate class-aware ESI-MS/MS spectra) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Assigns the chemical class label that routes structures to the suitable MSRB fragmentation rules)
- **msrb-fragmenter** (Standalone source for the SMIRKS-based rule library; rules also listed in article Supplemental Table S1) — https://bitbucket.org/wishartlab/msrb-fragmenter
- **Docker** (Deployment container for running CFM-ID 4.0 locally) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker pull wishartlab/cfmid && docker run wishartlab/cfmid cfm-id predict-msms --input compound.smiles --method msrb --collision-energies 10,20,40 --output predicted_spectrum.msp
```

## Evaluation signals

- Compiled MSRB rule count equals 313 hand-made rules covering 11 chemical classes and 3 adduct types
- Each rule in the output table carries a SMIRKS pattern, a ClassyFire chemical class label, an adduct type, and an average scaled relative intensity
- Relative intensities in rule entries correspond to those scaled from experimental QTOF spectra at 10, 20, and 40 eV collision energies
- Class routing is consistent: structures not in a covered class are not routed to MSRB rules
- Predicted spectra for covered-class compounds improve match against experimental Metlin QTOF reference spectra relative to CFM-ID 3.0

## Limitations

- Rule-based MSRB prediction is limited to 11 chemical classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides, lipids); other classes fall back to generic prediction
- Relative intensities are calibrated on QTOF instruments only; CFM-ID 4.0 does not yet handle spectra from instruments other than QTOF
- Rule intensities are averages (scaled relative intensities), so compound-specific deviation from average behavior is not captured

## Evidence

- [supplementary] CFM-ID 4.0's extended SMIRKS-based fragmentation rules cover 11 chemical classes and 3 adduct types, generating a total of 313 hand-made MSRB rules: [evidence span withheld — anchor status: not_found]
