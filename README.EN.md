---

DMEI: Distributed Multi-valued Edge Intelligence Architecture

Document Version: v3.3
Project Nature: Theoretical specification (No code, no implementation)
Target Audience: In-memory computing chip designers, edge computing engineers, asynchronous inference researchers

---

1. Design Context

Current LLM response latency is constrained by software-level Token preprocessing and centralized training. DMEI shifts signal processing down to the physical layer, utilizing multi-valued gate arrays for direct sampling. Inference and updates are decoupled physically. No default parameters are set; no ethical preferences are embedded.

2. System Layering

2.1 Physical Sensing Layer

Hardware: Multi-valued logic gate arrays based on RRAM or FeFET.
Function: Slices input signals via fixed clock windows, generates fragments, and appends a monotonic sequence number. On signal loss, inserts a vacancy length count. Fragments are transmitted upstream without tokenization or semantic parsing.

Design Rationale: Multi-valued gates achieve sufficient encoding density once state counts surpass a physical threshold. Slicing relies on randomization; by retaining the sequence number and vacancy length count, the upper layer can locate signal loss even when interruptions occur, eliminating the need for semantic judgment at the physical layer.

---

2.1.1 Low-Level Sampling and Frame Protocol

A. Sampling
Gate arrays sample at a fixed clock cycle. Slicing is based on fixed sample counts; continuous signals are sliced sequentially.

B. Order Locking
Each fragment is assigned a monotonic sequence number. The upper layer sorts strictly by this sequence number, locking the physical order.

C. Vacancy Length and Space Boundary Flag
An integrated interrupt trigger detects breakpoints when continuous clock cycles fall below the physical threshold. A placeholder is inserted, containing:

· Vacancy Length Count (duration of the interruption);
· Space Boundary Flag (whether the breakpoint crosses a physical space).

D. Frame Format

```
[Header] | [Seq ID] | [Data Length] | [Payload] | [Vacancy Length Count] | [Space Boundary Flag] | [Checksum]
```

Edge Cases:
If the system forces a condition where every random slice must be flanked by an anchor point, deadlock can occur in continuous signal segments or at boundaries. Therefore, the system adopts "random slicing where possible, allowing anchor loss." Deadlock risk is bypassed; when anchor conditions fail, a "No Anchor Noise" flag is generated. Semantic ambiguity (e.g., "s()b") is not handled at this layer; it is passed upward for probability collapse.

---

2.2 Edge Inference Subsystem

Structure: Multi-layer LLM pipeline; depth and width defined by the user.
Mechanisms:

· Independent physical cache queues between layers, decoupling serial generation.
· Single-Shot Lifecycle: Cache clears immediately after a single request.
· Final-layer Self-Verification: Internal consistency check after output; serves purely as a confidence marker (no modification, no backtracking).
· Timeout threshold defined by the user; hard interrupt triggered upon timeout, returning a predefined fallback.

Design Rationale: Asynchronous caching resolves the mismatch between physical sensing speed and LLM generation latency. Self-verification is only a confidence marker because backtracking based on already-generated content cannot escape false trajectories. Each request resets to the factory template to avoid residual context contamination.

---

2.3 Global Update Subsystem

Structure: Independent server cluster, physically isolated from inference nodes.
Update Method: Mirror broadcasting (similar to DNS root servers). The update center publishes suggested versions; edge nodes pull them autonomously. Edge nodes run self-test sets locally; switch to the new version if the error rate is below threshold, or auto-rollback if above. Caches are isolated by version stamps in external storage. If the central update center fails, edge nodes with stable versions continue offline via the Single-Shot Lifecycle.

Design Rationale: Full-push updates risk global failure; mirror broadcasting shifts autonomy to the edge. Central point failure is accepted as a reasonable risk.

---

2.4 User Interface Layer

Exposed Parameters: Layer count, width, cache switch, timeout threshold (exposed as hard parameters; no defaults).
Error Output: Self-verification failures, injection errors, or timing interrupts are returned raw, without formatting or sanitization.
Ethical Presets: None; no value judgments are made.

Design Rationale: The sole function is to hand over control. Any cache format injected is executed directly without validation. Errors are returned to the user and written concurrently to the update center's external training corpus for subsequent back-training.

---

3. Probability Collapse and Self-Verification Model

Upon receiving the vacancy length count and the space boundary flag, the upper-layer engine executes Bayesian probability collapse.

Given physical missing length  L , context fragment  C , and candidate completion space  W , the posterior probability is:

P(w | L, C) = \frac{P(L | w) \cdot P(w | C)}{\sum_{w' \in W} P(L | w') \cdot P(w' | C)}

 P(L|w)  is the match score between the candidate completion and the physical length. The self-verification module outputs the normalized score of this posterior. A "Low Confidence" label is appended if the score falls below the user-defined threshold. The Single-Shot Lifecycle maintains statistical independence across rounds.

---

4. Network Topology and Synchronization

The edge nodes and the update center communicate via a Gossip protocol.

· Version Broadcast: The update center broadcasts time-stamped version suggestions at intervals.
· Pull/Push Mechanism: Edge nodes pull new versions from external storage during low-load periods, run self-tests locally, and switch based on error rates.
· Bandwidth Isolation: Data is segregated by version stamps in external storage; concurrent reads do not cause contamination.

---

5. Adversarial Resilience

The system executes adversarial inputs directly. The physical defense relies on the atomicity of the sequence numbers. If the caller attempts to forge out-of-order frames or bypass the vacancy length count, the upper layer will reject them as unmatched fragments during reassembly, returning a "No Anchor Noise" marker without incurring additional computational load or overflow.

---

6. Cross-Language Adaptation

Vacancy length counts and space boundary flags are global frame fields.
For Latin-script languages, the following operations are required:

1. Length Mapping: Map the interruption duration directly to the number of missing letters.
2. Boundary Enforce: Use the space boundary flag to distinguish whether the vacancy crosses word boundaries.
3. Length Validation: The final-layer self-verification checks if the completion length matches the vacancy length count.

---

7. Comparison

Architecture Signal Entry Update Mode Ethical Preset Behavior on Center Failure
RLHF Software Token Central Push Strong Paralysis
MoE Software Token Distributed Shared Weak Partial Loss
RAG Vector Retrieval Static Knowledge Base None Degradation
DMEI Physical Gate Array Mirror Broadcast Zero Independent Survival

DMEI is positioned for industrial IoT, high-reliability edge terminals, and decentralized verification scenarios.

---

8. Known Constraints

· No mature calibration scheme currently exists for multi-valued gate mass-production consistency.
· Asynchronous parallel operation introduces additional power consumption; cooling and distribution depend on deployment specifics.
· The configuration barrier is high; a simplified mode may be introduced in future iterations.

---

9. Open Source Statement

This is a pure document repository; no code is provided.
Discussions is enabled with the following rules:

1. The author does not provide engineering implementation guidance or technical Q&A.
2. Peer-to-peer discussion and theoretical exchange are encouraged.
3. The author reserves the right to ignore posts directly asking for guidance or implementation.

License: Apache License 2.0. Commercial derivatives are permitted with proper attribution. The core design patents may not be used to restrict other users.

---

10. Disclaimer

This document is a conceptual specification and does not constitute an executable engineering blueprint. Multi-valued logic gates, RRAM, and FeFET are currently in the laboratory research phase, with no mature mass-production solutions. This document makes no express or implied guarantees regarding manufacturability or commercial viability.

The copyright of the system architecture and textual content belongs to the original author. Any third party implementing or commercializing based on this specification must cite the original source. The author assumes no liability for direct or indirect losses incurred from development and deployment. All engineering risks and responsibilities remain with the implementing party.

---

Version: v3.3
Last Updated: July 2026

---

This document was compiled with the assistance of AI.
