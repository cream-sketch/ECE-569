# ECE 569 — Pauli-Based QEC Simulation

Course project repository for **CSC/ECE 569 Quantum Computing**.

## Project

**Extending Stabilizer-Based QEC Simulation with Pauli-Based Computation for Clifford+T Circuits**

This project studies how Clifford+T circuits can be transformed into a Pauli-based, measurement-driven representation that is compatible with stabilizer/QEC simulation. The main prototype will:

- classify Clifford and T operations;
- propagate Pauli operators through Clifford gates;
- track Pauli-frame updates classically;
- generate Pauli-measurement sequences;
- isolate T gates as magic-state resources;
- validate supported circuits with Stim/LightStim; and
- estimate magic-state/distillation overhead.

## Proposal

See [proposal.md](proposal.md) for the two-page project proposal, methodology, evaluation plan, timeline, and references.

## Planned Repository Structure

```text
ECE-569/
├── README.md
├── proposal.md
├── src/
│   ├── compiler/
│   └── resource_model/
├── benchmarks/
├── experiments/
├── results/
└── docs/
```

## Status

Proposal and project setup in progress.
