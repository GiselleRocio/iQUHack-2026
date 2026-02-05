# Quantum Entanglement Distillation for Network Edge Claiming

*MIT iQuHack 2026 — Team Report*

## Problem Formulation

We consider a quantum network graph $G(V,E)$ where edges represent noisy entanglement links. Each edge holds $N$ identical Werner-like Bell pairs with initial fidelity $F_0 < F_{\text{th}}$, where $F_{\text{th}}$ is the threshold for claiming. The fidelity is measured with respect to the maximally entangled state:

$$\ket{\Phi^+} = \frac{1}{\sqrt{2}}\left(\ket{00} + \ket{11}\right), \quad F = \left|\braket{\Phi^+|\psi}\right|^2$$

A noisy Bell pair under depolarizing channel can be modeled as a Werner state:

$$\rho_W = F\ket{\Phi^+}\bra{\Phi^+} + \frac{1-F}{3}\sum_{i \neq \Phi^+}\ket{B_i}\bra{B_i}$$

where $\{B_i\} = \{\ket{\Phi^\pm}, \ket{\Psi^\pm}\}$ is the Bell basis. Our objective is to design LOCC (Local Operations and Classical Communication) circuits that distill high-fidelity pairs from noisy inputs.

## BBPSSW Protocol Implementation

We implement the Bennett-Brassard-Popescu-Schumacher-Smolin-Wootters (BBPSSW) protocol for $N=2$ Bell pairs. The qubit pairing follows the outside-in convention: pair 1 uses qubits $(q_0, q_3)$ as ancilla, and pair 2 uses $(q_1, q_2)$ as the data pair.

**Circuit Operations:** Apply bilateral CNOT gates locally—Alice performs $\text{CNOT}_{1\to 0}$ and Bob performs $\text{CNOT}_{2\to 3}$:

$$U_{\text{LOCC}} = (I_B \otimes \text{CX}_{2\to 3}) \cdot (\text{CX}_{1\to 0} \otimes I_B)$$

After measuring ancilla qubits $q_0$ and $q_3$, we apply **post-selection**: keep only outcomes where $c_0 \oplus c_1 = 0$ (parity check):

$$\text{flag} = c_0 \oplus c_1 = 0 \implies \text{accept}$$

For input Werner states with fidelity $F_0$, the output fidelity is:

$$F_{\text{out}} = \frac{F_0^2 + \frac{1}{9}(1-F_0)^2}{F_0^2 + \frac{2}{3}F_0(1-F_0) + \frac{5}{9}(1-F_0)^2}$$

The success probability is $P_{\text{succ}} = F_0^2 + \frac{2}{3}F_0(1-F_0) + \frac{5}{9}(1-F_0)^2$. Distillation succeeds when $F_0 > 0.5$, yielding $F_{\text{out}} > F_0$.

## DEJMPS Variant

We also implement a Deutsch-Ekert-Jozsa-Macchiavello-Popescu-Sanpera (DEJMPS) inspired variant that handles phase errors more effectively. The modification applies Hadamard gates to ancilla qubits before the bilateral CNOTs:

$$U_{\text{DEJMPS}} = U_{\text{LOCC}} \cdot \left(H_0 \otimes I \otimes I \otimes H_3\right)$$

This transforms the noise basis, enabling the protocol to correct both bit-flip ($X$) and phase-flip ($Z$) errors simultaneously. For asymmetric noise channels where $p_X \neq p_Z$, this variant can outperform standard BBPSSW.

## Competitive Strategy

The claim strength for a vertex is computed as:

$$S = \sum_{i=1}^{n} \frac{F_i \cdot P_i}{\sqrt{r_i}}$$

where $F_i$ is the achieved fidelity, $P_i$ is success probability, and $r_i$ is the rank of edge $i$ (sorted by fidelity). Our automated solver prioritizes edges by difficulty rating and threshold, iteratively applying both protocols until $F \geq F_{\text{th}}$. Failed attempts do not consume budget, enabling aggressive exploration of the solution space while preserving resources for successful claims.

## Key Discovery: Difficulty-Dependent Fidelity Limits

Systematic testing reveals that each difficulty level corresponds to an effective noise model that fixes a maximum achievable fidelity $F_{\max}$. The relationship between difficulty and conquerable status is summarized below:

| Difficulty | Noise Model | $F_{\max}$ | Conquerable? |
|------------|-------------|-------|--------------|
| 1 | Light phase ($Z$) noise | 0.9698 | Yes |
| 2 | Depolarizing (~43%) | 0.8000 | No |
| 3 | Heavy phase noise | 0.9000 | No (fixed) |
| $\geq 4$ | Mixed / heavy noise | $<0.90$ | No |

For difficulties $D \geq 2$, failure is not algorithmic but *mathematical*: no single-pass LOCC protocol can exceed the required fidelity threshold $F_{\text{th}} = 0.90$.

### Difficulty Level Details

**D = 1 (BBPSSW):** For $D=1$ edges, noise is dominated by phase ($Z$) errors. We apply BBPSSW with $N=2$ pairs, achieving $F = 0.9698$ with success probability $P \approx 0.745$. The CNOT structure extracts phase parity into ancilla qubits, while post-selection optimally matches the noise model.

**D = 2 (Depolarizing):** Extensive testing with DEJMPS, BBPSSW variants, and entanglement pumping consistently converged to:

$$F_{\max} = \frac{F^2}{F^2 + (1-F)^2} \approx 0.80$$

This lies strictly below 0.90, rendering $D=2$ edges unconquerable.

**D = 3:** BBPSSW distillation converges exactly to $F = 0.9000$. Since the game enforces $F > 0.90$, these edges are also impossible to claim.
