# Max-Cut Optimization via Ising Model, Simulated Annealing and Bayesian Optimization

This project explores how physical-inspired stochastic solvers behave when solving NP-hard combinatorial optimization problems, focusing on the Max-Cut problem formulated as an Ising energy model.

The main goal is not to find an exact optimum, but to understand the solver as a physical system, analyze how its behavior depends on hyperparameters, and apply machine learning techniques to predict and optimize those parameters efficiently.

## Project Motivation

Max-Cut is a classical NP-hard combinatorial optimization problem that can be naturally mapped into an Ising Hamiltonian. This makes it an ideal minimal example—a kind of “Hello World” or harmonic oscillator—for studying stochastic optimization dynamics inspired by physics.

Instead of focusing solely on solution quality, this project treats the solver itself as an object of study:

- How does it explore the energy landscape?
- How does stochasticity affect outcomes?
- How can machine learning help predict good parameter choices?

## Problem Formulation

Given a weighted graph, the Max-Cut problem consists of partitioning the nodes into two groups such that the sum of weights of edges crossing the partition is maximized.
This is mapped to an Ising model by assigning each node a binary spin:

$$
s_i \in \{ -1, 1\}
$$

the energy of a configuration is defined as: 

$$
E(\textbf{s}) = \sum_{(i,j) \in E} J_{ij}s_i s_j 
$$

Minimizing this energy is equivalent to maximizing the cut value.

> Once the problem is formulated as a physical system, the central question shifts from “how do we find the best partition?” to “how can the system efficiently explore its energy landscape?”.  I found this change of perspective fascinating.

## Solver Dynamics: Simulated Annealing

The system evolves using Simulated Annealing (SA) with a Metropolis acceptance rule, inspired by physical cooling processes.

At each step:
- A random spin flip is proposed
- The energy change $\Delta E$ is computed
- The move is accepted based on temperature-dependent probability

The temperature schedule is linear:
$$
T(t) = T_0 - (T_0 - T_f)\dfrac{t}{N_{steps}}
$$