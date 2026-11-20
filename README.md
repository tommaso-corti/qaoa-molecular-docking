# qaoa-molecular-docking

Gate-based quantum optimisation for molecular docking: QAOA with RX and XY mixers and a linear-ramp variant (LR-QAOA), implemented from scratch in NumPy and simulated exactly at the statevector level.

The project adapts the QUBO formulation of Triuzzi et al. [1], who solved molecular docking as a weighted subgraph isomorphism problem on a quantum annealer, to gate-based algorithms, and compares them with exhaustive enumeration, simulated annealing and random baselines. It is a self-directed exploratory study. Its purpose is to understand how these algorithms behave on a constrained optimisation problem, not to provide evidence of a quantum advantage.

A non-technical account of the first version of the project is available on [Medium](https://medium.com/@corti.tommaso/i-spent-a-week-running-quantum-algorithms-on-a-drug-discovery-problem-heres-what-i-learned-49f654287ce5).

## Contents

| Notebook | Instance | Role |
|---|---|---|
| `molecular_docking_qaoa_toy.ipynb` | Synthetic three-atom ligand and four-point pocket | Validation: the optimal mapping is known by construction. Also contains the scaling study of Part 2 |
| `molecular_docking_qaoa_5f4l.ipynb` | PDB structure 5F4L (CDK2 kinase) | Real pharmacophore distances from the data set of Ding et al. [6], as provided in [7] |

Both instances have three ligand points and four pocket points, hence 12 binary variables and 12 qubits, of which only 24 basis states out of 4096 correspond to feasible (injective) mappings.

Each notebook has two parts:

- **Part 1** is the original study: problem setup, QUBO construction, classical baselines, QAOA with the RX mixer, QAOA with the XY mixer, LR-QAOA, a summary of results, an analysis of the energy scale of the penalty term, and conclusions. It has been revised to fix how results are measured and compared; the algorithms are unchanged.
- **Part 2 — Revisiting the results** asks whether the weak results of Part 1 are a property of QAOA on this problem or of the way its angles were chosen: a fair classical baseline (2.1), depth-by-depth warm start with INTERP (2.2), the CVaR objective (2.3), fixed-angle LR-QAOA and the transfer of its angles between instances (2.4), and a scaling study on synthetic instances from 12 to 20 qubits (2.5, toy notebook only).

## Main findings

All results are exact statevector simulations, without gate noise or shot noise.

**Part 1, read with corrected metrics.** All QAOA variants concentrate the amplitude on the feasible subspace, but within it they hardly distinguish the optimal mapping: their probability of returning the optimum stays close to that of a random feasible mapping, 1/24 ≈ 4.2%. QAOA-RX at p = 10 is about as likely to return the optimum as a random feasible guess, and the quality of the valid mappings it returns (approximation ratio, average bond distortion) is close to that of random ones. At depths 8 and 10 the optimiser used almost all of its budget of 300,000 circuit evaluations without converging. Simulated annealing returns the exact optimum in 30% (toy) and 48% (5F4L) of its runs; an earlier version reported 80% and about two thirds, because its success test used a tolerance larger than the gap to the second-best mapping.

**Why.** Section 8 shows that, after normalisation, the penalty term compresses the 24 feasible mappings into a narrow band of energies, which the circuit cannot resolve; the XY mixer is effectively switched off by the optimiser. The penalty weight is correct, in the sense that the optimum is the ground state of the QUBO, but it is not a good value for QAOA.

**Part 2.**
- Warm-starting each depth from the previous one (INTERP) makes the energy decrease with depth, with about 19,000 circuit evaluations for all depths up to p = 10, against several hundred thousand in Part 1. The obvious alternative, adding a layer with zero angles, stalls.
- Optimising CVaR instead of the expectation value raises the probability of the optimum at p = 30 from 6.2% to 8.6% on 5F4L; combined with a lower penalty weight it reaches 17.3%, about four times the random feasible value.
- Fixed linear-ramp angles chosen on the toy instance and applied to 5F4L without any optimisation give 8.7% at p = 30, as much as the best angles found on 5F4L itself and more than the variational LR-QAOA of Part 1.
- Simulated annealing still needs only a few thousand classical energy evaluations in total to find the optimum with 99% probability.
- On synthetic instances from 24 to 120 feasible mappings (12 to 20 qubits), the advantage of fixed-angle LR-QAOA over a random feasible guess shrinks from about 2.3 to about 1.5.

On 5F4L the pocket points are pharmacophore features of the protein, whereas the formulation of [1] expects empty positions where a ligand atom can be placed. Even the optimal mapping has an average bond distortion of 2.6 Å, and the second-best is only 0.05 Å worse, so this instance is a benchmark of geometric matching with real distances rather than a realistic docking problem.

## Running the notebooks

Tested with Python 3.11.9 and the versions pinned in `requirements.txt`.

```
pip install -r requirements.txt
jupyter notebook
```

No quantum hardware or quantum SDK is required. A complete execution of one notebook takes about four hours on a laptop: about three and a half for the QAOA optimisations of Part 1 with 2p free parameters, and about half an hour for Part 2. Results are saved after each circuit depth in `checkpoints_toy/` and `checkpoints_5f4l/`, together with the state of the random number generator and the library versions, so that an interrupted run resumes where it stopped and a repeated run loads the saved results in a few minutes. A checkpoint is ignored if the problem instance or the optimisation hyperparameters change, and setting `FORCE_RECOMPUTE = True` recomputes everything. Figures are saved in `figures_toy/` and `figures_5f4l/`.

The Part 1 results were reproduced to four decimals on a second machine. Optimisations with many free parameters can nevertheless depend on small numerical differences between environments: an older execution gave different QAOA-XY results at depths 8 and 10, and the XY warm start of Section 2.2 gave a different probability of the optimum on two machines. Part 2 reports the spread over several seeds where it is affordable.

## Limitations

The study uses exact simulation: gate counts, circuit depth, noise and shot noise are not considered, and nothing in it is a statement about quantum hardware. There is a single real instance; the scaling study uses synthetic instances up to 20 qubits and depths up to p = 30, and gives a trend, not a prediction for larger problems. Many configurations are the result of a single optimisation. Choosing fixed angles by the probability of the optimum requires knowing the optimum of the tuning instance, which is possible only because it is solved classically. The objective of [1] depends only on distances, so the optimum is the best geometric match, not a validated binding pose.

## References

[1] E. Triuzzi et al., "Molecular docking via weighted subgraph isomorphism on quantum annealers", *Quantum Sci. Technol.* 10, 045049 (2025).

[2] S. Hadfield, Z. Wang, B. O'Gorman, E. G. Rieffel, D. Venturelli and R. Biswas, "From the Quantum Approximate Optimization Algorithm to a Quantum Alternating Operator Ansatz", *Algorithms* 12, 34 (2019).

[3] E. Farhi, J. Goldstone and S. Gutmann, "A Quantum Approximate Optimization Algorithm", arXiv:1411.4028 (2014).

[4] Cadavid et al., "Large-scale portfolio optimization on a trapped-ion quantum computer", arXiv:2602.23976 (2026).

[5] J. A. Montañez-Barrera and K. Michielsen, "Towards a Linear-Ramp QAOA protocol: Evidence of a scaling advantage in solving some combinatorial optimization problems", arXiv:2405.09169 (v3, 2025; earlier versions were titled "Towards a universal QAOA protocol").

[6] Q.-M. Ding, Y.-M. Huang and X. Yuan, "Molecular docking via quantum approximate optimization algorithm", *Phys. Rev. Applied* 21, 034036 (2024), arXiv:2308.04098.

[7] B. Li, molecular-docking-qaoa, [github.com/25benjaminli/molecular-docking-qaoa](https://github.com/25benjaminli/molecular-docking-qaoa).

[P1] P. Kl. Barkoutsos, G. Nannicini, A. Robert, I. Tavernelli and S. Woerner, "Improving Variational Quantum Optimization using CVaR", *Quantum* 4, 256 (2020).

[P2] L. Zhou, S.-T. Wang, S. Choi, H. Pichler and M. D. Lukin, "Quantum Approximate Optimization Algorithm: Performance, Mechanism, and Implementation on Near-Term Devices", *Phys. Rev. X* 10, 021067 (2020).

[P3] G. Curato, M. Ponzi, E. Chiatello and R. Giorgetti, "Benchmark study of simulated quantum annealing and asynchronous parallel simulated annealing for quadratic unconstrained binary optimization-based semiprime factorization", *APS Open Science* 1, 000067 (2026).
