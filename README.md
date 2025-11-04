# QuantumJam 2025: BB84 QKD on Noisy Simulators

Simulates the **BB84** quantum key distribution protocol in Qiskit under realistic
hardware noise, uses the Quantum Bit Error Rate (QBER) to detect an eavesdropper,
and derives a symmetric encryption key from what survives.

Built at **QuantumJam 2025** (ITBA and IBM Quantum, Qiskit Fall Fest), Argentina's
first quantum computing hackathon, by team *Planck Sinatra*: Juan Quiroga Bonetto,
Naomi Couriel, Ana Paula Tissera and Athina Salim.

## The problem: RSA's decline and the two answers to it

Public-key cryptography like RSA rests on mathematics that is intractable for
classical computers. Shor's algorithm makes it trivial for a quantum one, which is
an existential threat to secure communication. Two answers have been proposed:

- **Post-quantum cryptography (PQC).** Classical algorithms believed to resist
  quantum attack.
- **Quantum key distribution (QKD).** Protocols that distribute a key using
  quantum physics, where security rests on the laws of nature rather than on an
  assumption about what is hard to compute.

This project implements BB84, the cornerstone of QKD, and then asks the question
that decides whether it is usable in practice: does it still hold up once the
hardware is noisy?

## What the notebook does

| Phase | What happens |
|---|---|
| Quantum exchange | Alice encodes 200 random bits in random bases, Bob measures in his own random bases. |
| Sifting | Alice and Bob publicly compare bases and keep only the positions where they agree, roughly half. |
| QBER estimation | They sacrifice 50 sifted bits to measure the error rate. Above an 11% threshold the channel is treated as compromised. |
| Reconciliation | Cascade, with shuffled passes and recursive binary search, removes the errors that noise left behind. |
| Privacy amplification | XOR folding shrinks the key, destroying whatever partial information an eavesdropper might hold. |
| Key derivation | PBKDF2 turns the surviving key into an AES-256 key. |

Noise comes from Qiskit Aer's `NoiseModel`: amplitude damping, phase damping,
depolarising and readout error. An explicit identity gate is inserted so that idle
qubits decohere as well, instead of passing through the circuit untouched.

## Results

Three scenarios, 200 qubits each:

| Scenario | QBER | Verdict |
|---|---|---|
| Ideal channel | 0.00% | secure |
| Realistic noise (1% amplitude damping, 1.5% depolarising, 5% readout) | 2.00% | secure |
| Eve intercepting every qubit | 24.00% | eavesdropper detected |

Intercept-and-resend drives QBER to about a quarter of the sifted bits, matching
the theoretical 25%, and lands far above the 11% threshold. Noise on its own stays
well below it, so the protocol still separates a merely noisy channel from an
attacked one, which is the whole point.

Broken down by noise channel, the contribution to QBER is uneven:

| Channel | QBER |
|---|---|
| Depolarising | 6.00% |
| Classical channel | 5.15% |
| Amplitude damping | 4.00% |
| Readout | 2.00% |
| Phase damping | 0.00% |

Adding idle-qubit decoherence and two-qubit gate error on top of the base model
pushes QBER from 8% to 10%, still under threshold but visibly closer to it.

The notebook closes with an E91 proof of concept, entangled pairs and a CHSH test,
and a short comparison of QKD against post-quantum cryptography.

## Running it

Open [`bb84_qkd.ipynb`](bb84_qkd.ipynb) in Colab with the badge at the top of the
notebook, or run it locally:

```bash
pip install "qiskit[visualization]" qiskit-ibm-runtime qiskit-aer qiskit_qasm3_import
jupyter notebook bb84_qkd.ipynb
```

The challenge prompt we were given is in
[`docs/challenge-prompt.pdf`](docs/challenge-prompt.pdf).

## Authors

[Juan Quiroga Bonetto](https://github.com/Juanchi2112),
[Naomi Couriel](https://github.com/naomicouriel),
[Ana Paula Tissera](https://github.com/anatissera) and
[Athina Salim](https://github.com/athinasalimm).
