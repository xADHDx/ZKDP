# ZKDP — Zero Knowledge Diagnostic Protocol

**Version:** 0.1 (Draft)  **Author:** xADHDx  **License:** AGPL-3.0  **Repository:** https://github.com/xADHDx/ZKDP  **Reference Implementation:** AIWarden — https://github.com/xADHDx/AI-Warden

---

## Abstract

ZKDP (Zero Knowledge Diagnostic Protocol) is a protocol designed to eliminate data leakage when communicating with an AI through API calls. Instead of sending raw logs, files, or personal infrastructure data, ZKDP mathematically transforms all observable events into anonymized vectors before transmission. The receiving AI reasons over the math — never the data. Raw content never leaves the machine. Ever. 

ZKDP is AI-agnostic and designed to work across major models 
including Claude (Sonnet, Opus), GPT, and any API-accessible 
LLM. Payload sizes are dramatically smaller than raw log 
transmission, reducing token consumption and improving 
diagnostic signal-to-noise ratio, without sacrificing 
reasoning capability or the privacy principles that 
privacy focused operators already depend on.

---

## Problem Statement

Every existing method of AI-assisted infrastructure troubleshooting requires sending real data to an external model — IP addresses, hostnames, file paths, credentials, and raw log content. Privacy-focused operators, self-hosters, and regulated industries have been forced to choose between AI capability and data privacy. ZKDP removes that tradeoff entirely. Life doesn't always allow you to be at your desk when something breaks. Automation is not optional — it is essential. The problem is that every existing AI-assisted automation pipeline leaks your infrastructure identity to an external model. ZKDP is the solution to that problem. It allows full AI diagnostic capability without a single byte of real infrastructure data ever leaving your network.

---

## Core Principle

Everything a computer does is already math. A log file is bytes. An IP address is a 32-bit integer. A failure event is a state transition. ZKDP strips the human abstraction layer and sends the AI the math directly. The AI reasons over the shape of the failure — not the identity of the system that produced it. Failure shapes are universal. A cascade looks like a cascade whether it originated in a homelab in Ohio or a data center in Tokyo. The math is the same. The identity is irrelevant. That last step — making everything mathematical such that an AI can fully troubleshoot without ever knowing what software, OS, or system it is looking at — is near impossible. But due to math, somehow possible.

---

## Protocol Flow

**Step 1 — Local Failure Detection:** The local watchdog detects a failure. The local LLM and repair scripts attempt to resolve it automatically. If local resolution fails, ZKDP initiates.

**Step 2 — Mandatory Snapshot:** Before any data is gathered or any action is ever taken, a snapshot of the affected service or LXC is taken. This is non-negotiable. No snapshot confirmation = no execution. Ever. The snapshot provider is environment-aware: Proxmox uses native LXC snapshot via Proxmox API, Docker uses container state snapshot, bare metal uses targeted config backup via timeshift or equivalent. Snapshots are per-service and per-LXC. Never host-level for a container-level repair. Each container is an isolated blast radius.

**Step 3 — Data Gathering:** AIWarden gathers all necessary troubleshooting data — logs, failure codes, service states, resource metrics, running services, dependency states. Everything needed to fully understand the failure. Nothing is filtered or excluded at this stage. Everything gets collected.

**Step 4 — Sanitization Pipeline:** All collected data passes through a mandatory 3-layer sanitization pipeline before any transmission occurs. Layer 1 is the Regex Tokenizer — every IP address, MAC address, hostname, domain, file path, port, API key, and credential pattern is replaced with a session-scoped anonymous token. Layer 2 is the Local LLM — a local 3B parameter model running on-device reviews Layer 1 output and catches anything regex missed including contextual PII, unusual formats, encoded values, and semantic identifiers. Layer 3 is the Egress Leak Check — a fail-closed verification pass where any value that pattern-matches as potentially real infrastructure data aborts transmission entirely. Partial sanitization is treated identically to no sanitization. If uncertain, abort.

**Step 5 — Mathematical Transformation via SFL:** Sanitized data is transformed into Sequence Formula Language (SFL) — a mathematical representation of events that preserves causal and temporal relationships without timestamps or real values. SFL primitives are: S(n) = sequence position n, Δ(n1,n2) = relative distance between two events, σ(n) = scale class of that distance, R(n) = recurrence count at position n. Scale classes are: σ0 = instant under 1 second, σ1 = rapid in seconds, σ2 = short in minutes to hours, σ3 = medium in hours to days, σ4 = long in days to weeks or more. No real timestamps ever exist in a ZKDP payload. Sequence ordering and scale classes provide all temporal reasoning capability the AI needs.

**Step 6 — Packet Construction:** Sanitized mathematically transformed data is assembled into a ZKDP surprise packet. All transmission defaults to single-pass — one complete payload, one AI response, one repair attempt. Round trips are available for complex multi-system failures where the AI requests additional targeted data. Packet types are: SPIKE for sudden single-sequence divergence, DRIFT for slow accumulated divergence crossing a cumulative threshold, CASCADE for multiple observables diverging within a σ0 window forming a causal chain, DRAIN for monotonic depletion of a finite resource heading toward a hard floor or ceiling with a PROJ field containing floor/rate/ttf, and ANOMALY for unrecognized failure shapes requiring human escalation. Packet format fields are: BASE for the baseline version both sides reference, SEQ for a monotonic counter replacing timestamps, TYPE for failure pattern classification, DIV for predicted vs actual state per observable, SCALE for SFL temporal distance classes, MAG for surprise magnitude from 0 to 1 which gates transmission, and PROJ for depletion projection on DRAIN type only.

**Step 7 — Two Channel Transmission:** ZKDP transmits across two strictly separated channels that never merge into a single payload. Channel A carries sanitized event data — the SFL packet with zero real values, all tokens, mathematical representation of the failure only. Channel B carries anonymous context — non-identifying software metadata only with no real values, pre-resolved from the local Service Profile Registry containing only semantic class information such as SERVICE_CLASS and KNOWN_ISSUE class and severity.

**Step 8 — AI Response Constraint:** The AI must respond exclusively in ZKDP protocol language. No natural language. No prose. No free text fields. A response that does not parse as valid ZKDP is rejected entirely and never reaches the repair engine. This is the single most important privacy rule in the protocol. A constrained response format makes inference leakage mathematically impossible — there is no field in the response schema capable of containing a real value. Response format fields are: BASE for baseline version, SEQ for sequence number, CONFIDENCE for a score from 0 to 1, ACTION for a whitelisted action against a token, VERIFY for the expected post-action state, and FALLBACK for the snapshot restore instruction if verification fails.

**Step 9 — Confidence and Whitelist Gating:** Before any repair action executes, two gates must both pass. The confidence gate requires Claude's confidence score to meet or exceed the configured threshold — below threshold nothing executes. The whitelist gate requires the proposed action to exist on the per-service whitelist maintained locally by AIWarden — if the action is not on the whitelist the repair engine physically cannot execute it regardless of what the AI recommended. Destructive commands do not exist on any whitelist. Ever.

**Step 10 — Execute, Verify, and Rollback:** The repair action executes against the snapshot-protected service. AIWarden collects the post-action state and sends a verification ZKDP packet. If the AI confirms resolution, the repair is committed and the shared baseline updates to reflect the new normal. If the AI determines the repair made things worse, automatic rollback triggers — the snapshot restores, state returns to pre-repair condition, and a human alert fires. The operator receives full context: what was attempted, what the outcome was, and what the current state is.

---

## Shared Baseline

Both the machine and the AI hold an agreed model of normal called the baseline. The baseline is established once at session start and referenced by version number in every subsequent packet. It contains the service set, their expected states, the SFL schema version, and the session token map. Neither side re-sends it after establishment. A packet is always read as a delta against the current baseline. No baseline confirmation = no packet accepted. A successful repair updates the baseline to reflect the new normal state going forward.

---

## Surprise Transmission Model

ZKDP only transmits when reality diverges from the baseline prediction. Silence is a valid and expected protocol state meaning everything matches what the AI already believes. A healthy infrastructure generates zero bytes. This is not compression — it is the deliberate decision to only communicate surprise. The unit of transmission is divergence, measured as the delta between predicted state and actual state. Low divergence below the configured MAG threshold does not generate a packet and quietly updates the local baseline instead. Cumulative drift that crosses the cumulative threshold generates a DRIFT packet even if no single step crossed the individual threshold, ensuring slow-creeping failures are never silently missed.

---

## Token Vault

The token vault is the mapping between real infrastructure values and their session-scoped anonymous tokens. It lives exclusively on the local machine. It never leaves the network. It is encrypted at rest. Values are decrypted into memory only during active sanitization operations and are zeroed from memory immediately after use. Token mappings rotate every session — no token from a previous session is ever reused in a new session, defeating long-term correlation attacks. The vault is single-writer with mutex locking to prevent race conditions during concurrent operations.

---

## Session Token Rotation

Every ZKDP session generates a completely fresh token map. IP_001 in session 1 is not IP_001 in session 2. An attacker holding packets from multiple sessions cannot correlate tokens across sessions because the mapping is different every time. This defeats statistical correlation attacks against long-running infrastructure monitoring.

---

## Canary Verification System

Before any live data is ever processed, AIWarden plants synthetic canary values in a test payload — values that look exactly like real IPs, API keys, hostnames, and credentials but are not. The sanitizer runs against the canary payload. Every single canary must be caught and tokenized. If even one canary passes through unsanitized, the system fails setup and refuses to proceed until the issue is resolved. The canary verification process runs on a completely separate codebase from the sanitizer itself. The thing checking the sanitizer shares no code with the sanitizer. This is the mathematical proof that the system works before it ever touches real data.

---

## Service Profile Registry

The Service Profile Registry is a locally-maintained, auto-updating knowledge base that resolves the AI's generic structural hypotheses into service-specific actionable guidance without ever revealing service identity to the AI. When the AI produces a hypothesis such as a dependency in the codec class is missing, the registry resolves that locally to the specific package name and documents the fix. The AI provided the reasoning. The registry provided the identity-specific facts. They never meet inside a payload. The registry updates itself by pulling from public bug trackers, CVE feeds, and release notes on a schedule. This is an outbound read of public data that reveals nothing about what software you run.

---

## Unknown Failure Behavior

An unknown bug still produces math. It still causes state transitions. It still generates divergence from baseline. It still has a shape. The AI still reasons over the causal structure of what happened even if it has never encountered that exact failure pattern before. If the ZKDP packet does not fit any known type, it is classified as ANOMALY. On ANOMALY: a snapshot is taken immediately, the packet is sent to the AI anyway with the ANOMALY flag for best-effort reasoning, and a human alert fires. The operator is never left with a silent failure. Worst case outcome is AIWarden saying I do not recognize this pattern, here is the math, human eyes are needed. That is still better than waking up to a dead server with no context.

---

## Failure Modes and Mitigations

**Race condition on startup:** Sanitizer must report READY before the watchdog is allowed to collect anything. Hard gate, no exceptions.

**Sanitizer crash mid-process:** Any sanitizer layer failure results in full abort. Partial sanitization is treated identically to no sanitization.

**New PPI pattern not in config:** The filesystem scanner and auto-discovery system continuously update the token vault. The egress check is the last line of defense and flags unknown patterns as suspicious rather than safe.

**Log encoding edge cases:** The log normalizer decodes all common encodings including unicode, base64, hex, and URL encoding before sanitization runs. Encoded values are decoded first, then sanitized.

**Token vault desync:** Vault writes are atomic, single-writer, and mutex locked. Concurrent write attempts queue, never race.

**Canary test bypass:** Canary verification runs on completely separate code from the sanitizer. Shared code between checker and checked is prohibited.

**Claude response inference leakage:** Eliminated entirely by the AI Response Constraint. Claude responds only in ZKDP protocol language. There is no field in the response schema capable of containing a real value.

**Session token correlation:** Eliminated by per-session token rotation. No token mapping persists across sessions.

**Memory tool leakage:** Memory entries are audited on every session start. Any memory entry containing pattern-matched PII is flagged for manual review before the session proceeds.

---

## Reference Implementation

AIWarden is the reference implementation of ZKDP, built for self-hosted homelab infrastructure running on Proxmox. It implements every component of this specification including the watchdog daemon, 3-layer sanitization pipeline, SFL transformation engine, two-channel packet builder, confidence and whitelist gating, snapshot integration, canary verification system, Service Profile Registry, and token vault. Repository: https://github.com/xADHDx/AI-Warden

---

## License

ZKDP is published under the GNU Affero General Public License v3.0. Any implementation of this protocol used in a networked product or service must open source that implementation under the same license. No exceptions.

---

*ZKDP was designed and authored by xADHDx in collaboration with Claude (Anthropic) on May 26, 2026. This document constitutes the original specification and establishes prior art for the Zero Knowledge Diagnostic Protocol.*