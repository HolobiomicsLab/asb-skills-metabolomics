---
name: nmr-solvent-suppression
description: Use when processing raw Bruker 2D FID data (fid/ser) from aqueous samples
  where the dominant solvent peak must be suppressed automatically before Fourier
  transformation and phase correction.
when_to_use_negative:
- Data are already processed/suppressed spectra (e.g. 2D FT peak lists) rather than
  raw time-domain fid/ser files — COLMARvista's automated suppression targets raw
  Bruker time-domain input
- Samples with no dominant solvent/water resonance where suppression would distort
  real peaks
- The goal is NUS reconstruction (SMILE/direct-FT) rather than conventional uniformly
  sampled FID processing
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: COLMARvista
  role: Executes the autonomous processing chain (zero-filling, apodization, water
    suppression, Fourier transformation, phase correction) on 2D and pseudo-3D NMR
    data
  repo: https://github.com/lidawei1975/colmarvista
- name: WebAssembly
  role: Client-side runtime in which raw Bruker fid/ser data are processed directly
    in the browser with automated solvent suppression
  repo: https://github.com/lidawei1975/colmarvista
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:aeeb46328a1f71f9bf79ae666cf8fced7fe7d918fc5d171d0ce74efef6c81c7b
  source_papers:
  - doi: 10.1007/s10858-025-00465-y
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lidawei1975/colmarvista
  tools:
  - COLMARvista
  - JavaScript
  - WebAssembly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# nmr-solvent-suppression

## Summary

Autonomous solvent (water) suppression applied to time-domain or freshly transformed 2D NMR data as one stage of the COLMARvista zero-filling → apodization → solvent suppression → Fourier transformation → phase correction chain, executed client-side via WebAssembly.

## When to use

Use when processing raw Bruker 2D FID data (fid/ser) from aqueous samples where the dominant solvent peak must be suppressed automatically before Fourier transformation and phase correction.

## When NOT to use

- Data are already processed/suppressed spectra (e.g. 2D FT peak lists) rather than raw time-domain fid/ser files — COLMARvista's automated suppression targets raw Bruker time-domain input
- Samples with no dominant solvent/water resonance where suppression would distort real peaks
- The goal is NUS reconstruction (SMILE/direct-FT) rather than conventional uniformly sampled FID processing

## Inputs

- Raw Bruker time-domain FID data (fid file) for 2D experiments
- Raw Bruker ser data for pseudo-3D/serial experiments
- Zero-filled and apodized intermediate FID from the preceding autonomous steps

## Outputs

- Solvent-suppressed FID (intermediate file, e.g. fid_solvent_suppressed.dat)
- Baseline-corrected spectrum from the same automated browser-side pass
- Phased 2D spectrum obtained after downstream Fourier transformation and automated phase correction

## How to apply

Load the raw Bruker time-domain file (fid or ser) into the COLMARvista client-side processing environment. Apply solvent suppression at the correct position in the autonomous chain: after zero-filling and apodization of the FID and before Fourier transformation along both dimensions (the abstract orders the chain as zero-filling, apodization, water suppression, Fourier transformation, phase correction; the methods section lists solvent suppression among automated browser-side steps). Run it with the automated settings rather than manual parameter tuning — COLMARvista performs this step without user intervention. Persist the solvent-suppressed intermediate (e.g. fid_solvent_suppressed.dat) before transforming, so the suppression result is inspectable separately. Baseline correction is listed alongside solvent suppression in the browser-side processing suite and can follow as part of the same automated pass.

## Related tools

- **COLMARvista** (Executes the autonomous processing chain (zero-filling, apodization, water suppression, Fourier transformation, phase correction) on 2D and pseudo-3D NMR data) — https://github.com/lidawei1975/colmarvista
- **WebAssembly** (Client-side runtime in which raw Bruker fid/ser data are processed directly in the browser with automated solvent suppression) — https://github.com/lidawei1975/colmarvista

## Evaluation signals

- Each intermediate file from the chain (fid_zero_filled.dat, fid_apodized.dat, fid_solvent_suppressed.dat, spectrum_ft.dat, spectrum_phased.dat) exists and is non-empty
- The final phased 2D spectrum file is produced and loadable in COLMARvista
- The solvent-suppressed intermediate shows strong attenuation of the solvent resonance relative to the apodized pre-suppression FID
- Processing completes fully autonomously (no manual per-spectrum parameter entry) when a raw Bruker fid/ser is loaded
- Suppression is applied upstream of Fourier transformation in the saved-step ordering, matching the abstract's chain order

## Limitations

- Local (offline) use requires manual browser configuration for WebWorker/WebAssembly (Chrome --allow-file-access-from-files, Firefox security.fileuri.strict_origin_policy=false, Safari Disable Local File Restrictions), which the README flags as a security risk
- Automated suppression offers no user-tuned parameters for edge cases; spectra with unusual solvent conditions must rely on the defaults
- No changelog is available in the repository, so changes to the automated suppression implementation between versions cannot be traced

## Evidence
