# qaoa-molecular-docking

Gate-based quantum optimization for molecular docking — QAOA (RX & XY mixer) and LR-QAOA implemented from scratch on a 12-qubit statevector simulator.

Inspired by Triuzzi et al. (2025), who demonstrated molecular docking on a D-Wave quantum annealer using a weighted subgraph isomorphism QUBO formulation. This project adapts that mathematical formulation to a gate-based paradigm and benchmarks three algorithmic variants against Simulated Annealing and Brute Force enumeration.

📝 [Read the full write-up on Medium](https://medium.com/@corti.tommaso/i-spent-a-week-running-quantum-algorithms-on-a-drug-discovery-problem-heres-what-i-learned-49f654287ce5)

---

## What's in here

| Notebook | Problem | Notes |
|---|---|---|
| `molecular_docking_qaoa_toy.ipynb` | Synthetic 3-atom ligand, 4-point pocket grid | Optimal mapping known by construction — used for debugging and validation |
| `molecular_docking_qaoa_5f4l.ipynb` | PDB structure 5F4L (CDK2 kinase) | Real pharmacophore distance data from crystal structure |

Both notebooks are 12 qubits (3 atoms × 4 pocket points = 12 binary variables), the practical limit for exact statevector simulation on a standard laptop.

---

## Algorithms implemented

All quantum simulation is implemented from scratch in NumPy — no Qiskit, no PennyLane, no quantum SDK dependency. Gates are applied as linear algebra operations on the full 4096-dimensional statevector.

- **QAOA with RX mixer** — standard single-qubit rotations, unconstrained exploration of the full Hilbert space
- **QAOA with XY mixer** — pairwise swap gates that preserve the one-hot constraint by construction, restricting exploration to the valid subspace
- **LR-QAOA** — linear ramp schedule (2 parameters regardless of circuit depth), motivated by the adiabatic protocol; reaches p=30 in ~2 minutes on a laptop where standard QAOA gets stuck at p=10

Classical baselines: exact Brute Force enumeration and Simulated Annealing.

---

## How to run

```bash
pip install numpy scipy matplotlib
jupyter notebook
```

Open either notebook and run all cells. No quantum hardware or quantum SDK required.

---

## Context and limitations

This work is intentionally exploratory rather than a rigorous benchmarking study. The results should be interpreted as pipeline validation — an attempt to understand what actually happens when these algorithms are implemented, tuned, and tested on structured optimization problems.

Specific limitations:
- Single optimizer initialization per run (COBYLA, deterministic) — results are not robust estimates; proper benchmarking requires 20–50 restarts with variance reporting
- P(valid) and P(optimal) are not tracked separately
- Statevector simulation assumes perfect gates — real hardware noise will degrade all results shown here

This is the validation layer. Quantum advantage for problems of this class requires 50+ qubits, where classical enumeration fails entirely.

---

## References

- Triuzzi et al., *Quantum Sci. Technol.* 10, 045049 (2025) — QUBO formulation for molecular docking
- Ding et al., arXiv:2308.04098 (2023) — pharmacophore data source for 5F4L
- Cadavid et al., arXiv:2602.23976 (2026) — large-scale portfolio optimization on trapped-ion hardware
- Tripier et al., arXiv:2604.19481 (2026) — IonQ Walking Cat fault-tolerant architecture
- Benjamin Li's implementation: [github.com/25benjaminli/molecular-docking-qaoa](https://github.com/25benjaminli/molecular-docking-qaoa)
