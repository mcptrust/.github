<div align="center">

# MCPTrust

**Hardening the Model Context Protocol.**

**The era of implicit trust is over.** As AI agents evolve from passive chatbots to autonomous executors, the interface between the model and the world—the **Model Context Protocol**—becomes the critical attack surface.

MCPTrust provides the **cryptographic control plane** for this new runtime.

We replace dynamic, probabilistic tool discovery with **deterministic security**. By locking server capabilities to **Ed25519** and **Sigstore** signatures, we enable engineering teams to establish a verifiable **Chain of Custody** for every tool their agents touch.

[View the Repository](#) · [Read the Documentation](#)

</div>

---

## Core Principles

### 1. Immutable Toolchains
Dynamic discovery is an operational risk. We capture the exact state of an MCP server—schemas, resources, and prompt definitions—into a verifiable `mcp-lock.json` artifact.

### 2. Zero-Trust Verification
We validate integrity before execution. Using local keys or OIDC-based Keyless Signing, agents can cryptographically verify that their tools haven't drifted or been tampered with.

### 3. Governance as Code
Security policy shouldn't be a PDF. We use **Common Expression Language (CEL)** to enforce granular rules (e.g., "Block all tools with `shell_exec` capability") directly in the CI pipeline.

---

## Get Started

Our primary repository contains the CLI, the specification, and the integration suite.

* [View the Repository](#)
* [Read the Documentation](#)

---

### Architecture

MCPTrust injects a security layer between tool development and agent runtime, enforcing a strict "Scan → Lock → Sign → Verify" pipeline.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                             MCPTrust Data Flow                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────┐       ┌──────────────┐       ┌──────────────┐                 │
│   │   MCP   │─────▶ │   SCANNER    │─────▶ │  ScanReport  │                 │
│   │ Server  │ STDIO │   (engine)   │       │    (JSON)    │                 │
│   └─────────┘       └──────┬───────┘       └──────┬───────┘                 │
│                            │                      │                         │
│          ┌─────────────────┴──────────────────────┴──────────────┐          │
│          │                      │                            │              │
│          ▼                      ▼                            ▼              │
│   ┌──────────────┐       ┌──────────────┐             ┌─────────────┐       │
│   │    POLICY    │       │    LOCKER    │             │   DIFFER    │       │
│   │   (engine)   │       │   (manager)  │             │  (engine)   │       │
│   │   CEL rules  │       │    SHA-256   │             │  jsondiff   │       │
│   └──────┬───────┘       └──────┬───────┘             └──────┬──────┘       │
│          │                      │                            │              │
│          ▼                      ▼                            ▼              │
│   ┌──────────────┐       ┌──────────────┐             ┌──────────────┐      │
│   │ PolicyResult │       │   Lockfile   │             │  DiffResult  │      │
│   │ (pass/fail)  │       │ mcp-lock.json│             │  (patches)   │      │
│   └──────────────┘       └──────┬───────┘             └──────────────┘      │
│                                 │                                           │
│                                 ▼                                           │
│                          ┌──────────────┐                                   │
│                          │    CRYPTO    │                                   │
│                          │    Ed25519   │                                   │
│                          └──────┬───────┘                                   │
│                                 │                                           │
│                                 ▼                                           │
│                          ┌──────────────┐                                   │
│                          │  Signature   │                                   │
│                          │  (.sig file) │                                   │
│                          └──────┬───────┘                                   │
│                                 │                                           │
│                                 ▼                                           │
│                          ┌──────────────┐                                   │
│                          │    BUNDLER   │                                   │
│                          │    (writer)  │                                   │
│                          └──────┬───────┘                                   │
│                                 │                                           │
│                                 ▼                                           │
│                          ┌──────────────┐                                   │
│                          │     .zip     │                                   │
│                          │    Bundle    │                                   │
│                          └──────────────┘                                   │
└─────────────────────────────────────────────────────────────────────────────┘