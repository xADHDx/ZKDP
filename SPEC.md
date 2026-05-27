# ZKDP — Zero Knowledge Diagnostic Protocol

**Version:** 0.1 (Draft)
**Author:** xADHDx
**License:** AGPL-3.0
**Repository:** https://github.com/xADHDx/ZKDP
**Reference Implementation:** AIWarden — https://github.com/xADHDx/AI-Warden
**Date:** May 26, 2026

## Abstract

ZKDP (Zero Knowledge Diagnostic Protocol) is a protocol designed to eliminate data leakage when communicating with an AI through API calls. Instead of sending raw logs, files, or personal infrastructure data, ZKDP mathematically transforms all observable events into anonymized vectors before transmission. The receiving AI reasons over the math — never the data. Raw content never leaves the machine. Ever.

ZKDP is AI-agnostic and designed to work across major models including Claude (Sonnet, Opus), GPT, and any API-accessible LLM. Payload sizes are dramatically smaller than raw log transmission, reducing token consumption and improving diagnostic signal-to-noise ratio, without sacrificing reasoning capability or the privacy principles that privacy-focused operators already depend on.

## Problem Statement

Every existing method of AI-assisted infrastructure troubleshooting requires sending real data to an external model — IP addresses, hostnames, file paths, credentials, and raw log content. Privacy-focused operators, self-hosters, and regulated industries have been forced to choose between AI capability and data privacy. ZKDP removes that tradeoff entirely.

Life does not always allow you to be at your desk when something breaks. Automation is not optional — it is essential. The problem is that every existing AI-assisted automation pipeline leaks your infrastructure identity to an external model. ZKDP is the solution. It allows full AI diagnostic capability without a single byte of real infrastructure data ever leaving your network.

Tools like Claude Code, Cursor, and other AI-assisted developer tools run locally but still transmit everything they see — logs, file contents, terminal output, file paths, credentials — to an external API. A local interface is not a privacy guarantee. ZKDP is the privacy layer that sits beneath any AI interface and ensures what reaches the API is never reconstructable back to your infrastructure.

## Core Principle — Everything Is Already Math

Every log ever written by any software on any operating system on any device is doing exactly one thing — recording state transitions. Nothing more.

service started = null to running. connection failed = expected to failed. disk at 90% = normal to warning. user logged in = unauthenticated to authenticated. process crashed = running to null.

This is true for every system that has ever existed. Windows, Linux, Docker, Kubernetes, Raspberry Pi, Cisco routers, IoT devices, cloud instances. Every log line ever written is a thing that was in one state and is now in another state at some point in time.

The software name is a label on top of that transition. The IP address is a label. The username is a label. Strip the labels and what remains is pure state transition data — identical in mathematical structure across every system ever built.

ZKDP does not convert infrastructure into math. It was always math. ZKDP simply stops pretending the human labels on top of that math are necessary for reasoning. They are not. An AI does not need to know it is Navidrome to diagnose a transcode dependency cascade. It needs to see the shape of the failure. The shape is universal. The label is irrelevant.

That last step — making everything mathematical such that an AI can fully troubleshoot without ever knowing what software, OS, or system it is working with — is near impossible. But due to math, somehow possible.

## Theoretical Foundation

ZKDP is grounded in three established pillars of computer science and one novel verification mechanism.

Shannon Information Theory 1948: Information has a measurable quantity independent of its meaning. A failure event carries a specific entropy value regardless of what software produced it or what IP address was involved. ZKDP transmits exactly that information content and nothing else. Identity labels carry zero diagnostic entropy. They are overhead.

State Transition Theory: Every observable system can be modeled as a finite set of states and transitions between them. Failure behavior has mathematical shape that exists independently of identity. A cascade is a cascade. A drain is a drain. These shapes are recognizable to a reasoning system without any identity context.

Zero Knowledge Proofs: A zero knowledge proof allows one party to prove a statement is true to another party without revealing any information about why it is true beyond the truth of the statement itself. ZKDP applies this principle to open-ended diagnostic reasoning. The machine proves to the AI that a specific failure pattern exists. The AI reasons about that pattern and returns a structured response. At no point does the AI hold information sufficient to reconstruct the original data.

Checksum Proof Verification: Every valid ZKDP session token is a number that satisfies a session-specific mathematical property — its digit sum modulo a session-scoped prime equals zero. This property is self-verifying, requires no vault lookup, and is computationally trivial. A real infrastructure value slipping through sanitization will almost certainly fail this proof. The session prime rotates every session and never leaves the local machine. This mechanism was derived from the same mathematical principle used in credit card validation, extended with session-scoped prime rotation for cryptographic strength.

## Universal Compatibility

ZKDP is universally compatible by nature, not by engineering effort. Because every log is a state transition, and state transitions are mathematically identical across all systems, ZKDP works on any infrastructure that produces logs.

Infrastructure: Proxmox, LXC, VMs, bare metal Linux, Docker, Podman, Kubernetes, Windows Server, Raspberry Pi, ARM devices, IoT devices, cloud instances on AWS GCP and Azure.

Log formats: Systemd journald, syslog, rsyslog, Docker logs, Windows Event Log, and any application-specific log format from any software ever written.

AI targets: Claude Sonnet and Opus, GPT-4 and GPT-4o, Gemini, local models via Ollama or LM Studio, and any OpenAI-compatible API endpoint.

The SFL transformation layer is the universal adapter. It ingests any log format and outputs the same mathematical packet structure regardless of source. The AI receives identical ZKDP packets whether the source was a Proxmox LXC, a Windows Server, or a Kubernetes pod. This is the same principle TCP/IP uses — the protocol abstracts the physical medium. ZKDP abstracts infrastructure identity.

## Hardware Constraints

ZKDP is explicitly designed to run within the resource constraints of a standard LXC container on commodity hardware. No component of this protocol requires a GPU, high core count, or large memory allocation. The entire pipeline is designed to run on CPU with a memory footprint under 512MB. This is a first-class design requirement, not an afterthought. A privacy protocol that only runs on expensive hardware is not a protocol for the world — it is a protocol for the privileged.

Recommended minimum LXC allocation: 2 CPU cores, 512MB RAM, 4GB disk.
Recommended comfortable allocation: 2 CPU cores, 1GB RAM, 8GB disk.

## Token Design

ZKDP tokens are pure integers, not human-readable strings. A token does not contain class information, count information, or any human-readable structure. 192.168.1.57 becomes 9242. xADHDx becomes 3451. /mnt/media2 becomes 7819.

Every valid session token satisfies the following mathematical property: checksum(token) mod session_prime = 0, where checksum is the sum of the token's digits and session_prime is a prime number generated fresh at session start and stored only in the local token vault.

Tokens are self-verifying. At any point in the pipeline, any value can be checked for token validity without a vault lookup. A real value that slipped through sanitization will fail this check with near certainty. The probability of a real infrastructure value accidentally satisfying a session-scoped prime checksum is astronomically small and decreases as prime size increases.

Session primes rotate every session. No prime is ever reused. An attacker holding packets from multiple sessions cannot reconstruct the verification property across sessions.

## 4-Layer Sanitization Pipeline

All collected data passes through four independent verification layers before any transmission occurs. Every layer must pass. Any layer failure aborts transmission entirely. Partial sanitization is treated identically to no sanitization. If uncertain, abort. Always.

Layer 1 — Regex Tokenizer: Every IP address, MAC address, hostname, domain, file path, port, API key, and credential pattern is replaced with a valid session token satisfying the checksum property. The log normalizer runs before this layer, decoding all encodings including unicode, base64, hex, and URL encoding so no encoded value bypasses regex detection.

Layer 2 — Local LLM: A quantized 1B to 3B parameter model running entirely on-device reviews Layer 1 output and catches anything regex missed — contextual PII, semantic identifiers, unusual formats, and values that pattern matching cannot detect. This model never connects to any external service. It runs entirely in local memory within the LXC resource budget.

Layer 3 — Egress Leak Check: A fail-closed verification pass that pattern-matches the full outbound payload against the known PII universe one final time. Any match aborts transmission. Unknown patterns are treated as suspicious not safe. This layer operates independently of Layers 1 and 2 and shares no code with either.

Layer 4 — Checksum Proof Verification: A formal mathematical proof pass. For every value in the outbound packet the following must hold. Given that valid tokens satisfy checksum(V) mod session_prime = 0, and given that session_prime is known only to the local vault, and given that real infrastructure values do not satisfy this property — for all values V in packet P, if checksum(V) mod session_prime = 0 then V is a proven valid token and the packet is mathematically clean. If any V fails then a real value has been detected with near certainty, transmission is aborted, the failure is logged, and the operator is alerted. This layer requires no ML, no heavy compute, and no external calls. It is pure arithmetic running in microseconds on any CPU. It does not search for known bad patterns — it proves the presence of known good ones. The absence of proof is proof of failure. All four layers failing simultaneously on the same value is not a practical attack surface. It is a philosophical one.

## Sequence Formula Language

Sanitized data is transformed into Sequence Formula Language — a mathematical notation that preserves causal and temporal relationships between events without timestamps or real values.

SFL primitives: S(n) is sequence position n. Delta(n1,n2) is relative distance between two events. Sigma(n) is the scale class of that distance. R(n) is recurrence count at position n.

Scale classes: sigma-0 is instant under 1 second. sigma-1 is rapid in seconds. sigma-2 is short in minutes to hours. sigma-3 is medium in hours to days. sigma-4 is long in days to weeks or more.

No real timestamps ever exist in a ZKDP payload. Sequence ordering and scale classes provide all temporal reasoning capability the AI needs without exposing wall-clock time or enabling timestamp correlation attacks.

## Protocol Flow

Step 1 — Local Failure Detection: The local watchdog detects a failure. Local LLM and repair scripts attempt to resolve it automatically. If local resolution fails ZKDP initiates. ZKDP is the last resort before human escalation, not the first response.

Step 2 — Mandatory Snapshot: Before any data is gathered or any action is ever taken, a snapshot of the affected service or instance is taken. This is non-negotiable. No snapshot confirmation equals no execution. Ever. The snapshot provider is environment-aware. Proxmox uses native LXC snapshot via Proxmox API. Docker uses container state snapshot. Bare metal uses targeted config backup via timeshift or equivalent. Snapshots are per-service and per-instance. Never host-level for a container-level repair. Each instance is an isolated blast radius.

Step 3 — Data Gathering: AIWarden gathers all necessary troubleshooting data — logs, failure codes, service states, resource metrics, running services, dependency states. Everything needed to fully understand the failure. Nothing is filtered at this stage. Everything gets collected first.

Step 4 — 4-Layer Sanitization: All collected data passes through the 4-layer sanitization pipeline as defined above. No transmission occurs until all four layers pass.

Step 5 — SFL Transformation: Sanitized data is transformed into SFL vectors. State transitions replace raw log lines. Scale classes replace timestamps. Tokens replace all real values.

Step 6 — Packet Construction: Sanitized mathematically transformed data is assembled into a ZKDP surprise packet. Transmission defaults to single-pass — one complete payload, one AI response, one repair attempt. Round trips are available for complex multi-system failures where the AI requests additional targeted data. Packet types are SPIKE for sudden single-sequence divergence, DRIFT for slow accumulated divergence crossing a cumulative threshold, CASCADE for multiple observables diverging within a sigma-0 window forming a causal chain, DRAIN for monotonic depletion of a finite resource heading toward a hard floor or ceiling, and ANOMALY for unrecognized failure shapes requiring immediate human escalation. Packet format fields are BASE for the baseline version both sides reference, SEQ for a monotonic counter replacing all timestamps, TYPE for failure pattern classification, DIV for predicted versus actual state per observable, SCALE for SFL temporal distance classes, MAG for surprise magnitude from 0 to 1 which gates transmission, and PROJ for depletion projection on DRAIN type only expressed in sigma classes never real time values.

Step 7 — Two Channel Transmission: ZKDP transmits across two strictly separated channels that never merge into a single payload. Channel A carries sanitized event data — the SFL packet with zero real values, all tokens, pure mathematical representation of the failure. Channel B carries anonymous context — non-identifying software metadata only, pre-resolved from the local Service Profile Registry. No real values. No version numbers that could fingerprint. Only semantic class information such as SERVICE_CLASS and KNOWN_ISSUE class and severity. Channel B primes the AI's domain reasoning without revealing identity.

Step 8 — AI Response Constraint: The AI must respond exclusively in ZKDP protocol language. No natural language. No prose. No free text fields. Any response that does not parse as valid ZKDP is rejected entirely and never reaches the repair engine. This is the single most important privacy rule in the protocol. A constrained response format makes inference leakage mathematically impossible. There is no field in the response schema capable of containing a real value. Response format fields are BASE for baseline version, SEQ for sequence number, CONFIDENCE for a score from 0 to 1, ACTION for a whitelisted action against a token, VERIFY for the expected post-action state, and FALLBACK for the snapshot restore instruction if verification fails.

Step 9 — Confidence and Whitelist Gating: Before any repair action executes two gates must both pass. The confidence gate requires the AI confidence score to meet or exceed the configured threshold — below threshold nothing executes. The whitelist gate requires the proposed action to exist on the per-service whitelist maintained locally — if the action is not whitelisted the repair engine cannot execute it regardless of what the AI recommended. Destructive commands do not exist on any whitelist. Ever.

Step 10 — Execute, Verify, and Rollback: The repair action executes against the snapshot-protected service. AIWarden collects post-action state and sends a verification packet. If the AI confirms resolution the repair is committed and the shared baseline updates to reflect the new normal. If the AI determines the repair made things worse automatic rollback triggers — the snapshot restores, state returns to pre-repair condition, and a human alert fires with full context of what was attempted, what the outcome was, and what the current state is.

## Shared Baseline

Both the machine and the AI hold an agreed model of normal. The baseline is established once at session start and referenced by version number in every subsequent packet. It contains the service set, their expected states, the SFL schema version, and the session token map. Neither side re-sends it after establishment. A packet is always a delta against the current baseline. No baseline confirmation equals no packet accepted. A successful repair updates the baseline to reflect the new normal state.

## Surprise Transmission Model

ZKDP only transmits when reality diverges from the baseline prediction. Silence is a valid and expected protocol state meaning everything matches what the AI already believes. A healthy infrastructure generates zero bytes. This is not compression — it is the deliberate decision to transmit only surprise. The unit of transmission is divergence. Low divergence below the MAG threshold does not generate a packet. Cumulative drift that crosses the cumulative threshold generates a DRIFT packet even if no single step crossed the individual threshold, ensuring slow-creeping failures are never silently missed.

## Token Vault

The token vault is the mapping between real infrastructure values and their session-scoped anonymous tokens. It lives exclusively on the local machine. It never leaves the network. It is encrypted at rest using AES-256. Values are decrypted into memory only during active sanitization operations and zeroed from memory immediately after use. Token mappings rotate every session — no token from a previous session is ever reused — defeating long-term correlation attacks. The vault is single-writer with mutex locking to prevent race conditions. The session prime lives in the vault and is the most sensitive value in the entire system.

## Session Token Rotation

Every ZKDP session generates a completely fresh token map and a completely fresh session prime. A token from session 1 is not valid in session 2. An attacker holding packets from multiple sessions cannot correlate tokens across sessions because both the mapping and the verification property change every time. This defeats statistical correlation attacks against long-running infrastructure monitoring.

## Canary Verification System

Before any live data is ever processed AIWarden plants synthetic canary values in a test payload — values that look exactly like real IPs, API keys, hostnames, and credentials but are not. The sanitizer runs against the canary payload. Every single canary must be caught, tokenized, and pass the checksum proof. If even one canary passes through unsanitized the system fails and refuses to proceed until the issue is resolved. The canary verification process runs on completely separate code from the sanitizer. The thing checking the sanitizer shares no code with the sanitizer. This is the mathematical proof that the system works before it ever touches real data.

## Service Profile Registry

The Service Profile Registry is a locally-maintained auto-updating knowledge base that resolves the AI's generic structural hypotheses into service-specific actionable guidance without ever revealing service identity to the AI. When the AI concludes that a dependency in the codec class is missing the registry resolves that locally to the specific package name and the correct install command. The AI provided the reasoning. The registry provided the identity. They never meet inside a payload. The registry updates itself by pulling from public bug trackers, CVE feeds, and release notes on a schedule — an outbound read of public data that reveals nothing about what software you run.

## Unknown Failure Behavior

An unknown bug still produces math. It still causes state transitions. It still generates divergence from baseline. It still has a shape. The AI still reasons over the causal structure of what happened even if it has never encountered that exact failure pattern. If the packet does not fit any known type it is classified as ANOMALY. On ANOMALY a snapshot is taken immediately, the packet is sent to the AI with the ANOMALY flag for best-effort reasoning, and a human alert fires. The operator is never left with a silent failure. Worst case outcome is AIWarden reporting it does not recognize this pattern and escalating to human review with full mathematical context preserved.

## Known Attack Surfaces

A 300 billion parameter AI model reviewed this protocol and identified the following as the primary attack surface: information gets reconstructed across time, tools, and composition. This is the most important external feedback this protocol has received and it is documented here as a first-class concern.

Temporal reconstruction: Statistical patterns in mathematical payloads across many sessions could fingerprint a specific infrastructure stack even without real values. Mitigated by session token rotation, session prime rotation, and payload variance injection — controlled noise introduced into non-critical fields to defeat statistical fingerprinting without affecting AI reasoning.

Cross-tool reconstruction: ZKDP sanitizes what reaches the AI API. Other tools in the stack may also communicate with AI systems without ZKDP. Privacy guarantees break at those seams. Mitigated by defining ZKDP scope boundaries explicitly and documenting that the privacy guarantee applies only to traffic that passes through the ZKDP pipeline.

Compositional reconstruction: Two packets individually reveal nothing. Multiple packets composed across sessions may reveal more than any single packet. This is how anonymized datasets get de-anonymized in the real world. Mitigated by payload variance injection, session token and prime rotation, and strict Channel A and Channel B separation.

## Failure Modes and Mitigations

Race condition on startup: sanitizer must report READY before the watchdog collects anything. Hard gate, no exceptions.

Sanitizer crash mid-process: any layer failure results in full abort. Partial sanitization equals no sanitization.

New PPI pattern not in config: filesystem scanner continuously updates the token vault. Egress check flags unknown patterns as suspicious not safe.

Log encoding edge cases: log normalizer decodes all encodings before sanitization runs.

Token vault desync: vault writes are atomic, single-writer, mutex locked.

Canary test bypass: canary verification runs on completely separate code from the sanitizer.

AI response inference leakage: eliminated by the AI Response Constraint. No field in the response schema can contain a real value.

Session token correlation: eliminated by per-session token and prime rotation.

Memory tool leakage: memory entries audited on every session start. Any entry containing pattern-matched PII flagged for manual review before the session proceeds.

Checksum collision: the probability of a real value satisfying the session prime checksum is inversely proportional to the size of the prime. Minimum prime size is enforced by the vault generator to keep collision probability below one in ten billion per session.

## Reference Implementation

AIWarden is the reference implementation of ZKDP, built for self-hosted homelab infrastructure on Proxmox. It implements every component of this specification including the watchdog daemon, 4-layer sanitization pipeline, SFL transformation engine, two-channel packet builder, confidence and whitelist gating, snapshot integration, canary verification system, Service Profile Registry, and token vault with session prime rotation.

Repository: https://github.com/xADHDx/AI-Warden

## License

ZKDP is published under the GNU Affero General Public License v3.0. Any implementation of this protocol used in a networked product or service must open source that implementation under the same license. No exceptions.

---

ZKDP was designed and authored by xADHDx in collaboration with Claude (Anthropic) on May 26, 2026. This document constitutes the original specification and establishes prior art for the Zero Knowledge Diagnostic Protocol.
