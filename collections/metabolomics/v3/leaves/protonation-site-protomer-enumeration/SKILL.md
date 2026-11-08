---
name: protonation-site-protomer-enumeration
description: 'Use when preparing a small molecule (<205 Da or larger) for CID-MS/MS
  spectrum prediction from structure alone, and the exact protonation site in ESI
  is unknown: enumerate every candidate protomer/tautomer at heteroatom sites and
  simulate each independently before comparing to the experimental.'
when_to_use_negative:
- The experimental spectrum is already known and only a single dominant protomer was
  confirmed by prior orthogonal evidence—enumerating all sites adds cost without improving
  the prediction.
- The molecule's protonation state under ESI is known unambiguously (e.g., permanent
  quaternary ammonium) and there are no alternative heteroatom sites—run a single
  CIDMD simulation instead.
- Molecules with very large numbers of protomers/tautomers where the article warns
  average CIDMD prediction accuracy decreases as protomer count grows, making exhaustive
  enumeration computationally prohibitive.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: Avogadro
  role: Manually construct each candidate protonated molecular ion structure at its
    protonation site
- name: TeraChem
  role: Optimize the 3D shape of each enumerated protomer with unrestricted B3LYP/6–31G*
    before CIDMD
- name: CIDMD_setup
  role: Generate collision input files (start.xyz, vel.xyz) per protomer structure
  repo: https://github.com/jesilee/CIDMD_setup
- name: CIDMD_compare
  role: Score each protomer's predicted spectrum against the reference experimental
    spectrum to rank protomers
  repo: https://github.com/jesilee/CIDMD_compare
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jcim.4c00760
    title: Predicting Collision-Induced-Dissociation Tandem Mass Spectra (CID-MS/MS)
      Using Ab Initio Molecular Dynamics
  source_content_hash: sha256:224afe981562cd065ce70dc75d0779b3ed4ede7b35bb8892232638d839483cce
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jesilee/CIDMD_setup
  tools:
  - 6–31G*
  - Avogadro
  - B3LYP
  - CIDMD_compare
  - CIDMD_setup
  - TeraChem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# protonation-site-protomer-enumeration

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Systematically enumerate candidate protonated molecular ion (protomer/tautomer) structures at plausible protonation sites—primarily heteroatoms—before running CIDMD simulations, then evaluate each protomer's predicted spectrum against experimental MS/MS to select the most probable protomer(s). The article shows that different protomers yield markedly different fragmentation patterns, so enumeration directly controls prediction accuracy.

## When to use

Use when preparing a small molecule (<205 Da or larger) for CID-MS/MS spectrum prediction from structure alone, and the exact protonation site in ESI is unknown: enumerate every candidate protomer/tautomer at heteroatom sites and simulate each independently before comparing to the experimental spectrum.

## When NOT to use

- The experimental spectrum is already known and only a single dominant protomer was confirmed by prior orthogonal evidence—enumerating all sites adds cost without improving the prediction.
- The molecule's protonation state under ESI is known unambiguously (e.g., permanent quaternary ammonium) and there are no alternative heteroatom sites—run a single CIDMD simulation instead.
- Molecules with very large numbers of protomers/tautomers where the article warns average CIDMD prediction accuracy decreases as protomer count grows, making exhaustive enumeration computationally prohibitive.

## Inputs

- Neutral molecule structure in .xyz format (e.g., from PubChem or vendor SMILES converted to 3D)
- List of candidate protonation sites (heteroatoms such as N, O, S) and tautomeric arrangements
- Experimental CID-MS/MS spectrum for the precursor ion (e.g., from NIST20) for post-hoc protomer discrimination
- mol_info.in file specifying mol_name, mol_id, mol_mf, mol_mw, mol_xw

## Outputs

- A set of optimized protonated-ion .xyz structures, one per enumerated protomer/tautomer
- Per-protomer CIDMD input folders (cid/calcs/XXX/chunk_0000/) containing start.xyz and vel.xyz
- Per-protomer predicted CID-MS/MS spectra in cidmd.msp and cidmd.jdx formats
- Entropy/cosine/Wdot similarity scores per protomer against the experimental spectrum, supporting protomer ranking

## How to apply

Construct each candidate protonated molecular ion manually in Avogadro by adding a proton to every plausible heteroatom site, generating a distinct start structure per protomer/tautomer rather than a single structure. Optimize each ion's 3D shape in TeraChem with unrestricted B3LYP/6–31G* before simulation. Run a full CIDMD trajectory set (argon collider placed 5 Å from the point of closest approach, NVE ensemble, 1.0 fs timestep, ≥1000 steps) separately for each protomer, and convert accumulated fragmentations into a per-protomer predicted MSP spectrum. Compare each protomer's spectrum to the experimental spectrum using the entropy similarity score (plus cosine and mass-weighted dot scores); the article found a jump from 624 ± 189 to 828 ± 77 when restricting to the most probable protomers, confirming that protomer selection—not just fragmentation simulation—is a major accuracy lever. When multiple protomers produce overlapping but non-identical fragment ion sets, treat the protomer with the highest entropy similarity as the dominant gas-phase species; if several score comparably, report all as plausible since specific fragment ions can arise from different protomers via multiple fragmentation pathways.

## Related tools

- **Avogadro** (Manually construct each candidate protonated molecular ion structure at its protonation site)
- **TeraChem** (Optimize the 3D shape of each enumerated protomer with unrestricted B3LYP/6–31G* before CIDMD)
- **CIDMD_setup** (Generate collision input files (start.xyz, vel.xyz) per protomer structure) — https://github.com/jesilee/CIDMD_setup
- **CIDMD_compare** (Score each protomer's predicted spectrum against the reference experimental spectrum to rank protomers) — https://github.com/jesilee/CIDMD_compare

## Examples

```
python3 CIDMD_setup.py protomer_N1.xyz --set_distance 5.0 --n_points 200 --v_scale 8; terachem run.in &> run.out
```

## Evaluation signals

- Every enumerated protomer yields an optimized .xyz whose total charge is +1 and whose proton is positioned on the intended heteroatom site.
- For each protomer, CIDMD_analysis produces a cidmd.msp with precursor m/z equal to the protonated molecular mass (within 0.01 mDa tolerance).
- Entropy similarity score of the best protomer's predicted spectrum vs. the experimental spectrum reaches the high-scoring regime reported in the paper (≈828 when the correct protomer is selected, vs. ~624 averaged over all protomers).
- Across the protomer set, fragment-ion m/z values in the predicted spectra collectively cover the experimental spectrum's major peaks; a protomer whose predicted peaks miss all major experimental ions is likely a minor/nonforming protomer.
- The number of enumerated protomers is documented per molecule, and the selected protomer(s) with highest similarity are explicitly recorded rather than collapsing to a single prediction.

## Limitations

- With increasing numbers of protomers and tautomers, the average CIDMD prediction accuracy decreases, so exhaustive enumeration is expensive and may reduce confidence for molecules with many sites.
- No method currently exists to accurately predict protomer formation itself, so fewer, targeted simulations cannot yet be chosen a priori.
- Specific fragment ions can be produced from different protomers via multiple fragmentation pathways, making unique protomer assignment ambiguous when several protomers share dominant peaks.

## Evidence

- [abstract] Different protomers showed large differences in fragmentation; prediction accuracy decreased as the number of protomers and tautomers increased.: [evidence span withheld — anchor status: not_found]
