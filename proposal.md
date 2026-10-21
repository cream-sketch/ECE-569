# Project Proposal: QEC Simulation-(3) Pauli-Based Computation

**Course:** CSC/ECE 569 Quantum Computing  
**Project:** Project 6 — QEC Simulation  
**Project ID:** QEC Simulation-(3) Pauli-Based Computation  
**Team members:** Yongxiang Lyu, Sheng Chen, Zhuozi Xie  
**Repository:** https://github.com/cream-sketch/ECE-569

## 1. Problem Description and Motivation

Fault-tolerant quantum computation must support universal circuits while keeping the overhead of quantum error correction manageable. Stabilizer circuits, consisting of Clifford operations, Pauli measurements, and stabilizer-state preparation, are especially attractive because they can be simulated efficiently on a classical computer. Stim is a high-performance stabilizer circuit simulator that is designed for this regime and is widely used to study quantum error-correction circuits. LightStim builds higher-level QEC abstractions on top of Stim, including encodings, syndrome extraction, noise models, and decoding workflows.

The main limitation is that a universal quantum circuit also requires non-Clifford operations. In the commonly used Clifford+T gate set, the T gate is the essential non-Clifford resource. Stim itself does not natively simulate arbitrary T gates as part of a stabilizer circuit, and in fault-tolerant implementations T gates are typically realized through magic-state injection, with high-fidelity magic states produced by a distillation protocol. Therefore, a useful QEC simulation workflow should separate the efficiently simulable Clifford structure from the expensive non-Clifford resources.

This project investigates **Pauli-based computation (PBC)** as a bridge between these two regimes. PBC represents a quantum computation using adaptive multi-qubit Pauli measurements acting on magic-state resources. A Clifford+T circuit can first be rewritten using T-gadgets, after which Clifford evolution can be absorbed into classical Pauli-frame tracking and transformed into a sequence of Pauli measurements. The remaining non-stabilizer content is then isolated in the magic-state inputs associated with T gates. This representation is particularly compatible with stabilizer-based QEC simulators because the Clifford and Pauli-measurement portions remain in the efficiently simulable stabilizer domain.

Our central research question is:

> **How can Pauli-based computation be integrated with a stabilizer-based QEC simulator to transform small Clifford+T circuits into measurement-driven Pauli operations while isolating the remaining T-gate resources for magic-state distillation?**

The goal is not to build a complete surface-code architecture or a full physical magic-state factory. Instead, we will develop a focused prototype that demonstrates the PBC transformation, validates functional equivalence on small circuits, and estimates the fault-tolerant resources associated with the remaining non-Clifford operations.

## 2. Proposed Approach

The project will use **Python**, **Stim**, and **LightStim** as the primary software environment, with Qiskit used when convenient for generating or independently validating Clifford+T benchmark circuits.

### Stage 1 — Baseline stabilizer/QEC simulation

We will first reproduce several small Clifford-only circuits in Stim/LightStim to establish a working baseline. The circuits will contain gates such as H, S, CNOT, Pauli operations, and measurements. We will record circuit depth, gate count, measurement count, and simulation runtime. This step will also familiarize us with LightStim's circuit construction and QEC abstractions.

### Stage 2 — Clifford+T circuit front end

We will build or import small Clifford+T circuits and classify every operation as Clifford or non-Clifford. The initial benchmark range will be approximately **4–10 logical qubits** and **20–200 gates**, with several T-gate densities (for example 5%, 10%, 20%, and 30%). The compiler front end will record the original number of Clifford gates, T gates, two-qubit gates, circuit depth, and ancillas.

### Stage 3 — Pauli-based transformation

The core contribution will be a restricted PBC compiler prototype. We will implement Clifford conjugation rules for Pauli operators and maintain a classical Pauli frame. Supported Clifford segments will be propagated into Pauli-frame updates and measurement operators instead of being simulated as explicit logical gates whenever possible. T gates will be converted into magic-state/T-gadget resources, and the resulting computation will be represented as an adaptive sequence of Pauli measurements plus classical feed-forward information.

For each transformed circuit, the prototype will output:

- the generated Pauli-measurement sequence;
- the Pauli-frame/classical correction information;
- the number and weight of Pauli measurements;
- the number of remaining T/magic-state resources; and
- the corresponding stabilizer/QEC circuit that can be evaluated with Stim/LightStim where supported.

### Stage 4 — Magic-state resource model

The non-Clifford portion will be modeled as magic-state demand. The baseline model will assume one injected high-fidelity magic state per logical T resource and will estimate the number of states that must be supplied by a distillation layer. We will use published magic-state distillation results to define a simple cost model rather than implementing a complete physical factory. If time permits, a supported distillation circuit will be simulated in LightStim as a stretch goal.

## 3. Evaluation Plan

We will evaluate both **correctness** and **resource cost**.

First, for small noiseless examples, we will compare the output distribution or logical measurement results of the original Clifford+T circuit with the PBC-transformed representation. This establishes that the transformation preserves the intended computation.

Second, we will compare the original and transformed representations using:

- explicit Clifford-gate count;
- T count / magic-state count;
- number of Pauli measurements;
- average and maximum Pauli-measurement weight;
- circuit depth or measurement depth;
- ancilla count;
- simulation runtime; and
- estimated magic-state/distillation overhead.

The main expected result is that a large fraction of explicit Clifford evolution can be absorbed into Pauli-frame and measurement processing, while the T gates remain visible as the dominant non-Clifford fault-tolerant resource. We will also study how the measurement cost and magic-state demand scale with circuit size and T density.

## 4. Scope and Deliverables

The minimum viable project will support small Clifford+T circuits, implement a correct PBC transformation for the supported gate set, verify the transformed circuits, and report T/magic-state resource requirements. Full surface-code scheduling, detailed lattice-surgery routing, hardware-specific connectivity optimization, and a complete magic-state factory are outside the primary scope.

The final deliverables will include:

1. a documented Python implementation in this GitHub repository;
2. benchmark Clifford+T circuits and reproducible experiment scripts;
3. Stim/LightStim-based validation where applicable;
4. plots/tables comparing original and PBC representations;
5. a simple magic-state/distillation resource model; and
6. a final report summarizing correctness, resource trends, limitations, and future work.

## 5. Timeline

| Period | Planned work |
|---|---|
| Week 1 | Read Stim, LightStim, PBC, and magic-state references; reproduce baseline stabilizer circuits |
| Week 2 | Build Clifford+T parser/generator and baseline metrics |
| Week 3 | Implement Clifford-to-Pauli propagation and Pauli-frame tracking |
| Week 4 | Generate adaptive PBC measurement sequences and validate small examples |
| Week 5 | Add magic-state resource model and integrate supported workflows with LightStim |
| Week 6 | Run scaling experiments, generate plots, analyze results, and prepare the final report |

## References

1. C. Gidney, **“Stim: a fast stabilizer circuit simulator,”** *Quantum* 5, 497 (2021). https://doi.org/10.22331/q-2021-07-06-497
2. QuTone, **LightStim: A QEC Evaluator built on Stim**, GitHub repository. https://github.com/QuTone/LightStim
3. M. Yoganathan, R. Jozsa, and S. Strelchuk, **“Quantum advantage of unitary Clifford circuits with magic state inputs,”** *Proceedings of the Royal Society A* 475, 20180427 (2019). https://doi.org/10.1098/rspa.2018.0427
4. F. C. R. Peres and E. F. Galvão, **“Reducing Depth and Measurement Weights in Pauli-based Computation,”** *Physical Review A* 112, 062604 (2025). https://doi.org/10.1103/d3x5-cgky
5. D. Litinski, **“Magic State Distillation: Not as Costly as You Think,”** arXiv:1905.06903 (2019). https://arxiv.org/abs/1905.06903
6. Stim project repository and documentation. https://github.com/quantumlib/Stim
