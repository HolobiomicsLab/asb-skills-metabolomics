---
name: ablation-component-contribution-attribution
description: Use when a paper claims performance gains from several newly added components
  and you must reproduce/verify the per-component contribution breakdown via controlled
  ablation, e.g. CFM-ID 4.0 on Metlin ESI-MS/MS prediction.
when_to_use_negative:
- The goal is overall spectral-prediction accuracy or compound identification benchmarking,
  not attribution of gains to individual components — run full CFM-ID 4.0 against
  CFM-ID 2.0/3.0 without ablation variants.
- Input spectra come from instruments other than QTOF — the authors note CFM-ID 4.0
  currently does not handle MS spectra collected on non-QTOF instruments.
- You cannot retrain model variants (only pretrained deployments are available), since
  the ablation requires training each reduced configuration under the same cross-validation
  setup.
tools:
- name: CFM-ID 4.0
  role: Model under ablation; topology, ring-cleavage, and MSRB components are toggled
    on/off and retrained to measure per-component deltas
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker (wishartlab/cfmid)
  role: Containerized distribution used to obtain and run the CFM-ID 4.0 ablation
    variants reproducibly
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ClassyFire
  role: Identifies chemical classes eligible for MSRB fragmentation, gating which
    compounds route through the rule-based ablation component
- name: Metlin
  role: Source of the experimental ESI-MS/MS training and evaluation spectra used
    to score each ablation variant
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:0cacb6c6d6856ecd1721af5f9c0881ba44a2ae77fa1437de8ad8ad6d000219ef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - ClassyFire
  - Docker (wishartlab/cfmid)
  - Metlin
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# ablation-component-contribution-attribution

## Summary

Decomposes a multi-component method's reported performance gain by cumulatively ablating its added components (here CFM-ID 4.0's molecular topology features, sequential ring cleavage modeling, and MSRB rules) and attributing metric deltas to each component. Yields quantitative contribution shares rather than a single aggregate accuracy claim.

## When to use

Use when a paper claims performance gains from several newly added components and you must reproduce/verify the per-component contribution breakdown via controlled ablation, e.g. CFM-ID 4.0 on Metlin ESI-MS/MS prediction.

## When NOT to use

- The goal is overall spectral-prediction accuracy or compound identification benchmarking, not attribution of gains to individual components — run full CFM-ID 4.0 against CFM-ID 2.0/3.0 without ablation variants.
- Input spectra come from instruments other than QTOF — the authors note CFM-ID 4.0 currently does not handle MS spectra collected on non-QTOF instruments.
- You cannot retrain model variants (only pretrained deployments are available), since the ablation requires training each reduced configuration under the same cross-validation setup.

## Inputs

- Metlin 2015/2019 QTOF ESI-MS/MS training sets for [M+H]+ and [M−H]− at collision energies 10/20/40 eV
- Experimental peak lists used for Dice/Dot Product spectral matching
- CFM-ID 4.0 model configuration (fragmentation graph depth 2, max ring breaks 2, two 128-node hidden layers, 10-fold CV splits)

## Outputs

- Component-share table attributing performance deltas to topology features (~65%), ring cleavage (~35%), and MSRB rules (~1%)
- Per-variant metric table (Dice, Precision, Recall, re-weighted Stein Dot Product) for baseline through full CFM-ID 4.0
- Fraction of MS/MS spectra predicted by the MSRB rule-based module (expected <3%)

## How to apply

Load the four Metlin 2015/2019 QTOF ESI-MS/MS training sets ([M+H]+ and [M−H]− at collision energies 10/20/40 eV) plus the experimental peak lists used for Dice/Dot Product scoring. Train CFM-ID 4.0 variants under the 10-fold cross-validation framework with the paper's hyperparameters — fragmentation graph depth 2, max ring breaks 2, fully connected NN with two 128-node hidden layers — in a cumulative ablation: baseline without connectivity-matrix topology features, then adding topology features, then sequential ring cleavage modeling, then MSRB rules. For each variant compute Dice coefficient, Precision, Recall, and re-weighted Stein Dot Product against experimental spectra using a peak match tolerance of m/z difference < 0.01 Da or 10 ppm, whichever comes first. Attribute the delta between the full model and each ablation stage to the corresponding component, and separately tabulate the fraction of spectra predicted via the MSRB module. Accept the run if derived shares land within a few percentage points of the paper's reported values (topology ~65%, ring cleavage 35%, MSRB ~1%; MSRB predicted <3% of all MS/MS spectra).

## Related tools

- **CFM-ID 4.0** (Model under ablation; topology, ring-cleavage, and MSRB components are toggled on/off and retrained to measure per-component deltas) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker (wishartlab/cfmid)** (Containerized distribution used to obtain and run the CFM-ID 4.0 ablation variants reproducibly) — https://hub.docker.com/r/wishartlab/cfmid
- **ClassyFire** (Identifies chemical classes eligible for MSRB fragmentation, gating which compounds route through the rule-based ablation component)
- **Metlin** (Source of the experimental ESI-MS/MS training and evaluation spectra used to score each ablation variant)

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- Reproduced component shares fall within a few percentage points of the paper's values: molecular topology ~65%, ring cleavage ~35%, rule-based ~1%.
- The MSRB module accounts for fewer than 3% of all predicted MS/MS spectra.
- Metrics are monotonically non-decreasing as components are added (baseline ≤ +topology ≤ +ring cleavage ≤ full model) on Dice, Precision, Recall, and re-weighted Stein Dot Product.
- Peak matching uses the paper's tolerance — m/z difference < 0.01 Da or 10 ppm, whichever first — before computing similarity scores.
- All ablation variants are trained with identical hyperparameters (graph depth 2, max ring breaks 2, two 128-node hidden layers, 10-fold CV) so deltas are attributable to components only.

## Limitations

- MSRB rules cover only specific chemical classes (acylcarnitines, acylcholines, flavonols, flavones, flavanones, flavonoid glycosides), capping the rule-based component's contribution share.
- Deeper fragmentation graphs introduce extra computational complexity, constraining ablation sweeps to the stated depth of 2.
- Runtime and accuracy still have room for improvement, and the approach is limited to QTOF ESI-MS/MS data.

## Evidence

- [results] Three-component contribution breakdown: [evidence span withheld — anchor status: not_found]
- [abstract] Component definitions: "This work improves CFM-ID's ability to predict ESI-MS/MS spectra from compounds by (1) learning parameters from features based on the molecular topology, (2) adding a new approach to ring cleavage"
- [intro] MSRB class gating: "the new version of CFM-ID uses ClassyFire37 to identify the specific chemical classes that are suitable for MSRB fragmentation"
- [intro] Topological feature basis: "we developed an improved version of CFM-ID (version 4.0) that learns its parameters from a detailed representation of the molecule’s chemical bond topology"
