# Project Aegis — LLM Security Gateway

[![Security Engineering](https://img.shields.io/badge/Domain-Security%20Engineering-243B53)](https://en.wikipedia.org/wiki/Information_security)
[![DLP](https://img.shields.io/badge/Spec-DLP%20%26%20Sanitization-CC0000)](https://en.wikipedia.org/wiki/Data_loss_prevention_software)
[![Cryptography](https://img.shields.io/badge/Spec-Key%20Management-6F42C1)](https://en.wikipedia.org/wiki/Key_management)
[![Zero Trust](https://img.shields.io/badge/Architecture-Zero%20Trust-005A9C)](https://en.wikipedia.org/wiki/Zero_trust_security_model)
[![RAG Security](https://img.shields.io/badge/Spec-Vector%20DB%20Security-F7941E)](https://en.wikipedia.org/wiki/Retrieval-augmented_generation)

The cybersecurity workstream of **Project Aegis** — a centralised reverse-proxy security gateway sitting between enterprise O&G endpoints and third-party LLMs. This repository contains the full security specification suite authored for Phase 1 and Phase 2, covering DLP sanitization, cryptographic key management, vector database security, and tamper-evident audit telemetry.

> **Core principle:** the gateway is only as trustworthy as the controls that enforce it. Every specification includes named acceptance criteria and verification evidence so compliance is demonstrable, not assumed.

## Project Overview

Aegis intercepts every prompt, system message, and tool payload before it reaches a third-party model — and every document before it enters the RAG knowledge base. The cybersecurity workstream defines the contracts that make that proxy safe across four interdependent layers.

- **Role:** Cybersecurity Engineer — Core Specialist
- **Designation:** Abdullah Zubair
- **Status:** Active Directive
- **Programme:** Phase 1 (Weeks 1–4) → Phase 2 (Weeks 5–8) → Phase 3 (Weeks 9–12)

## Specifications

| Document | Title | Phase |
|---|---|---|
| [AEGIS-SEC-DLP-001](01-dlp-pattern-spec.pdf) | DLP Pattern & Sanitization Specification | Phase 1 |
| [AEGIS-SEC-KMS-002](02-key-management-runbook.pdf) | Cryptographic Key Management Runbook | Phase 1 |
| [AEGIS-SEC-VDB-003](03-vector-db-security-standard.pdf) | Vector Database Security Standard | Phase 2 |
| [AEGIS-SEC-AUD-004](04-audit-telemetry-spec.pdf) | Audit Logging & Security Telemetry Spec | Phases 1–3 |

## DLP Pattern & Sanitization (AEGIS-SEC-DLP-001)

Defines the sanitization contract for every prompt and RAG ingestion payload — five sensitivity classes, a four-layer detection engine, and a tokenization contract that preserves model coreference without exposing plaintext.

| Class | Content | Action |
|---|---|---|
| C1 Secrets | API keys, private keys, JWTs, DB connection strings, cloud credentials | Block |
| C2 Financial | Payment cards, IBANs, financial identifiers | Tokenize |
| C3 PII | Names, national IDs, emails, phones, addresses | Tokenize |
| C4 Proprietary code | Internal repos, large code spans, internal hostnames | Redact + warn |
| C5 Project confidential | Document refs, unreleased roadmap, client/partner names | Redact + warn |

Detection runs four layers in order — regex, validators (Luhn, mod-97, Shannon entropy), local on-premise NER, and domain deny-lists — with a match acted on only after validation. Patterns ship as signed, versioned config in Git and hot-reload without a gateway redeploy.

**Phase 1 exit targets:** ≥0.98 recall on C1 secrets · ≤2% false-positive rate · p95 latency added under 50 ms · fail-closed on engine error.

## Cryptographic Key Management (AEGIS-SEC-KMS-002)

Defines the key hierarchy, custody rules, and rehearsed rotation, revocation, and recovery procedures for every key Aegis depends on.

```
Root KMS key (HSM-backed · FIPS 140-2 L3 · never exported)
├── Vector store DEK     AES-256-GCM at rest
├── Audit signing key    Ed25519 · sign only
├── Tokenization DEK     request-scoped in-memory vault
└── Session DEK          gateway state · cache
```

LLM provider credentials sit outside this tree in the secrets manager, delivered to workloads through short-lived OIDC federation — never from environment files, container images, or source control.

## Vector Database Security (AEGIS-SEC-VDB-003)

Mandatory controls for Qdrant/Milvus RAG stores holding the enterprise knowledge base. Every control has a named verification so compliance is demonstrable.

| Control | Requirement |
|---|---|
| VDB-01 | No public endpoint — private subnet only, ingress restricted to gateway and ingestion workers |
| VDB-02 | Authentication enforced on every node; default open mode explicitly disabled |
| VDB-03 | mTLS on the data path and cluster-internal traffic |

> A retrieval bug is not a bug — it is a disclosure.

## Audit Logging & Telemetry (AEGIS-SEC-AUD-004)

Records metadata, classification decisions, and content hashes — never prompt or completion bodies. The audit trail proves what happened without becoming a second copy of the sensitive data it protects.

```json
{
  "event_id": "uuid-v7",
  "ts": "2026-08-17T09:41:22.104Z",
  "actor": { "user_id": "...", "roles": ["finance"], "dept": "..." },
  "dlp": { "pack": "v1.0", "classes": ["C3"], "action": "tokenize", "match_count": 2, "latency_ms": 31 },
  "prompt_sha256": "...",
  "verdict": "allowed"
}
```

## Phase Roadmap

| Phase | Weeks | Focus | Primary Output |
|---|---|---|---|
| Phase 1 — Foundation | 1–4 | Proxy Security Gateway & Sanitization | DLP engine rules & Key Management |
| Phase 2 — RAG & Analytics | 5–8 | Private Vector DB Security | RBAC & Audit Telemetry |
| Phase 3 — Rollout | 9–12 | Enterprise Production Deployment | Pen-testing & Security Sign-off |

## Cross-Functional Assignments

| Task | Cross-Functional Lead |
|---|---|
| Zero-Trust Access Policies for RAG Data Stores | Atif Shamim (AI App Dev) & Swaadesh U.S (Data Analytics) |
| Gateway Real-Time Threat & Anomaly Logging | Arjun Pathan (Engineering Execution Lead) |
| Admin Console Security Dashboard Integration | Khushi Vaishnav (UX & Front-End Developer) |
| Production Penetration Testing & Final Audit Sign-Off | Jamil Jawed (QA Testing Intern) |

## Repository Structure

```text
.
├── README.md
├── Abdullah Task.pdf                                               # Task assignment brief
├── Project_Aegis_Cybersecurity_Workstream_Security_Specification_v1.0.pdf
├── Project_Aegis_Cybersecurity_Workstream_Security_Specification_v1.0.docx
├── 01-dlp-pattern-spec.pdf                                         # AEGIS-SEC-DLP-001
├── 02-key-management-runbook.pdf                                   # AEGIS-SEC-KMS-002
├── 03-vector-db-security-standard.pdf                              # AEGIS-SEC-VDB-003
└── 04-audit-telemetry-spec.pdf                                     # AEGIS-SEC-AUD-004
```

## Limitations

- Specifications are Phase 1 and Phase 2 deliverables; Phase 3 production deployment and penetration testing sign-off are out of scope for this repository.
- Pattern pack v1 covers initial deployment countries; national ID patterns are extended through the signed update process.
- Acceptance criteria targets are defined for Phase 1 exit; Phase 3 load-test evidence is not yet included.
- This repository contains specification documents only — no production code, credentials, or environment configuration is included.

## Lessons Learned

- A proxy is only as safe as the sanitization contract it enforces — vague policy produces vague protection.
- Tokenization preserves model utility without exposing plaintext; redaction trades utility for certainty.
- Versioned, hot-reloadable pattern packs decouple policy updates from gateway deployments.
- Envelope encryption scopes compromise: one DEK per purpose means a single key breach is never total.
- Audit logs that record content hashes rather than bodies prove what happened without becoming a second data exposure.

## Author

**Abdullah Zubair**  
Cybersecurity | GRC | Security Automation
- GitHub: [@AvatarParzival](https://github.com/AvatarParzival)
- LinkedIn: [Abdullah Zubair](https://www.linkedin.com/in/abdullahzubairr)
- Email: [abdullah69zubair@gmail.com](abdullah69zubair@gmail.com)

## Responsible Use

This repository is intended for educational and professional portfolio purposes. All specifications were produced in a training and internship context. No production credentials, environment files, or proprietary system details are included. Classification markings are reproduced for portfolio authenticity only.
