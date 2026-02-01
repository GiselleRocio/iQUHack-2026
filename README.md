# Quantum Entanglement Distillation for Network Edge Claiming

**MIT iQuHack 2026 — Team Report**

---

## 1. Problem Formulation

We consider a quantum network graph $G(V, E)$ where edges represent noisy entanglement links. Each edge holds $N$ identical Werner-like Bell pairs with initial fidelity $F_0 < F_{\mathrm{th}}$, where $F_{\mathrm{th}}$ is the threshold for claiming.

The fidelity is measured with respect to the maximally entangled Bell state $|\Phi^+\rangle = (|00\rangle + |11\rangle)/\sqrt{2}$, and the fidelity is $F = |\langle \Phi^+|\psi\rangle|^2$.

A noisy Bell pair under a depolarizing channel can be modeled as a Werner state:
$\rho_W = F\,|\Phi^+\rangle\langle\Phi^+| + \frac{1-F}{3}\sum_{i\neq \Phi^+} |B_i\rangle\langle B_i|$,
where $\{B_i\} = \{|\Phi^\pm\rangle, |\Psi^\pm\rangle\}$ is the Bell basis.

Our objective is to design LOCC (Local Operations and Classical Communication) circuits that distill high-fidelity pairs from noisy inputs.

---

## 2. BBPSSW Protocol Implementation

We implement the Bennett–Brassard–Popescu–Schumacher–Smolin–Wootters (BBPSSW) protocol for $N=2$ Bell pairs. The qubit pairing follows the outside-in convention: pair 1 uses qubits $(q_0, q_3)$ as ancilla, and pair 2 uses $(q_1, q_2)$ as the data pair.

### BBPSSW Distillation Protocol ($N=2$)

Circuit operations: apply bilateral CNOT gates locally—Alice performs $\mathrm{CNOT}_{1\to 0}$ and Bob performs $\mathrm{CNOT}_{2\to 3}$. In compact form,
$U_{\mathrm{LOCC}} = (I_B \otimes \mathrm{CX}_{2\to 3})\cdot(\mathrm{CX}_{1\to 0}\otimes I_B)$.

After measuring ancilla qubits $q_0$ and $q_3$, we apply post-selection and keep only outcomes where the parity is even:
$\mathrm{flag} = c_0 \oplus c_1 = 0 \Rightarrow \mathrm{accept}$.

For input Werner states with fidelity $F_0$, the output fidelity is
$F_{\mathrm{out}} = \dfrac{F_0^2 + \frac{1}{9}(1-F_0)^2}{F_0^2 + \frac{2}{3}F_0(1-F_0) + \frac{5}{9}(1-F_0)^2}$.

The success probability is
$P_{\mathrm{succ}} = F_0^2 + \frac{2}{3}F_0(1-F_0) + \frac{5}{9}(1-F_0)^2$.

Distillation succeeds when $F_0 > 0.5$, yielding $F_{\mathrm{out}} > F_0$.

---

## 3. DEJMPS Variant

We also implement a Deutsch–Ekert–Jozsa–Macchiavello–Popescu–Sanpera (DEJMPS) inspired variant that handles phase errors more effectively. The modification applies Hadamard gates to the ancilla qubits before the bilateral CNOTs:
$U_{\mathrm{DEJMPS}} = U_{\mathrm{LOCC}}\cdot(H_0 \otimes I \otimes I \otimes H_3)$.

This transforms the noise basis, enabling the protocol to correct both bit-flip ($X$) and phase-flip ($Z$) errors simultaneously. For asymmetric noise channels where $p_X \ne p_Z$, this variant can outperform standard BBPSSW.

---

## 4. Competitive Strategy

The claim strength for a vertex is computed as
$S = \sum_{i=1}^{n}\dfrac{F_i \cdot P_i}{\sqrt{r_i}}$,
where $F_i$ is the achieved fidelity, $P_i$ is the success probability, and $r_i$ is the rank of edge $i$ (sorted by fidelity).

Our automated solver prioritizes edges by difficulty rating and threshold, iteratively applying both protocols until $F \ge F_{\mathrm{th}}$. Failed attempts do not consume budget, enabling aggressive exploration of the solution space while preserving resources for successful claims.
