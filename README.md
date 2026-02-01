# Quantum Entanglement Distillation for Network Edge Claiming

**MIT iQuHack 2026 — Team Report**

---

## 1. Problem Formulation

We consider a quantum network graph \( G(V, E) \) where edges represent noisy entanglement links.  
Each edge holds \( N \) identical Werner-like Bell pairs with initial fidelity \( F_0 < F_{\text{th}} \), where \( F_{\text{th}} \) is the threshold for claiming.

The fidelity is measured with respect to the maximally entangled Bell state:

$$
\lvert \Phi^+ \rangle = \frac{1}{\sqrt{2}} ( \lvert 00 \rangle + \lvert 11 \rangle ),
\qquad
F = |\langle \Phi^+ | \psi \rangle|^2
\tag{1}
$$

A noisy Bell pair under a depolarizing channel can be modeled as a Werner state:

\[
\rho_W = F \lvert \Phi^+ \rangle \langle \Phi^+ \rvert
+ \frac{1 - F}{3} \sum_{i \neq \Phi^+} \lvert B_i \rangle \langle B_i \rvert
\tag{2}
\]

where  
\(\{ B_i \} = \{ \lvert \Phi^\pm \rangle, \lvert \Psi^\pm \rangle \}\) is the Bell basis.

Our objective is to design **LOCC (Local Operations and Classical Communication)** circuits that distill high-fidelity entangled pairs from noisy inputs.

---

## 2. BBPSSW Protocol Implementation

We implement the **Bennett–Brassard–Popescu–Schumacher–Smolin–Wootters (BBPSSW)** protocol for \( N = 2 \) Bell pairs.

**Qubit pairing (outside-in convention):**
- Ancilla pair: \( (q_0, q_3) \)
- Data pair: \( (q_1, q_2) \)

### BBPSSW Distillation Protocol (\(N = 2\))

The protocol applies **bilateral CNOT gates** locally:

- Alice: \( \text{CNOT}_{1 \rightarrow 0} \)
- Bob: \( \text{CNOT}_{2 \rightarrow 3} \)

\[
U_{\text{LOCC}} =
(I_B \otimes \text{CX}_{2 \rightarrow 3})
\cdot
(\text{CX}_{1 \rightarrow 0} \otimes I_B)
\tag{3}
\]

After measuring ancilla qubits \( q_0 \) and \( q_3 \), we apply **post-selection** using a parity check:

\[
\text{flag} = c_0 \oplus c_1 = 0 \;\Rightarrow\; \text{accept}
\tag{4}
\]

For input Werner states with fidelity \( F_0 \), the output fidelity is:

\[
F_{\text{out}} =
\frac{F_0^2 + \frac{1}{9}(1 - F_0)^2}
{F_0^2 + \frac{2}{3}F_0(1 - F_0) + \frac{5}{9}(1 - F_0)^2}
\tag{5}
\]

The **success probability** is:

\[
P_{\text{succ}} =
F_0^2 + \frac{2}{3}F_0(1 - F_0) + \frac{5}{9}(1 - F_0)^2
\]

Distillation succeeds when \( F_0 > 0.5 \), yielding \( F_{\text{out}} > F_0 \).

---

## 3. DEJMPS Variant

We also implement a **Deutsch–Ekert–Jozsa–Macchiavello–Popescu–Sanpera (DEJMPS)** inspired variant, which handles phase errors more effectively.

The modification applies **Hadamard gates** to the ancilla qubits before the bilateral CNOTs:

\[
U_{\text{DEJMPS}} =
U_{\text{LOCC}} \cdot
(H_0 \otimes I \otimes I \otimes H_3)
\tag{6}
\]

This basis transformation allows the protocol to correct both:
- Bit-flip errors (\(X\))
- Phase-flip errors (\(Z\))

For asymmetric noise channels where \( p_X \neq p_Z \), this variant can outperform the standard BBPSSW protocol.

---

## 4. Competitive Strategy

The **claim strength** for a vertex is computed as:

\[
S = \sum_{i=1}^{n} \frac{F_i \cdot P_i}{\sqrt{r_i}}
\tag{7}
\]

where:
- \( F_i \) is the achieved fidelity
- \( P_i \) is the success probability
- \( r_i \) is the rank of edge \( i \), sorted by fidelity

Our automated solver:
- Prioritizes edges by **difficulty rating** and **threshold**
- Iteratively applies BBPSSW and DEJMPS protocols until \( F \geq F_{\text{th}} \)
- Does **not consume budget on failed attempts**, enabling aggressive exploration
- Preserves resources for high-value, successful claims

---

