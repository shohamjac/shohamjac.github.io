
---
title: Finding the Circuit-Level Distance of a Code Using Integer Linear Programming
author: jacoby
date: 2025-10-04 11:33:00 +0200
categories: [Quantum Computing]
tags: [Quantum Computing, Quantum Error Correction]
pin: true
math: true
mermaid: true
---


Circuit-level distance is an important measure in fault tolerant quantum computing. It is the number of circuit-level faults that can cause an undetected logical error, and it directly affects the logical error rate of the circuit.

However, finding this circuit-level distance is generally known to be a hard problem.

Common approaches are:
- Asking a decoder to find the lowest weight error that flips a logical observable. This only gives you a lower bound on the circuit-level distance, as the decoder (generally) does not guarantee finding the lowest weight error.
- Converting the problem into a max-SAT problem, as explained in this QCSE post by Gidney. In my experience with the open-source max-SAT solvers, this took a really long time for anything beyond basic circuits.
- There is also the stim heuristic method `search_for_undetectable_logical_errors` which works quite well but does not guarantee anything about the result.

Thus, I needed a better approach.

Integer linear programming (ILP) is an optimization problem that was actually used in a few other projects to find the distance of a stabilizer code.

The distance of a stabilizer code is the lowest-weight non-stabilizer Pauli that commutes with the stabilizer group of a code. It is not exactly what I was looking for, but it was close enough to give it a try.

Let's start by formulating the ILP problem.

>Solve for the minimum number of DEM error mechanisms whose combined effect:
> -  flips NO detectors (all detectors have even parity)
> - flips AT LEAST ONE logical observable (some logical has odd parity)


Let $x\in\{0,1\}^n$ be a vector that controls whether an error mechanism is included or not in the combination of error mechanisms. 
Also, let $w\in\{0,1\}^k$ be the vector that controls whether a logical is flipped or not by this error mechanism combination.

We want to minimize ${||x||}_1$ while ${||w||}_1\ge1$.

Under the first constraint:
$$
\sum_j D_{i,j} \cdot x_j = 0 \mod{2}
$$
with $D_{i,j}\in\{0,1\}$ being the effect of error mechanism $j$ on detector $i$ (i.e., $D_{i,j}=1$ iff error $j$ activates detector $i$).

and under the second one:
$$
\sum_j L_{r,j} \cdot x_j = w_r \mod{2}
$$
with $L_{r,j}$ being the effect of error mechanism $j$ on logical $r$.

We can easily use python-mip or another ILP library to formulate those constraints, and use this to find the circuit distance of every circuit (probably written in stim, as this is the leading library for Clifford simulations at the moment).

The full implementation is given in this [repo](https://github.com/shohamjac/ILP-circuit-distance/tree/main).

To benchmark, we will use the Bivariate Bicycle code implementation given [here](https://quantumcomputing.stackexchange.com/questions/37289/compute-the-exact-minimum-distance-of-a-qecc-with-integer-linear-programming-met) by Craig Gidney.

We get the following results:
```
Solver: CBC
 | distance:  12
 | optimal:  True
 | time:  78.36955916206352

Solver: GRB
 | distance:  12
 | optimal:  True
 | time:  3.3316898189950734
```

It can be seen that both solvers give an exact solution quite fast, **with the Gurobi one obtaining the result in less than 4 seconds**.

Using this method, I was able to calculate the circuit level distance of much more complicated circuits. Soon to be published.