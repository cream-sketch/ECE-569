# Project Proposal: Logical T-State Injection and Resource Estimation for Pauli-Based QEC Simulation

**Course:** CSC/ECE 569 Quantum Computing  
**Project ID:** QEC Simulation-(3) Pauli-Based Computation  
**Team members:** Yongxiang Lyu, Sheng Chen, Zhuozi Xie  
**Repository:** https://github.com/cream-sketch/ECE-569

## 1. Motivation and Problem Definition

Pauli-based computation (PBC) separates the easy and hard parts of fault-tolerant quantum computation. Clifford operations and Pauli measurements remain in the stabilizer domain and can be simulated efficiently by tools such as Stim/LightStim, while the non-Clifford content of a Clifford+T circuit appears as T-type magic-state resources. Once these T resources are exposed, the practical problem is how to supply and consume them fault tolerantly.

This project therefore focuses on two concrete questions:

1. **Logical T-state preparation/injection:** how can a prepared magic state be consumed through a Clifford-and-measurement gadget to realize a logical T gate?
2. **Magic-state resource estimation:** given T-count/T-depth and a target logical error, how many distillation rounds, raw states, and potentially parallel factories are required?

The first problem establishes the execution primitive; the second quantifies its cost. We intentionally do not implement a complete PBC compiler or full surface-code architecture.

## 2. Problem 1 — From a Magic State to a Logical T Gate

The T gate is T = diag(1, exp(i*pi/4)), with magic state

|T> = T|+> = (|0> + exp(i*pi/4)|1>)/sqrt(2).

An ideal |T> can be generated using an ideal T gate for verification, but that is only a reference model. In a fault-tolerant implementation, the non-stabilizer resource must be prepared independently and then injected into the data path.

We will implement a standard T-injection gadget:

1. prepare an ancilla in |T>;
2. apply CNOT(data -> ancilla);
3. measure the ancilla in the Z basis;
4. for outcome 1, apply an S correction to the data qubit.

For outcome 0, the data becomes T|psi>. For outcome 1, the uncorrected branch is T^dagger|psi> up to global phase, and the Clifford S correction yields T|psi>. Thus, once the magic state is available, the online operation uses only Clifford gates, measurement, and classical feed-forward.

### Why preparation matters

The difficult part is obtaining a sufficiently accurate logical magic state. The conventional route is **magic-state distillation**, where many noisy T states are consumed to create fewer high-fidelity states. A complementary route is **code switching**. Daguerre et al. experimentally prepared high-fidelity logical magic states by switching from a 15-qubit quantum Reed–Muller code to the 7-qubit Steane code, showing that logical magic-state preparation can also be realized directly at the encoded level.

Our implementation will not reproduce that code-switching experiment. Instead, we use it to motivate treating magic-state preparation as an independent subsystem whose output feeds the same injection primitive. For our baseline experiments, we will use ideal and parametrically noisy |T> resource states and focus on injection correctness.

### Experiment A — Injection correctness

Using Qiskit/statevector simulation, we will compare a direct ideal T gate with the |T>-injection gadget for |0>, |1>, |+>, |->, and random one-qubit states. We will verify both measurement branches and report output-state fidelity. A small multi-T circuit will test composition.

Stim/LightStim will be used for the surrounding Clifford/measurement/QEC structure rather than for directly representing the non-stabilizer amplitudes of |T>.

## 3. Problem 2 — Resource Estimation for Supplying T States

Once the T resources are identified, the next question is their production cost. We will use **15-to-1 T-state distillation** as the baseline model. For sufficiently small input error p,

p_out ~= 35 p^3.

For repeated rounds,

p_(r+1) ~= 35 (p_r)^3.

Ignoring rejection/retry overhead in the first-order model, each round multiplies the raw-state demand by 15:

N_raw/output ~= 15^r.

For example, starting from p_raw = 10^-2, one round gives approximately 3.5e-5. A second round suppresses the error much further, but raises the idealized raw-state cost from 15 to 225 states per final T state.

### Estimator inputs

- circuit T-count N_T;
- circuit T-depth D_T, or an equivalent T-consumption schedule;
- raw magic-state error p_raw;
- target output error p_target, or an algorithm-level failure budget epsilon.

A simple per-T requirement such as p_target <= epsilon/N_T can be used as a first-order reliability constraint.

### Estimator outputs

- minimum distillation rounds;
- output error after each round;
- raw states required per final T state;
- total raw states required for the circuit;
- number of high-fidelity T states consumed;
- under simple timing assumptions, required factory throughput or parallelism.

This last quantity is motivated by Litinski's surface-code resource analysis. Magic-state factories create a direct **space-time tradeoff**: more parallel factories consume more qubits but can supply T states fast enough to avoid stalling a T-heavy computation, while fewer factories reduce space at the cost of longer runtime.

### Experiment B — Resource scaling

We will sweep representative values such as N_T = 10, 50, 100, 500, 1000; raw-state errors of 10^-2 and 10^-3; several target-error budgets; and selected T-depth/consumption-rate scenarios.

The main plots will show:

1. distillation rounds vs. target error;
2. total raw magic states vs. T-count;
3. total raw magic states vs. target fidelity;
4. factory parallelism or latency vs. T-consumption rate.

## 4. Scope and Expected Outcome

The project starts after PBC or a Clifford+T analysis has exposed the T resources. We will not implement a complete PBC compiler, the code-switching protocol itself, a full lattice-surgery scheduler, or a detailed physical magic-state factory.

The expected workflow is:

T resource identified -> injection verified -> required state fidelity specified -> resource cost estimated.

Minimum deliverables are:

1. T-state/injection code and correctness tests;
2. a 15-to-1 distillation resource estimator;
3. optional T-depth/factory-throughput estimates;
4. small Clifford+T examples;
5. resource-scaling plots/tables.

## 5. Timeline

| Period | Planned work |
|---|---|
| Week 1 | Review PBC, T injection, code switching, and 15-to-1 distillation |
| Week 2 | Implement ideal/noisy |T> model and injection; verify both branches |
| Week 3 | Test multi-T examples and surrounding Clifford/measurement simulation |
| Week 4 | Implement distillation error and raw-state resource estimator |
| Week 5 | Add T-depth/factory-throughput model and run scaling experiments |
| Week 6 | Analyze results, generate plots, and prepare final report |

## References

1. C. Gidney, “Stim: a fast stabilizer circuit simulator,” Quantum 5, 497 (2021).
2. QuTone, “LightStim: A QEC Evaluator built on Stim,” GitHub repository.
3. S. Bravyi and A. Kitaev, “Universal quantum computation with ideal Clifford gates and noisy ancillas,” Physical Review A 71, 022316 (2005).
4. F. C. R. Peres and E. F. Galvão, “Quantum circuit compilation and hybrid computation using Pauli-based computation,” arXiv:2203.01789.
5. D. Litinski, “Magic State Distillation: Not as Costly as You Think,” arXiv:1905.06903 (2019).
6. D. Litinski, “A Game of Surface Codes: Large-Scale Quantum Computing with Lattice Surgery,” Quantum 3, 128 (2019).
7. L. Daguerre, R. Blume-Kohout, N. C. Brown, D. Hayes, and I. H. Kim, “Experimental Demonstration of High-Fidelity Logical Magic States from Code Switching,” Physical Review X 15, 041008 (2025), arXiv:2506.14169.
