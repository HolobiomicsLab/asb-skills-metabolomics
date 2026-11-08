---
name: mass-decimal-digit-integer-conversion
description: Use when feeding mass_peak values (e.g. from GAME/examples/MW, such as
  286.24-478.41 Da for Cuscuta chinensis) into the csccp-solver-cli DP solver, or
  when benchmarking how runtime scales with mass precision D=1-5.
when_to_use_negative:
- Input masses are already provided in integer format in a prepared CSCCP instance
  — re-scaling would change the searched mass by a factor of 10^D
- The GPU-accelerated G.A.M.E. path is available and D=5 precision is required; CPU
  DP runtime grows tenfold per digit and becomes infeasible at higher D
- Only a qualitative structure ranking is needed and sub-Dalton precision suffices
  (D=1), making extra digits unnecessary overhead
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: G.A.M.E.
  role: DP solver (csccp-solver-cli) whose --dec flag consumes the 10^D-scaled integer
    masses
  repo: https://github.com/CMDM-Lab/GAME
- name: Python 2.7
  role: Scripting environment for batch-scaling masses by 10^D and timing solver runs
    at each D
  repo: https://github.com/CMDM-Lab/GAME
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-017-0238-7
    title: 'G.A.M.E.: GPU-accelerated mixture elucidator'
  source_content_hash: sha256:39a0e4f48a2f07a407fbe9bfb6fcab63371b6a2c1f3bd558889fd5e50614a17a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/GAME
  tools:
  - G.A.M.E.
  - Python 2.7
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# Mass decimal-digit integer conversion

## Summary

Converts floating-point molecular masses into integers by multiplying by 10^D (D decimal digits) so that the DP/CSCCP solvers in G.A.M.E. and csccp-solver-cli can process them; the number of digits D directly controls pseudo-polynomial runtime (each added digit grows runtime roughly tenfold).

## When to use

Use when feeding mass_peak values (e.g. from GAME/examples/MW, such as 286.24-478.41 Da for Cuscuta chinensis) into the csccp-solver-cli DP solver, or when benchmarking how runtime scales with mass precision D=1-5.

## When NOT to use

- Input masses are already provided in integer format in a prepared CSCCP instance — re-scaling would change the searched mass by a factor of 10^D
- The GPU-accelerated G.A.M.E. path is available and D=5 precision is required; CPU DP runtime grows tenfold per digit and becomes infeasible at higher D
- Only a qualitative structure ranking is needed and sub-Dalton precision suffices (D=1), making extra digits unnecessary overhead

## Inputs

- Floating-point target molecular weight (e.g. 168.195105 from GAME/examples/MW)
- Chosen decimal-digit precision D (1-5)
- Seed scaffold folder and .cIdx configuration index for the CSCCP instance

## Outputs

- Integer mass w0 = mass_peak x 10^D used by the DP solver
- Scaled mass_peak_min/mass_peak_max bounds consistent with the ppm window
- Per-D runtime table (columns: D, runtime_seconds) for scaling verification

## How to apply

Choose a decimal-digit setting D (README guidance: 'In general, r is set to 3~10, dec is set to 1~5'); the paper allows up to five decimal digits with GPU acceleration. Multiply the target mass by 10^D and round to an integer before invoking the solver, matching the -dec/--dec flag so min/max mass windows (mass_peak_min/mass_peak_max) are computed on the same integer grid. For runtime-scaling benchmarks, run the CPU iterative DP (-v idp) successively at D=1, 2, 3 on the same scaffolds/configuration (from the .cIdx file) and wall-clock each invocation. Rationale: the DP loops are indexed over integer mass states, so runtime grows roughly tenfold per added decimal digit; integer conversion is a hard requirement of the program, not an optimization.

## Related tools

- **G.A.M.E.** (DP solver (csccp-solver-cli) whose --dec flag consumes the 10^D-scaled integer masses) — https://github.com/CMDM-Lab/GAME
- **Python 2.7** (Scripting environment for batch-scaling masses by 10^D and timing solver runs at each D) — https://github.com/CMDM-Lab/GAME

## Examples

```
csccp-solver-cli -s GAME/examples/normal/example-data -m 168.195105 -v idp -l cc -r 3 -c 0 --dec 5
```

## Evaluation signals

- For Cuscuta chinensis benchmarking, each consecutive-D runtime ratio (D=1→2, 2→3) falls between 5 and 15, consistent with the reported tenfold-per-digit DP scaling
- Solver output reports mass_peak (w0) equal to the scaled integer target, and min_possible_weight/max_possible_weight brackets it
- End-to-end script_runs check: csccp-solver-cli completes and returns the top R=3 ranked molecules for every D setting
- The same D is used consistently for both the -m/-dec arguments so the integer grid matches the mass window

## Limitations

- CPU DP runtime grows by orders of magnitude as mass decimal digits increase; only D=1-3 are feasible on CPU, while G.A.M.E. extends practical work to five decimal digits
- All masses in the program must be integers; mixing scaled and unscaled values corrupts the DP mass-state indexing
- Maximum searchable weight is bounded by the scaffold (e.g. wmax=500; s0000000001 max_possible_weight 498.210115), so scaling cannot rescue masses outside the scaffold range

## Evidence

- [methods] Mass value conversion to integer format (×10^D): "all mass values must be converted to integers by multiplying 10D in the program"
- [intro] Tenfold runtime growth per decimal digit: [evidence span withheld — anchor status: not_found]
- [abstract] Five-decimal-digit capability with GPU acceleration: "considerably improves the performance of the DP, allowing up to five decimal digits for input mass data"
- [readme] README guidance on dec parameter: "In general, r is set to 3~10, dec is set to 1~5"
- [readme] Scaffold weight bounds constrain the search: "min_possible_weight: 152.120115 max_possible_weight: 498.210115"
