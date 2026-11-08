---
name: failed-descriptor-compound-removal
description: 'Use when building the Retip feature matrix from SMILES: some compounds
  fail rCDK descriptor calculation during getCD() and must be auto-removed to yield
  a complete-case 286-descriptor table.'
when_to_use_negative:
- Structures contain salts or metal-containing compounds that should be curated/reformatted
  with the ChemAxon standardizer first, rather than silently dropped via descriptor
  failure
- The descriptor matrix is already computed and complete — re-running removal would
  needlessly discard compounds
- The task is retention-time prediction itself (e.g. MS-FINDER isomer filtering) rather
  than constructing the descriptor feature matrix
tools:
- name: Retip
  role: Provides the getCD() function that computes descriptors and automatically
    removes compounds whose descriptor calculation fails
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: R CDK wrapper underlying getCD() that performs the actual per-compound chemical
    descriptor computation
- name: ChemAxon standardizer
  role: Upstream structure curation (salt/metal removal, cxsmiles conversion, 2D coordinates,
    explicit hydrogens) that minimizes downstream descriptor failures
- name: alvaDesc
  role: Optional visual inspection of the descriptor space (molecular-weight histograms,
    logP distributions, PCA)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:d1440ff25d10d00c82fb549bed27ee4b3a6bb1d1829cd974d6ebe62af2fdeeae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - ChemAxon
  - ChemAxon standardizer
  - OpenBabel
  - R
  - Retip
  - alvaDesc
  - rcdk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# failed-descriptor-compound-removal

## Summary

Removes compounds whose chemical descriptors fail computation in Retip's getCD() pipeline (built on rCDK), so that the retention-time prediction models receive a complete-case feature matrix of exactly 286 descriptors per retained compound.

## When to use

Use when building the Retip feature matrix from SMILES: some compounds fail rCDK descriptor calculation during getCD() and must be auto-removed to yield a complete-case 286-descriptor table.

## When NOT to use

- Structures contain salts or metal-containing compounds that should be curated/reformatted with the ChemAxon standardizer first, rather than silently dropped via descriptor failure
- The descriptor matrix is already computed and complete — re-running removal would needlessly discard compounds
- The task is retention-time prediction itself (e.g. MS-FINDER isomer filtering) rather than constructing the descriptor feature matrix

## Inputs

- Compound library (e.g. Retip HILIC dataset of 981 primary metabolites and biogenic amines) with SMILES codes and retention times
- ChemAxon-standardized structures in cxsmiles format with 2D coordinates and explicit hydrogens

## Outputs

- descriptors_table.csv: a complete-case descriptor table with 286 chemical descriptor columns per retained compound
- Set of compounds excluded because descriptor calculation failed

## How to apply

First standardize structures with the ChemAxon standardizer to remove salts and metal-containing compounds, convert to ChemAxon Extended SMILES (cxsmiles), generate 2D coordinates from connectivity (2D-based descriptors avoid 3D computational overhead), and add explicit hydrogens so Alog/XlogP are computed accurately. Then run Retip's getCD() function, which relies on the rcdk package, to compute 286 chemical descriptors per compound; any compound whose descriptor calculation fails is automatically removed rather than imputed. Export the resulting complete-case descriptor table to descriptors_table.csv and verify it contains exactly 286 descriptor columns per retained compound with failed compounds gone. Optionally inspect the descriptor space with alvaDesc via molecular-weight histograms, logP distributions, and PCA.

## Related tools

- **Retip** (Provides the getCD() function that computes descriptors and automatically removes compounds whose descriptor calculation fails) — https://github.com/olobion/Retip
- **rcdk** (R CDK wrapper underlying getCD() that performs the actual per-compound chemical descriptor computation)
- **ChemAxon standardizer** (Upstream structure curation (salt/metal removal, cxsmiles conversion, 2D coordinates, explicit hydrogens) that minimizes downstream descriptor failures)
- **alvaDesc** (Optional visual inspection of the descriptor space (molecular-weight histograms, logP distributions, PCA))

## Examples

```
devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Output descriptors_table.csv contains exactly 286 descriptor columns per retained compound
- No compounds that failed descriptor calculation remain in the exported table (complete-case check)
- Explicit hydrogens were added before computation, confirmed by accurate Alog/XlogP values
- No missing/imputed values in the descriptor matrix — failed rows are dropped, not patched
- Descriptor space passes sanity inspection (alvaDesc molecular-weight histograms, logP distributions, PCA)

## Limitations

- Molecular descriptors alone may not fully explain complex interactions between LC buffer systems, microenvironments, and adsorption/desorption processes
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity; predictions outside the training chemical landscape may be more erroneous
- None of the current models yield validation MAEs better than ±1 min, suggesting the 286 descriptors are not sufficiently complex for narrow retention windows
- Complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines were largely excluded from training

## Evidence

- [methods] getCD computes descriptors and drops failures: [evidence span withheld — anchor status: not_found]
- [methods] validation criterion for the descriptor table: [evidence span withheld — anchor status: not_found]
