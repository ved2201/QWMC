# Quantum-Classical Hybrid Weighted Constrained Sampling (QWCS)

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![Qiskit](https://img.shields.io/badge/Qiskit-2.2%2B-purple)](https://qiskit.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A quantum-classical hybrid framework for **Weighted Model Counting (WMC)** and **Weighted Constrained Sampling** over Boolean CNF formulas. This project implements a custom weighted Grover's algorithm with a classical-to-quantum CNF oracle pipeline, Quantum Phase Estimation (QPE)-based WMC, and a rigorous performance evaluation suite.

---

## Overview

Given a Boolean formula in CNF and a set of variable weights, the system:
1. Converts the CNF formula into a Qiskit `PhaseOracle`
2. Constructs a **weighted state preparation operator** (A) using Ry rotations
3. Builds a **custom weighted Grover operator** with a proper reflection-about-zero diffuser
4. Amplifies satisfying states via iterative Grover steps
5. Estimates the **Weighted Model Count (WMC)** using **Quantum Phase Estimation (QPE)**
6. Evaluates sampling quality via Fidelity, KL Divergence, Jensen-Shannon Divergence, and Cross-Entropy

---

## Project Structure
├── cnf_converter.py # Boolean formula → numerical CNF clause list
├── oracle.py # Numerical CNF → Qiskit PhaseOracle
├── oracle_verifier.py # Unit test for oracle phase-flip correctness
├── rotations.py # Weighted state preparation gate (Ry rotations)
├── weighted_grover.py # Weighted Grover operator with correct diffuser
├── diffuser_debug.py # Eigenvalue analysis: buggy vs. correct diffuser
├── qpe_wmc.py # QPE-based Weighted Model Counting
├── qwcs_sampler.py # Full QWCS pipeline + performance evaluation
├── visualizations.py # Amplification curves, histogram plots
└── README.md

---

## Installation

```bash
pip install qiskit qiskit-aer sympy pylatexenc
```

Tested on Python 3.10+ with Qiskit 2.2.1 and Qiskit-Aer 0.17.2.

---

## Quickstart

### 1. Convert a Boolean formula to CNF

```python
from cnf_converter import convert_formula_to_cnf_numbers

formula = "(a | b) & (~a | c)"
clauses, mapping = convert_formula_to_cnf_numbers(formula)
# clauses: [, [3, -1]][1][2]
# mapping: {'a': 1, 'b': 2, 'c': 3}
```

### 2. Run Weighted Constrained Sampling

```python
num_vars = 3
clauses = [, [-1, -2, 3], [-1, 2, -3], [1, -2, -3]][2][3][1]
weights = [0.2, 0.9, 0.3]  # P(x_i = True)

# Build oracle, rotation gate, and weighted Grover operator
oracle = create_numerical_cnf_oracle(num_vars, clauses)
rot_gate = create_rotations_gate(weights)
grover_op = create_weighted_grover_operator(num_vars, oracle, rot_gate)

# Run simulation (4 Grover iterations, 8192 shots)
```

### 3. Estimate WMC via QPE

```python
# Uses 10 evaluation qubits for phase precision
pe_circuit = build_qpe_circuit(grover_op, num_eval_qubits=10, initial_state_prep=rot_gate)
# Output: estimated WMC ≈ 0.5945, classical WMC = 0.5960, error = 0.0015
```

---

## Key Results

| Metric | Value | Ideal |
|---|---|---|
| Fidelity | **0.9958** | → 1.0 |
| KL Divergence | 0.1155 | → 0.0 |
| Jensen-Shannon Divergence | **0.0042** | → 0.0 |
| Cross-Entropy | 1.0683 | Lower is better |
| WMC Estimation Error | **0.0015** | → 0.0 |

The quantum sampler achieves near-ideal fidelity (0.9958) and very low Jensen-Shannon divergence (0.0042) against the classical weighted target distribution over 8192 shots.

---

## Algorithm Details

### Weighted State Preparation (A Operator)
Each qubit `i` is initialized with an Ry rotation:
θ_i = 2 · arcsin(√w_i)

where `w_i = P(x_i = True)` is the variable weight.

### Weighted Grover Diffuser
The diffuser implements reflection about the weighted initial state A|0⟩:
D_w = A · R_0 · A†

where R_0 flips the phase of |0⟩. Eigenvalue analysis confirms the diffuser is a valid reflection (eigenvalues ∈ {+1, −1}).

### QPE-based WMC
The WMC is recovered from the dominant QPE eigenphase φ:
WMC = sin²(φ / 2)

---

## Oracle Verification

A dedicated verification script checks that the `PhaseOracle` correctly applies a phase flip of −1 to satisfying assignments and +1 to non-satisfying ones by inspecting the unitary diagonal.
state |000> (solution): phase = (−1+0j) 
state |110> (non-solution): phase = (+1+0j)

---

## Notes

- `PhaseOracle` is deprecated in Qiskit 2.2; migrate to `PhaseOracleGate` for Qiskit 3.0 compatibility.
- The number of Grover iterations is set experimentally (`k=4`). Optimal `k` can be computed as `⌊(π/4)·√(2^n / M)⌋` where M is the number of solutions.
- All simulations use `AerSimulator` (statevector-based).

---

## Citation

If you use this code in your research, please cite:

```bibtex
@misc{vedant2026qwcs,
  author = {Vedant Waykole},
  title  = {Quantum-Classical Hybrid Weighted Constrained Sampling},
  year   = {2026},
  url    = {https://github.com/<ved2201>/<QWMC>}
}
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.
