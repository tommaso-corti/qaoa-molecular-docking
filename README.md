# qaoa-molecular-docking

Gate-based quantum optimization for molecular docking — QAOA (RX & XY mixer) and LR-QAOA implemented from scratch on a 12-qubit statevector simulator.

Inspired by Triuzzi et al. (2025), who demonstrated molecular docking on a D-Wave quantum annealer using a weighted subgraph isomorphism QUBO formulation. This project adapts that mathematical formulation to a gate-based paradigm and benchmarks three algorithmic variants against Simulated Annealing and Brute Force enumeration.

📝 [Read the full write-up on Medium](https://medium.com/@corti.tommaso/i-spent-a-week-running-quantum-algorithms-on-a-drug-discovery-problem-heres-what-i-learned-49f654287ce5)

---

## What's in here

| Notebook | Problem | Notes |
|---|---|---|
| `molecular_docking_qaoa_toy.ipynb` | Synthetic 3-atom ligand, 4-point pocket grid | Used for debugging and validation |
| `molecular_docking_qaoa_5f4l.ipynb` | PDB structure 5F4L (CDK2 kinase) | Real pharmacophore distance data from crystal structure |

Both notebooks are 12 qubits (3 atoms × 4 pocket points = 12 binary variables), the practical limit for exact statevector simulation on a standard laptop.

---

## Algorithms implemented

All quantum simulation is implemented from scratch in NumPy. Gates are applied as linear algebra operations on the full 4096-dimensional statevector.

- **QAOA with RX mixer**
- **QAOA with XY mixer**
- **LR-QAOA**

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

---

## References

- Triuzzi et al., *Quantum Sci. Technol.* 10, 045049 (2025) — QUBO formulation for molecular docking
- Benjamin Li's github: [github.com/25benjaminli/molecular-docking-qaoa](https://github.com/25benjaminli/molecular-docking-qaoa)
