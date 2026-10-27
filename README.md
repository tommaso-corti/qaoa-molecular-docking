# qaoa-molecular-docking

Gate-based quantum optimisation for molecular docking: QAOA with RX and XY mixers and a linear-ramp variant (LR-QAOA), implemented from scratch in NumPy and simulated exactly on a 12-qubit statevector.

The project adapts the QUBO formulation of Triuzzi et al. [1], who solved molecular docking as a weighted subgraph isomorphism problem on a quantum annealer, to gate-based algorithms, and compares three variants with exhaustive enumeration and simulated annealing. It is a self-directed exploratory study. Its purpose is to understand how these algorithms behave on a constrained optimisation problem, not to provide a rigorous benchmark or evidence of a quantum advantage.

A non-technical account of the first version of the project is available on [Medium](https://medium.com/@corti.tommaso/i-spent-a-week-running-quantum-algorithms-on-a-drug-discovery-problem-heres-what-i-learned-49f654287ce5).

## Contents

| Notebook | Instance | Role |
|---|---|---|
| `molecular_docking_qaoa_toy.ipynb` | Synthetic three-atom ligand and four-point pocket | Validation: the optimal mapping is known by construction |
| `molecular_docking_qaoa_5f4l.ipynb` | PDB structure 5F4L (CDK2 kinase) | Real pharmacophore distances from the data set of Ding et al. [6], as provided in [7] |

Both instances have three ligand points and four pocket points, hence 12 binary variables and 12 qubits, of which only 24 basis states out of 4096 correspond to feasible (injective) mappings. The two notebooks share the same structure: problem setup, QUBO construction, classical baselines, QAOA with the RX mixer, QAOA with the XY mixer, LR-QAOA, a summary of results, an analysis of the energy scale of the penalty term, and conclusions.

The size of the instances is set by the cost of the classical optimisation loop, not by memory: a 12-qubit statevector occupies 64 KiB, and a 24-qubit one would still fit in 256 MiB.

## Main findings

All three variants concentrate most of the amplitude on the feasible subspace, raising the probability of a feasible mapping from 0.59% for a random bitstring to about 90% or more. Within the feasible subspace, however, they hardly distinguish the optimal mapping from the others: with the original penalty weight, the conditional probability of the optimum stays close to the uniform value 1/24, and additional layers do not improve it.

Section 8 of each notebook shows that this behaviour is due to the energy scale set by the penalty term rather than to the depth of the circuits. After normalisation the objective terms are compressed into a narrow band of energies, which the circuit cannot resolve with angles of order one. The XY mixer used here preserves only the one-placement-per-atom constraint, so any mixing moves amplitude into infeasible states; the optimiser therefore converges to mixing angles close to π/2 or π, at which the mixer reduces to a permutation of the feasible mappings and the state does not change. Reducing the penalty weight towards its minimum admissible value increases selectivity at the cost of feasibility. On 5F4L with LR-QAOA at p = 30, lowering the penalty from 3.5 to 0.5 times the largest objective coefficient raises the conditional probability of the optimum from 6.3% to 47.2% and the probability of sampling the optimum from 6.2% to 14.3%, while the probability of a feasible mapping falls from 98.6% to 30.4%.

An earlier version of the notebooks attributed the lack of improvement at larger depth to barren plateaus. The analysis above does not support that interpretation at 12 qubits, and the notebooks have been revised accordingly.

## Running the notebooks

```
pip install -r requirements.txt
jupyter notebook
```

No quantum hardware or quantum SDK is required. A complete execution of one notebook takes more than three hours on a laptop, almost entirely in the QAOA optimisations with 2p free parameters. Results are saved after each circuit depth in `checkpoints_toy/` and `checkpoints_5f4l/`, together with the state of the random number generator, so that an interrupted run resumes where it stopped and a repeated run loads the saved results in a few minutes. A checkpoint is ignored if the problem instance or the optimisation hyperparameters change, and setting `FORCE_RECOMPUTE = True` recomputes everything. Figures are saved in `figures_toy/` and `figures_5f4l/`.

The optimisations with 2p free parameters use up to 150 random restarts, and their results can differ between environments, because small numerical differences change which restart reaches the lowest energy. The LR-QAOA results and the analysis of Section 8, which use few parameters and their own random generators, were reproduced exactly across executions.

## Limitations

Each configuration is the result of a single optimisation, without error bars. The instances are small enough to be solved classically in a fraction of a second, and the study says nothing about scaling. The cost unitary is applied as an exact phase on each amplitude, so gate counts, circuit depth and noise are not considered. The objective of [1] depends only on distances, so the optimum is the best geometric match, not a validated binding pose.

## References

[1] Triuzzi et al., *Quantum Sci. Technol.* 10, 045049 (2025).

[2] S. Hadfield, Z. Wang, B. O'Gorman, E. G. Rieffel, D. Venturelli and R. Biswas, "From the Quantum Approximate Optimization Algorithm to a Quantum Alternating Operator Ansatz", *Algorithms* 12, 34 (2019).

[3] E. Farhi, J. Goldstone and S. Gutmann, "A Quantum Approximate Optimization Algorithm", arXiv:1411.4028 (2014).

[4] Cadavid et al., "Large-scale portfolio optimization on a trapped-ion quantum computer", arXiv:2602.23976 (2026).

[5] J. A. Montanez-Barrera and K. Michielsen, "Towards a universal QAOA protocol: evidence of a scaling advantage in solving some combinatorial optimization problems", arXiv:2405.09169 (2024).

[6] Q.-M. Ding, Y.-M. Huang and X. Yuan, "Molecular docking via quantum approximate optimization algorithm", *Phys. Rev. Applied* 21, 034036 (2024), arXiv:2308.04098.

[7] B. Li, molecular-docking-qaoa, [github.com/25benjaminli/molecular-docking-qaoa](https://github.com/25benjaminli/molecular-docking-qaoa).
