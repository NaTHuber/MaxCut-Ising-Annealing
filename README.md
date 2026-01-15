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

All parameters are kept fixed except the initial temperature $T_0$, which is the focus of the study.

## Experimental Design

For each value of $T_0$

```mermaid 
    graph LR
    A[1 <br> Initialize random spin configurations]
    B[2 <br> Run Simulated Annealing]
    C[3 <br> Record: Best energy achieved, Final energy, Best cut value]
    D[4 <br> Repeat multiple stochastic runs]

    A --> B
    B --> C
    C --> D

    style A fill: #7067a4ff
    style B fill: #7067a4ff
    style C fill: #7067a4ff
    style D fill: #7067a4ff

```


This allows us to capture both mean performance and intrinsic variability of the solver.

## Project Phases 

```mermaid
    graph LR
    A[Phase 1 – Baseline Temperature Analysis]
    B[Phase 2 - Supervised Regression for Parameter Prediction]
    C[Phase 3 – Bayesian Optimization]
    A --> B
    B --> C

    style A fill: #318291
    style B fill: #318291
    style C fill: #318291
```
### Phase 1 – Baseline Temperature Analysis
Notebook: `01_baseline_temperature_analysis.ipynb`

Key observations:
- Low $T_0$: premature freezing, poor exploration 
- Intermediate $T_0$: it gave the best balance between exploration and exploitation 
- Hight $T_0$: excessive noise, degraded convergence

This reveals a non-trivial dependency between solver performance and temperature.

### Phase 2 - Supervised Regression for Parameter Prediction
Notebook: `02_supervised_regression_parameter_prediction.ipynb`
Each stochastic run is treated as a data point, forming a supervised dataset:
- input: $\log(T_0)$
- Target: best cute value 

Three regression models are trained (Linear, Polynomial, Ridge), showing that:

- Linear models underfit
- Low-degree polynomial models capture the non-linear trend
- Performance is inherently noisy, reflecting solver stochasticity

This step reframes parameter selection as a supervised learning problem.

### Phase 3 – Bayesian Optimization
Notebook: `03_bayesian_optimization.ipynb`

Bayesian Optimization is applied to efficiently search the parameter space:

- A Gaussian Process (GP) models solver performance and uncertainty
- Expected Improvement (EI) balances exploration and exploitation
- The method identifies promising temperature regions without exhaustive search

This approach mirrors real-world scenarios where:

- Evaluations are expensive
- The objective function is unknown
- Uncertainty must be explicitly modeled

