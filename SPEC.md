# ZKDP — Zero Knowledge Diagnostic Protocol

**Version:** 0.2 (Draft)
**Author:** xADHDx
**License:** AGPL-3.0
**Repository:** https://github.com/xADHDx/ZKDP
**Reference Implementation:** AIWarden — https://github.com/xADHDx/AI-Warden
**Date:** May 27, 2026

---

## Abstract

ZKDP (Zero Knowledge Diagnostic Protocol) is a protocol designed to eliminate data leakage when communicating with an AI through API calls. Instead of sending raw logs, files, or personal infrastructure data, ZKDP mathematically transforms all observable events into anonymized vectors before transmission. The receiving AI reasons over the math — never the data. Raw content never leaves the machine. Ever.

ZKDP is AI-agnostic and designed to work across major models including Claude (Sonnet, Opus), GPT, and any API-accessible LLM. Payload sizes are dramatically smaller than raw log transmission, reducing token consumption and improving diagnostic signal-to-noise ratio, without sacrificing reasoning capability or the privacy principles that privacy-focused operators already depend on.

---

## Problem Statement

Every existing method of AI-assisted infrastructure troubleshooting requires sending real data to an external model — IP addresses, hostnames, file paths, credentials, and raw log content. Privacy-focused operators, self-hosters, and regulated industries have been forced to choose between AI capability and data privacy. ZKDP removes that tradeoff entirely.

Tools like Claude Code, Cursor, and other AI-assisted developer tools run locally but still transmit everything they see — logs, file contents, terminal output, file paths, credentials — to an external API. A local interface is not a privacy guarantee. ZKDP is the privacy layer that sits beneath any AI interface and ensures what reaches the API is never reconstructable back to your infrastructure.

The compute offload model is intentional. The local machine does cheap work — sanitization, tokenization, SFL transformation, registry lookup. The AI does expensive work — pattern recognition, causal reasoning, diagnostic inference. No GPU required locally. Designed to run on a 2-core LXC with 512MB RAM on commodity hardware.

---

## Core Principle — Everything Is Already Math

Every log ever written by any software on any operating system on any device is doing exactly one thing — recording state transitions.

service started = null to running. connection failed = expected to failed. disk at 90% = normal to warning. user logged in = unauthenticated to authenticated. process crashed = running to null.

This is true for every system that has ever existed. Windows, Linux, Docker, Kubernetes, Raspberry Pi, Cisco routers, IoT devices, cloud instances. Every log line is a thing that was in one state and is now in another state.

The software name is a label on top of that transition. The IP address is a label. The username is a label. Strip the labels and what remains is pure state transition data — identical in mathematical structure across every system ever built.

ZKDP does not convert infrastructure into math. It was always math. ZKDP stops pretending the human labels are necessary for reasoning. They are not. An AI does not need to know it is Navidrome to diagnose a transcode dependency cascade. It needs to see the shape of the failure. The shape is universal. The label is irrelevant.

That last step — making everything mathematical such that an AI can fully troubleshoot without ever knowing what software, OS, or system it is working with — is near impossible. But due to math, somehow possible.

---

## Theoretical Foundation

Shannon Information Theory 1948: Information has a measurable quantity independent of its meaning. A failure event carries a specific entropy value regardless of what software produced it. ZKDP transmits exactly that information content and nothing else. Identity labels carry zero diagnostic entropy.

State Transition Theory: Every observable system can be modeled as a finite set of states and transitions between them. Failure behavior has mathematical shape that exists independently of identity. A cascade is a cascade. A drain is a drain. These shapes are recognizable to a reasoning system without any identity context.

Zero Knowledge Proof Principles: ZKDP applies zero knowledge principles to open-ended diagnostic reasoning. The machine proves to the AI that a specific failure pattern exists. The AI reasons about that pattern and returns a structured response. At no point does the AI hold information sufficient to reconstruct the original data. Note: ZKDP is not a formal cryptographic ZKP system. It is an identity-minimized diagnostic abstraction system that applies zero knowledge principles at the protocol level.

BLAKE3 Keyed Verification: Every valid ZKDP session token is verified using a BLAKE3 keyed hash with the session prime as the key. This provides computationally infeasible forgery resistance. The session prime rotates every session and never leaves the local machine.

Universal Error Taxonomy: Every error in every system maps to one or a combination of seven fundamental categories: resource exhaustion, dependency failure, state violation, timeout, permission violation, data corruption, and concurrency violation. These categories are universal because they describe fundamental constraints of computation itself, not specific software.

---

## ZKDP Vocabulary and Terminology

This section defines every term used in the ZKDP protocol. All implementations must use these terms consistently.

### Packet Types

SURPRISE packet — the primary outbound transmission unit. Carries sanitized SFL event data from the local machine to the AI. Never contains real values. Always contains tokens.

CONTEXT packet — the Channel B transmission unit. Carries anonymous ontology IDs describing service class, known issue class, and fix class. Never contains human-readable labels. Never contains real values.

RESPONSE packet — the AI reply. Must be in ZKDP protocol language only. No prose. No natural language. Any response that does not parse as valid ZKDP is rejected and never reaches the repair engine.

### Failure Classifications

SPIKE — sudden single-sequence divergence. One observable deviates from baseline in a single sequence step. Characteristic of crashes, unexpected restarts, discrete failures.

DRIFT — slow accumulated divergence crossing a cumulative threshold. No single step is anomalous but the cumulative deviation is. Characteristic of memory leaks, gradual degradation, slow resource exhaustion.

CASCADE — multiple observables diverging within a σ0 window forming a causal chain. One failure triggers others in near-zero time. Characteristic of dependency failures propagating downstream.

DRAIN — monotonic depletion of a finite resource heading toward a hard floor or ceiling. Carries a PROJ field with depletion projection. Characteristic of disk fill, memory climb, connection pool exhaustion. The only packet type that fires before anything has actually failed.

ANOMALY — unrecognized failure shape that does not fit any known classification. Triggers immediate snapshot, best-effort AI reasoning, and mandatory human escalation.

### SFL Primitives

S(n) — sequence position n. Replaces timestamps. Ordinal only. No wall-clock time ever exists in a ZKDP payload.

Delta(n1,n2) — relative distance between two sequence positions. Unitless. Used to compute scale class.

sigma — scale class of a temporal distance. Five classes only:
sigma-0 = instant under 1 second
sigma-1 = rapid in seconds
sigma-2 = short in minutes to hours
sigma-3 = medium in hours to days
sigma-4 = long in days to weeks or more

R(n) — recurrence count at sequence position n. How many times this event pattern has occurred.

### Packet Fields

BASE — baseline version identifier. Both machine and AI reference the same baseline. A packet without a matching baseline is rejected.

SEQ — monotonic integer counter. Replaces all timestamps. Provides ordering without exposing wall-clock time.

TYPE — failure classification. One of SPIKE, DRIFT, CASCADE, DRAIN, ANOMALY.

DIV — divergence field. Contains predicted state P versus actual state A for each observable. The core diagnostic payload.

SCALE — array of sigma classes corresponding to each divergence event. Provides temporal relationship without real time values.

MAG — surprise magnitude. Float from 0 to 1. Gates transmission — below configured threshold no packet fires.

PROJ — projection field. DRAIN packets only. Contains floor, rate, and ttf expressed in sigma class never real time values.

### System Components

Token vault — local mapping between real infrastructure values and session-scoped anonymous tokens. Lives exclusively on local machine. Never transmitted. AES-256 encrypted at rest. Single-writer mutex locked.

Session prime — cryptographically random prime number generated fresh at each session start. Used as BLAKE3 verification key. Never transmitted. Rotates every session.

Checksum proof — the Layer 4 mathematical verification that every value leaving the machine is a registered session token and not a real value. A valid token satisfies a session-prime-scoped verification property that real infrastructure values do not. In the reference implementation this property is a BLAKE3 keyed hash, keyed by the session prime, computed at tokenization and re-verified at egress: BLAKE3(token, session_prime) must equal the stored hash for that token or transmission aborts. Requires no ML, no external calls, runs in microseconds on any CPU. It does not search for known-bad patterns — it proves the presence of known-good ones. The absence of proof is proof of failure.

Baseline — agreed model of normal shared between machine and AI. Established once at session start. Referenced by version number. Never re-sent. Updated after every successful repair.

Surprise transmission model — ZKDP transmits only when reality diverges from baseline. Silence is a valid protocol state. A healthy infrastructure generates zero bytes.

Channel A — sanitized event data channel. Carries SURPRISE packets with zero real values, all tokens.

Channel B — anonymous context channel. Carries CONTEXT packets with ontology IDs only. Never merges with Channel A.

Service Profile Registry — locally maintained knowledge base mapping ontology IDs to real service information. Never transmitted. Resolves AI generic hypotheses into service-specific repair actions locally.

Ontology IDs — numeric identifiers replacing human-readable semantic labels in Channel B. SC (service class), KI (known issue), FC (fix class). Mappings live only in local registry.

Canary values — synthetic PII-shaped values injected into test payloads before any live data is processed. Every canary must be caught and tokenized. Any canary passing through unsanitized halts the system.

Whitelist — per-service list of permitted repair actions. AI RESPONSE actions validated against whitelist before execution. Actions not on whitelist cannot execute regardless of AI recommendation. Destructive commands are never on any whitelist.

### Transmission Decisions

SAFE — token carries no identity risk. Passes through without tokenization.

SANITIZE — value requires transformation. Passes to tokenizer.

TOKENIZE — value is confirmed PII. Replaced with session token.

ESCALATE — value is ambiguous. Passes to LLM auditor in Layer 2.

---

## ZKDP Protocol Grammar

Formal grammar for all packet types. All implementations must conform.

SURPRISE PACKET:
ZKDP/1.0 SURPRISE
BASE: v[INTEGER]
SEQ: [INTEGER]
TYPE: SPIKE|DRIFT|CASCADE|DRAIN|ANOMALY
MAG: [0.0-1.0]
DIV: {
  S([n]): E([TOKEN], [EVENT_TYPE]) = { P=[STATE], A=[STATE] (, KEY=VALUE)* }
}
SCALE: [sigma-n, ...]
DRAIN: { observable: [TOKEN], floor: [FLOAT], rate: [+/-FLOAT]/sigma, ttf: sigma-n }

EVENT_TYPE values: BIND, AUTH, STREAM, TRANSCODE, SCAN, MEM, THROTTLE, DB, IMPORT, EXPORT, START, STOP, RESTART, CONNECT, DISCONNECT

STATE values: +1, -1, null, WARN, [FLOAT]

CONTEXT PACKET:
ZKDP/1.0 CONTEXT
BASE: v[INTEGER]
SC: [INTEGER]
KI: [INTEGER]
FC: [INTEGER]

RESPONSE PACKET:
ZKDP/1.0 RESPONSE
BASE: v[INTEGER]
SEQ: [INTEGER]
CONFIDENCE: [0.0-1.0]
ACTION_VECTOR: [
  { ACTION: [ACTION_TYPE], TARGET: [TOKEN], VERIFY: { [FIELD]: [STATE] }, ... }
]
VERIFY_CONDITION: { [FIELD]: [STATE], ... }
FAIL_CONDITION: { [FIELD]: [STATE], ... }
FALLBACK: { ACTION: SNAPSHOT_ROLLBACK, TRIGGER: any_fail_condition }

ACTION_TYPE values: RESTART, REINSTALL, RECLAIM, ISOLATE, RESET, REDUCE, VERIFY, SNAPSHOT_ROLLBACK

---

## Layer 2 — Three-Signal Context Sanitizer

Layer 2 uses three deterministic signals combined into a risk score. The LLM fires only on ESCALATE decisions — rare exception handler, not primary scanner.

Signal 1 — Vocabulary Classifier
Checks each token against known-safe system vocabulary. Output: VOCAB_CLASS in {SYSTEM, UNKNOWN, MIXED}. Known safe includes Linux system terms, log level words, HTTP method words, common service verbs, hardware interface names, standard error codes. This signal is noise reduction only — not a safety decision alone.

Signal 2 — Role Classifier
Determines structural role of each token. Output: ROLE in {KEY, VALUE, METRIC, EVENT, CONTROL}.
KEY — appears before equals sign or colon
VALUE — appears after equals sign or colon
METRIC — numeric value with unit context
EVENT — action or state descriptor
CONTROL — protocol or format token
Role is the strongest signal. Identity risk depends heavily on structural role.

Signal 3 — Source Context Classifier
Classifies log origin. Output: SOURCE_CLASS in {AUTH, SYSTEM, APP, NETWORK, UNKNOWN}.
AUTH — ssh, pam, sudo, login, authentication keywords
SYSTEM — kernel, systemd, cgroup, memory, cpu keywords
APP — application-specific patterns
NETWORK — nginx, proxy, firewall, connection keywords
UNKNOWN — unclassifiable

Risk Scoring:
RISK_SCORE = f(VOCAB_CLASS, ROLE, SOURCE_CLASS, entropy(token), uniqueness(token))

Weights:
ROLE=VALUE: +0.4
VOCAB_CLASS=UNKNOWN: +0.3
SOURCE_CLASS=AUTH: +0.2
entropy above 3.5 bits/char: +0.2
token appears only once in log: +0.1

Thresholds:
RISK_SCORE below 0.3 — SAFE
0.3 to 0.6 — SANITIZE
0.6 to 0.8 — TOKENIZE
0.8 and above — ESCALATE

LLM Auditor: fires only on ESCALATE. Sends flagged token plus surrounding context to local Ollama. Binary question only — is this PII. If yes, tokenize. If LLM fails or times out, tokenize anyway. Fail closed.

---

## 4-Layer Sanitization Pipeline

Layer 1 — Regex Tokenizer: deterministic pattern matching for all known PII shapes. Log normalizer decodes URL encoding, base64, and hex before regex runs. Date and time values protected and passed to SFL transformer.

Layer 2 — Three-Signal Context Sanitizer: vocabulary, role, and source context signals combine into risk score. SAFE, SANITIZE, TOKENIZE, or ESCALATE per token. LLM fires only on ESCALATE. Entirely on-device.

Layer 3 — Egress Leak Check: fail-closed final verification. Pattern-matches complete outbound payload against full known PII universe. Any match aborts. Unknown patterns treated as suspicious. Shares no code with Layers 1 or 2.

Layer 4 — BLAKE3 Proof Verification: for every value in outbound packet, BLAKE3(token, session_prime) must match stored hash. Any failure aborts. Pure arithmetic. No ML. No external calls. Microseconds on any CPU.

---

## Universal Compatibility

Infrastructure: Proxmox, LXC, VMs, bare metal Linux, Docker, Podman, Kubernetes, Windows Server, Raspberry Pi, ARM, IoT, AWS, GCP, Azure.

Log formats: journald, syslog, rsyslog, Docker logs, Windows Event Log, any application log format.

AI targets: Claude, GPT-4, Gemini, Ollama, LM Studio, any OpenAI-compatible endpoint.

---

## Hardware Constraints

Minimum: 2 CPU cores, 2GB RAM, 16GB disk.
Comfortable: 4 CPU cores, 4GB RAM, 32GB disk.
No GPU required. No enterprise hardware required.

---

## Protocol Flow

Step 1 — Local Failure Detection: watchdog detects failure. Local repair scripts attempt resolution. ZKDP initiates only if local resolution fails.

Step 2 — Mandatory Snapshot: snapshot taken before any data gathered or any action executed. No snapshot confirmation equals no execution. Ever. Per-service only. Never host-level.

Step 3 — Data Gathering: all logs, failure codes, service states, resource metrics, dependency states collected. Nothing filtered at this stage.

Step 4 — 4-Layer Sanitization: all data passes through all four layers. No transmission until all four pass.

Step 5 — SFL Transformation: state transitions replace raw log lines. sigma classes replace timestamps. Tokens replace all real values.

Step 6 — Packet Construction: single-pass default. Round trips available for complex failures. Packet type determined by failure shape.

Step 7 — Two-Channel Transmission: Channel A (SURPRISE) and Channel B (CONTEXT) transmitted separately. Never merged. Channel B uses ontology IDs only.

Step 8 — AI Response Constraint: AI responds in RESPONSE packet format only. No prose. Non-compliant responses rejected before reaching repair engine.

Step 9 — Confidence and Whitelist Gating: CONFIDENCE must meet threshold. ACTION must exist on per-service whitelist. Both gates must pass.

Step 10 — Execute, Verify, Rollback: repair executes. Verification packet sent. Success updates baseline. Failure triggers automatic rollback, snapshot restore, human alert.

---

## Shared Baseline

Established once at session start. Referenced by version number in every packet. Neither side re-sends after establishment. Successful repair updates baseline.

---

## Surprise Transmission Model

Transmits only on divergence from baseline. Silence means healthy. A healthy infrastructure generates zero bytes. Cumulative drift tracked to prevent slow failures from being silently missed.

---

## Token Vault

Lives exclusively on local machine. Never transmitted. AES-256 encrypted at rest. Values zeroed from memory immediately after use. Single-writer mutex locked. Session prime is the most sensitive value in the system.

---

## Session Token Rotation

Every session generates fresh token map and fresh session prime. No token or prime reused across sessions. Defeats statistical correlation attacks.

---

## Canary Verification System

Synthetic canary values injected into test payload before live data processed. Every canary must be caught and pass BLAKE3 verification. Any canary passing through undetected halts the system. Canary code shares nothing with sanitizer code.

---

## Service Profile Registry

Locally maintained. Never transmitted. Never published. Maps ontology IDs to real service information and repair actions. AI provides generic structural reasoning. Registry provides identity-specific resolution. They never meet in a payload.

---

## Unknown Failure Behavior

Unknown bugs still produce state transitions. Still generate divergence. Still have a shape. ANOMALY classification: snapshot taken immediately, packet sent to AI with ANOMALY flag, human alert fired.

---

## Known Attack Surfaces

A 300 billion parameter AI model reviewed this protocol and identified the primary attack surface as information reconstructed across time, tools, and composition.

Temporal reconstruction: mitigated by session token rotation, session prime rotation, payload variance injection.

Cross-tool reconstruction: ZKDP privacy guarantee applies only to traffic passing through the ZKDP pipeline.

Compositional reconstruction: mitigated by payload variance injection, session rotation, strict Channel A and B separation.

Channel B inference: numeric ontology IDs replace all human-readable labels. Mapping lives in local registry only.

Structural fingerprinting: unique failure topology can fingerprint infrastructure over time. Partially mitigated by payload variance injection. Acknowledged as an unsolved problem at the intersection of diagnostic utility and privacy.

---

## Failure Modes and Mitigations

Race condition on startup: sanitizer reports READY before watchdog collects. Hard gate.

Sanitizer crash mid-process: full abort. Partial sanitization equals no sanitization.

New PII pattern not in config: filesystem scanner continuously updates vault. Egress check flags unknown as suspicious.

Log encoding edge cases: normalizer decodes all encodings before sanitization.

Token vault desync: atomic writes, single-writer, mutex locked.

Canary bypass: separate codebase from sanitizer. No shared code.

AI response leakage: eliminated by response constraint. No field in response schema can contain real values.

Session correlation: eliminated by per-session rotation.

Memory tool leakage: memory entries audited on session start. PII-matching entries flagged for manual review.

BLAKE3 collision: computationally infeasible with session prime as key.

---

## Reference Implementation

AIWarden — self-hosted homelab infrastructure watchdog on Proxmox.
Repository: https://github.com/xADHDx/AI-Warden

Components: watchdog daemon, 4-layer sanitization pipeline, SFL transformation engine, two-channel packet builder, confidence and whitelist gating, snapshot integration, canary verification, Service Profile Registry, token vault with session prime and BLAKE3 verification.

---

## License

AGPL-3.0. Any implementation of this protocol used in a networked product or service must open source that implementation under the same license. No exceptions.

---

ZKDP was designed and authored by xADHDx in collaboration with Claude (Anthropic) on May 26-27, 2026. This document constitutes the original specification and establishes prior art for the Zero Knowledge Diagnostic Protocol.