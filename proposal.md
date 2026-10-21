# Project Proposal: T-State Injection and Magic-State Resource Estimation for Pauli-Based QEC Simulation

**Course:** CSC/ECE 569 Quantum Computing  
**Project ID:** QEC Simulation-(3) Pauli-Based Computation  
**Team members:** Yongxiang Lyu, Sheng Chen, Zhuozi Xie  
**Repository:** https://github.com/cream-sketch/ECE-569

## 1. Motivation and Focus

Stim and LightStim are well suited to stabilizer/QEC circuits containing Clifford operations, Pauli measurements, and Pauli noise. Universal Clifford+T computation, however, also requires the non-Clifford T gate, which is outside ordinary stabilizer simulation. Pauli-based computation motivates separating these two parts: Clifford processing remains in the stabilizer/measurement domain, while the non-Clifford content appears as T-type magic-state resources.

Rather than implementing a complete PBC compiler, this project focuses on two concrete problems at this stabilizer/non-Clifford boundary:

1. **T-state preparation and T-gate injection:** how to replace an explicit T gate with a prepared magic state plus Clifford operations, measurement, and feed-forward.
2. **Magic-state resource estimation:** given a circuit T-count and a target T-state error, how many distillation rounds and raw magic states are needed.

This scope gives the project one implementation problem and one quantitative resource-analysis problem.

## 2. Problem 1 — T-State Preparation and T-Gate Injection

The T gate is T = diag(1, exp(i*pi/4)). Its corresponding magic state is |T> = T|+> = (|0> + exp(i*pi/4)|1>)/sqrt(2). Preparing |T> by simply applying an ideal T gate is useful as a mathematical reference, but it does not solve the fault-tolerant problem because it assumes the non-Clifford operation that must ultimately be supplied fault-tolerantly. In a fault-tolerant workflow, the non-stabilizer resource is prepared offline, purified/distilled when necessary, and consumed through magic-state injection.

We will implement and verify a standard T-injection gadget. Given a data qubit |psi> and an ancilla in |T>, the gadget performs:

1. prepare the ancilla resource in |T>;
2. apply CNOT(data -> ancilla);
3. measure the ancilla in the Z basis;
4. if the measurement result is 1, apply an S correction to the data qubit.

For measurement outcome 0, the data becomes T|psi>. For outcome 1, the uncorrected branch is T^dagger|psi> up to global phase; applying the Clifford S correction yields T|psi>. Thus, once the magic state is available, the remaining online gadget uses only Clifford operations, measurement, and classical feed-forward.

### Implementation plan

- **Qiskit/statevector verification:** generate the ideal |T> reference state and compare the injection gadget with a direct ideal T gate.
- **Stim/LightStim interface:** represent the surrounding Clifford gates, measurements, Pauli-frame corrections, and QEC circuitry. We will treat the magic state itself as the non-Clifford resource boundary instead of claiming that Stim directly represents its non-stabilizer amplitudes.
- **Optional noisy-resource model:** attach an input magic-state error probability p_raw and study the logical effect of imperfect resource states.

### Correctness experiment

For |0>, |1>, |+>, |->, and several random one-qubit states, we will compare a direct ideal T gate with the injection gadget. The primary metric is output-state fidelity. In the noiseless case, the expected fidelity is 1 up to numerical precision. We will also verify both measurement branches and their feed-forward correction. A small circuit containing multiple T gates will test whether several injection gadgets compose correctly.

This experiment answers a precise implementation question: **Can an explicit T operation be replaced by a magic-state injection primitive while leaving the online execution in Clifford/measurement form?**

## 3. Problem 2 — Magic-State Distillation Resource Estimation

After PBC or another Clifford+T analysis identifies the number of T resources, the next problem is their production cost. We will build a resource estimator around one standard protocol, 15-to-1 T-state distillation, instead of modeling every possible factory.

For sufficiently small input error p, a standard leading-order model is:

p_out ~= 35 * p^3

For repeated rounds:

p_(r+1) ~= 35 * (p_r)^3

Ignoring rejection/retry overhead in the first-order model, each round multiplies raw-state demand by 15:

raw states per distilled output ~= 15^r

### Estimator inputs

- circuit T-count N_T;
- raw magic-state error probability p_raw;
- required target magic-state error p_target;
- optionally, an algorithm-level failure budget epsilon, from which a simple per-T target such as epsilon / N_T can be derived.

### Estimator outputs

- minimum number of distillation rounds;
- estimated output error after each round;
- raw T states required per high-fidelity output;
- total raw T states required for the circuit;
- number of high-fidelity T states consumed, equal to N_T;
- and, when simple timing/parallelism assumptions are supplied, an estimated latency or factory throughput.

For example, if p_raw = 10^-2, one idealized 15-to-1 round gives p_out ~= 3.5e-5. A second round suppresses the error much further but raises the ideal raw-state cost from 15 to 225 input states per final output. This exposes the tradeoff we want to quantify: stronger fidelity requirements can rapidly increase resource demand.

## 4. Evaluation

### Experiment A — Injection correctness

We will report direct-T vs. injected-T fidelity, correctness of both measurement branches, Clifford gate count, measurement count, correction count, and magic states consumed per injected T. A small multi-T circuit will provide an end-to-end sanity check.

### Experiment B — Distillation resource scaling

We will sweep representative T-counts N_T = 10, 50, 100, 500, 1000, several raw-state error rates such as 10^-2 and 10^-3, and multiple target error levels. The main plots/tables will show:

1. distillation rounds vs. target error;
2. raw magic states vs. T-count;
3. raw magic states vs. target fidelity;
4. total magic-state cost for circuits with different T-counts.

The estimator will also be applied to a few small Clifford+T benchmark circuits by counting their T gates and converting that count into a magic-state requirement.

## 5. Scope and Deliverables

The project intentionally does **not** attempt to implement a complete PBC compiler, a full surface-code architecture, lattice-surgery scheduling, or a detailed physical magic-state factory. PBC provides the motivation and interface: it isolates the T resources, and our work starts at that boundary.

Minimum deliverables:

1. Python implementation of T-state preparation/reference generation and T-gate injection;
2. correctness tests comparing direct and injected T operations;
3. a 15-to-1 magic-state distillation resource estimator;
4. experiments showing how T-count, raw-state quality, and target fidelity affect resource cost;
5. small Clifford+T integration examples;
6. documentation and plots in the repository.

## 6. Timeline

| Period | Planned work |
|---|---|
| Week 1 | Review Stim/LightStim, T injection, and 15-to-1 distillation |
| Week 2 | Implement ideal |T> reference and T-injection gadget; verify both branches |
| Week 3 | Test multi-T examples and connect Clifford/measurement portions to Stim/LightStim |
| Week 4 | Implement 15-to-1 error/cost recurrence and resource estimator |
| Week 5 | Sweep T-count/error targets and generate resource plots |
| Week 6 | Integrate results, document limitations, and prepare final report |

## References

1. C. Gidney, “Stim: a fast stabilizer circuit simulator,” Quantum 5, 497 (2021).
2. QuTone, “LightStim: A QEC Evaluator built on Stim,” GitHub repository.
3. S. Bravyi and A. Kitaev, “Universal quantum computation with ideal Clifford gates and noisy ancillas,” Physical Review A 71, 022316 (2005).
4. F. C. R. Peres and E. F. Galvao, “Quantum circuit compilation and hybrid computation using Pauli-based computation,” arXiv:2203.01789.
5. D. Litinski, “Magic State Distillation: Not as Costly as You Think,” arXiv:1905.06903 (2019).