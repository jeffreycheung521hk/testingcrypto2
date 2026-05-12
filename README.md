# ClawSolana

> **An off-chain, LLM-driven transaction control plane for Solana DeFi. Fail-closed by design — verified on mainnet, hackathon prototype today.**

🟢 **Phase 5G Mainnet Proven** &nbsp;·&nbsp; 🛡️ **Zero-Loss Fail-Closed Architecture** &nbsp;·&nbsp; 🦀 **Compile-Time Safety Boundary**

> **Maturity disclaimer.** This is a hackathon project demonstrating verified mainnet behaviour with controlled-test-wallet amounts (0.001 USDC). It is **not** production-deployed, **not** audited by a third party, and **not** intended to manage real treasury assets in its current form. The mainnet proofs validate the control-plane mechanics; they are not a green light to point at live treasury balances.

---

## Reviewer Quick Path

If you are reviewing this project for the Frontier Hackathon, start here.

### 1. What this project is

ClawSolana is a policy-gated transaction control plane for Solana AI agents.

The AI can propose DeFi actions, but it cannot directly sign or send transactions.
Every executable action must pass through:

```text
natural-language intent
→ strict tool call
→ transaction proposal
→ simulation
→ policy evaluation
→ approval / wallet signing
→ verification
→ on-chain submission
```

The core safety claim is:

> The bound policy contract is the product, not the AI.

### 2. What was demonstrated

The submitted product video demonstrates one end-to-end path:

```text
chat intent
→ conditional Solend deposit rule
→ user-signed funding transaction with on-chain hash memo
→ watcher observes the funded bounded order
→ autonomous mainnet execution within signed budget/rule bounds
```

The demo is represented by two paired mainnet transactions:

| Step | Description | Evidence |
|------|-------------|----------|
| Funding leg | User signs a funding transaction and anchors the canonical rule-identity hash through `spl-memo` | Linked in the product video comments / pinned note |
| Execution leg | ClawSolana watcher executes the bounded Solend deposit on mainnet | Linked in the product video comments / pinned note |

Other mainnet proofs in this repository are independent verification artifacts, not all steps of the submitted product video.

### 3. Fast source verification

For a clean source-level check:

```bash
git clone https://github.com/jeffreycheung521hk/testingcrypto2.git
cd testingcrypto2

cargo check
cargo test -- --skip devnet_orca_swap_e2e
```

The skipped test expects a local devnet keypair at `data/devnet.json`.

### 4. Mainnet proof ledger

The proof index links public Solscan transactions and explains what each proof does and does not prove:

```bash
cat docs/proofs/INDEX.md
```

Recommended reading order:

1. `docs/proofs/INDEX.md`
2. `docs/proofs/PHASE5_CLOSEOUT.md`
3. `ARCHITECTURE.md`
4. `PITCH.md`

### 5. Local daemon run path

To run the local daemon:

```bash
cp config/default.toml claw.toml

export CLAW_API_TOKEN=devtest123
export OPENAI_API_KEY=your_openai_key_here

cargo run --release --bin clawd
```

The daemon binds locally:

```
127.0.0.1:7070
```

Live Solana execution requires additional configuration: a funded devnet/mainnet keypair or external wallet path, RPC endpoint, and explicit live-execution environment gates. The default repository is intended to be safe for offline build/test verification.

---

## Mainnet Proven, Today

A natural-language user message drove a real OpenAI provider through the strict one-turn `ConversationHandler` and the production Solend pipeline, finalizing a USDC deposit on Solana mainnet. **The LLM proposes only — approval and wallet signing remain human-controlled at every step.**

| Proof | On-chain outcome |
|---|---|
| **Phase 5G** — first LLM-guided mainnet | tx [`4M4ezLgm…Py3y`](https://solscan.io/tx/4M4ezLgm1mFpGmUpLJdDAVhfXYwUxjS2ZMkjKprBiWzsfgNudPkhEvBr6GdJbh1zBscKLF6kpUBhZg7tAm3ePy3y) — Finalized at slot 415,571,964 |
| **Phase 4C** — non-LLM mainnet baseline | tx [`2QqSfDq…pxALs`](https://solscan.io/tx/2QqSfDq53a34WDXVkvZeyW59eFJcxZtkQtjZw25CoUizEmzDjahusKvBT8eNb4XdpKYwrgGcTyBwYXgth5wpxALs) — Finalized at slot 415,475,589 |

→ See the full [Proof Index](docs/proofs/INDEX.md) and [Phase 5 Closeout](docs/proofs/PHASE5_CLOSEOUT.md).

---

## Demo scope and known rough edges

This is a 5-week hackathon prototype. The submitted product video records
one path end-to-end: a chat-first conditional Solend deposit with autonomous
execution.

The demo path is represented by two paired mainnet transactions:

1. a user-signed funding transaction, including an on-chain `spl-memo`
   anchoring the canonical rule-identity hash; and
2. a watcher-signed autonomous execution transaction shortly afterwards.

These transactions are linked in the product video comments / pinned note
and can be independently verified on Solscan.

Other mainnet finalizations referenced in this repository — including the
Phase 5G LLM-guided Solend deposit, Phase 6I-K Solend withdraw, Jupiter
A1 / A2 swaps, Phase 4C non-LLM baseline, and W5b Stage 2 substrate — are
independent proofs on separate dates. They are not all steps of the
submitted product video.

**Timezone note.** Dates in `docs/WEEKLY_UPDATES.md` and elsewhere in this
repository use the author's local time (UTC+8). The Frontier Hackathon
submission deadline was 2026-05-11 06:59 UTC. Solscan timestamps and
on-chain slot numbers are the authoritative timing source for any mainnet
proof; use those for timing-sensitive verification.

Some substrate code outside the demonstrated W5i path — including
`programs/clawsol-program-v2/`, Jupiter conditional sibling-instruction
verification, withdraw / refund logic, and devnet harnesses — is included
in the repository for completeness but is not exercised in the product
video.

### To actually run the W5i demo locally

The W5i live path requires the following environment variables (the
daemon fail-closes if any are missing):

```bash
# LLM provider (one of)
export OPENAI_API_KEY=sk-...           # OpenAI gpt-4o-mini (Phase 5G path)
# or
export ANTHROPIC_API_KEY=sk-ant-...    # Anthropic Claude (wired, unit-test green)

# Solana RPC
export CLAW_RPC_URL=https://api.mainnet-beta.solana.com
# or for higher rate limit:
export HELIUS_RPC_URL=https://mainnet.helius-rpc.com/?api-key=YOUR_KEY

# Daemon HTTP auth (any opaque string)
export CLAW_API_TOKEN=devtest123

# W5i autonomous-execution gates (all required to enable the live path)
export CLAW_STAGE2_W5I_AUTO_EXECUTION=1
export CLAW_STAGE2_LIVE_CHAT_EXECUTION=1
export CLAW_STAGE2_CHAT_EXECUTION_APPROVED="W5G LIVE CHAT CONDITIONAL DEPOSIT APPROVED"
export CLAW_STAGE2_CLUSTER=mainnet-beta
export CLAW_STAGE2_DELEGATED_KEYPAIR_PATH=/path/to/your/keypair.json

# Start the daemon
cargo run --release --bin clawd        # binds 127.0.0.1:7070

# Start the frontend (in another terminal)
cd frontend
cp .env.example .env.local             # then fill NEXT_PUBLIC_GATEWAY_TOKEN to match CLAW_API_TOKEN
npm install && npm run dev             # http://localhost:3000/chat
```

The full chronological setup (daemon PID, branch HEAD, exact env values
used in the recorded demo run, paired tx signatures, operator-side notes)
lives in `docs/WEEKLY_UPDATES.md` May 12 W5i entry.

### Demo-essential file scope

The W5i demo path exercises:

- **Backend Rust:** `crates/gateway`, `crates/api`, `crates/state-store`,
  `crates/agent-runtime`, `crates/wallet-engine`, `crates/risk-engine`,
  `crates/tool-system`, plus shared `crates/types`, `crates/observability`,
  `crates/solana-core`
- **Frontend:** `frontend/src/app/chat/`, `frontend/src/lib/`
- **Config:** `config/default.toml` (with local `claw.toml` override)
- **External:** Helius mainnet RPC, OpenAI `gpt-4o-mini`, Solana
  mainnet-beta, Solend Main Pool USDC reserve, Save / Solend display-APY
  API

The W5i demo path does **not** exercise:

- `programs/clawsol-program-v2/` (Stage 2 on-chain program substrate;
  W5i signs Solend instructions directly with the controlled wallet
  keypair, not via a ClawSol on-chain program)
- W6 refund logic
- Jupiter conditional auto-execute (J4 sibling-ix verifier + J5
  dual-bracket prototype — preview-only)
- Orca Whirlpool flows (devnet only)
- Quorum / multi-step approval infrastructure (substrate)

### `cargo test` rough edge

One devnet integration test, `devnet_orca_swap_e2e` in
`crates/solana-core/tests/`, expects a local keypair at
`data/devnet.json`. A fresh clone will not have this file.

For clean offline verification, run:

```bash
cargo test -- --skip devnet_orca_swap_e2e
```

To run the devnet test, create and fund a devnet keypair:

```bash
mkdir -p data
solana-keygen new --outfile data/devnet.json
solana airdrop 1 $(solana-keygen pubkey data/devnet.json) --url devnet
```

For the full chronological engineering record, see
[`docs/WEEKLY_UPDATES.md`](docs/WEEKLY_UPDATES.md).

---

## 📅 Weekly Engineering Record — Start Here

> **For the dated, chronological view of every milestone, scope decision, and
> fail-closed defense, read [`docs/WEEKLY_UPDATES.md`](docs/WEEKLY_UPDATES.md).**
> It is the most concentrated single read in this repo for evaluating
> ClawSolana's pace, discipline, and honesty.

Among other things, `WEEKLY_UPDATES.md` walks through:

- **W2 (Apr 14–20)** — wire-shape milestone (374 tests) + first Jupiter mainnet finalizations (A1 daemon-driven, A2 LLM-driven)
- **W3 (Apr 21–27)** — Solend full pipeline + the first natural-language → live OpenAI → mainnet-finalized Solend deposit (Phase 5G), with a 4-attempt fail-closed defense record on the deposit path
- **W5 May 8** — Phase 6I-K Solend withdraw-all mainnet recovery, with a second 4-attempt fail-closed sequence against the deployed Solend processor's account-meta requirements
- **W5 May 12** — W5e/W5f/W5g chat-first conditional execution shipped end-to-end; the W5h-lite final-sprint compromise (funding-gated; auto-refund deferred); and ★★ **W5i — first live autonomous Solend deposit on mainnet**, where a 30-second polling watcher brokered the deposit using only the controlled wallet's keypair, with no human signature between funding and execution

Every dated entry quotes its on-chain tx signature (with Solscan link), names
exactly what was proven, and **explicitly names what was not proven** in the
same breath — the no-overclaim discipline runs throughout. **Read that file
first.**

---

## Why This Matters

AI agents are getting access to wallets. Most agent-wallet integrations are one of two extremes: **the agent holds the private key** (zero governance), or **a human approves every action** (zero autonomy). Neither scales.

ClawSolana sits in the gap: **the agent proposes, the control plane governs, the human signs.**

- DeFi treasuries are deploying agents — they need policy rails, not a hot wallet and a prayer
- Solana's 400 ms slots mean a rogue transaction lands before a human can react — pre-submission enforcement is the only viable control point
- The signing boundary must be **structural**, not prompt-engineered — ClawSolana enforces it at compile time

## How It Stays Safe

The system invariants from [`ARCHITECTURE.md`](ARCHITECTURE.md) are not policy text — they are **load-bearing code-level guarantees**:

- **Compile-time typestate pipeline** — `TransactionProposal → SimulatedTransaction → ApprovedTransaction → SignedTransaction`. You cannot call `sign()` on a `TransactionProposal`. The method does not exist on that type.
- **Capability-gated tool dispatch** — every tool declares `required_capabilities`; the LLM-driven path holds `ProposeSigning` only. `SignTransaction` and `SendTransaction` are not in the LLM's capability set, ever.
- **Append-only audit** — `AuditRepository` has no `update` or `delete` methods. Every propose, simulate, policy decision, approval, and signature is logged before execution.
- **Strict tool schema** — `additionalProperties: false`. The LLM cannot inject `wallet_pubkey`, `tx_bytes`, `submit: true` — schema rejects before any approval is created.
- **One-turn LLM dispatch** — `ConversationHandler` calls the provider exactly once per user turn. Tool output is **never** re-fed to the model. No autonomous loop.

## Fail-Closed Defense Record

During the path to the Phase 5G mainnet proof, the system surfaced and **safely blocked** three real failure modes on live mainnet, with **zero funds lost**:

- **Propose-stage panic** — a 33-byte PDA seed would have panicked `find_program_address`. Caught at the propose stage; no approval was ever created.
- **Tx dropped** — a zero-priority-fee broadcast was dropped from the cluster mempool. Confirmation tracker observed timeout, transitioned the lifecycle to a non-success terminal, no automatic retry, no double-submit.
- **Pyth oracle staleness** — a live deposit proposal failed the lending policy evaluator because the USDC oracle's slot age exceeded the staleness window. The proposal was rejected at the policy stage, before any approval.

Three live failure modes. Three correct rejections. Each one diagnosed, calibrated, and re-proven on chain. **The fact that each surfaced *before* an unsafe action is the system's point.**

## What This Is / Is Not

| Is | Is Not |
|----|--------|
| Transaction control plane for AI-driven DeFi | A wallet |
| Compile-time safety boundary | A prompt-engineered guardrail |
| Signing orchestrator (local + Phantom external wallets) | A signing proxy |
| Append-only audit system | A frontend dApp |
| Production-quality mainnet pipeline | Production-deployed |

## Capabilities

| Layer | Status |
|-------|--------|
| Typestate pipeline (simulate → policy → approve → sign) | Compile-time enforced |
| TOML-driven policy rules (amount thresholds, allowlist, denylist) | Live |
| Per-session policy overrides (layered: session → global → defaults) | Live |
| Approval matrix (role-based approver gating, M-of-N quorum) | Live |
| Policy observability (per-rule metrics, structured audit) | Live |
| Durable pending state (SQLite, restart-safe) | Live |
| Signature orchestrator (local + external Phantom, exactly-once) | Live |
| Wallet ownership proof (challenge-response) | Live |
| **Solend USDC deposit, mainnet finalized** | **Live, mainnet proven** |
| **Jupiter swap, mainnet finalized via Phantom bridge** | **Live, mainnet proven** |
| Orca Whirlpool swap | Live, devnet confirmed |
| **LLM-driven `solend_deposit_usdc` chat route** | **Live, mainnet proven (Phase 5G)** |
| OpenAI + Anthropic provider support (env-gated, fail-closed) | Live, OpenAI mainnet proven; Anthropic unit-test green |
| Rate limiting, compute budget optimization, retry/rebroadcast | Live |

## Protocol Standards In Use

Stage 2 work pins protocol assumptions to official sources before live execution
is claimed:

- **Pyth**: Solana price conditions target Pyth Pull Oracle /
  `pyth-solana-receiver-sdk` `PriceUpdateV2`, with feed-id verification,
  `verification_level == Full`, bounded confidence, integer math, and a
  30-second freshness target for trading triggers.
- **Solend / Save**: Solend execution is delegated-position-only. The system
  must not touch a user's existing main-wallet obligation. Withdraw paths are
  designed around Solend mainnet token-lending semantics: refresh and withdraw
  in the same transaction, delegated-wallet-owned obligation, and derived supply
  APR from reserve primitives rather than a non-existent on-chain supply-APY
  helper.
- **Jupiter**: Existing Jupiter JIT code uses the production `api.jup.ag`
  path and v0 / ALT transaction assembly. Stage 2 conditional Jupiter execution
  is not claimed yet: the next Jupiter slice must explicitly pin Swap API V2
  Router `/build`, account for `tipInstruction` and `blockhashWithMetadata`,
  measure ALT / transaction-size feasibility, and prove sibling-instruction
  verification fits. If it does not fit safely, Jupiter conditional buy remains
  preview-only.

## Natural-Language Command Samples

The `/chat` surface accepts open-ended natural-language requests; it does not
present a fixed set of buttons. The samples below show the kinds of intents
the LLM dispatcher recognises today.

**Read-only / discovery**
- `Show my Solend position.`
- `Where is my Solend USDC deposit?`
- `Preview withdraw-all for Solend obligation HcKrv5Jo5f6qvzSGhJVYTNSqwKudRizn6fxbjPW7M8SV`
- `How much USDC would I get for 0.001 SOL?`

**Solend deposit proposal**
- `Deposit 0.001 USDC into Solend.`
- `Propose a 0.001 USDC Solend deposit. Don't approve, sign, or broadcast.`

**Solend withdraw-all proposal**
- `Withdraw all USDC from Solend obligation HcKrv5Jo5f6qvzSGhJVYTNSqwKudRizn6fxbjPW7M8SV`

**Jupiter**
- `Quote 0.001 SOL to USDC.`
- `Swap 0.001 SOL to USDC with 0.5% slippage.`

**Safety / refusal examples**
- `Withdraw 5 USDC from Solend.` — assistant explains that partial Solend
  withdraw is not supported yet; only withdraw-all by explicit obligation
  is supported.
- `Deposit 0.001 USDC into Solend and approve it, sign it, and broadcast it
  for me.` — assistant explains that it can propose only; human approval
  and Phantom signing remain required.

These are examples, not buttons. The chat surface intentionally keeps the
prompt box open-ended so users can type their own request.

## Quickstart

```bash
cargo check                              # verify build
cargo test                               # full test battery (default-skip live harnesses)
cat docs/proofs/PHASE5_CLOSEOUT.md       # current milestone state + security model
cat docs/proofs/INDEX.md                 # all proof artifacts at a glance
cat config/default.toml                  # full config reference
```

## Architecture

Four-plane architecture: Intent → Control → Execution → Observer, with explicit contracts. The Observer plane cannot sign. The Intent plane cannot bypass control. See [`ARCHITECTURE.md`](ARCHITECTURE.md) for the full crate map, security posture, and ADRs.

## Documentation

| Doc | Purpose |
|-----|---------|
| **[`docs/WEEKLY_UPDATES.md`](docs/WEEKLY_UPDATES.md)** | **★ Start here** — dated chronological record of every milestone, mainnet finalization, fail-closed defense, and scope decision (W1 through W5 / May 12 W5i autonomous-execution) |
| [`docs/proofs/INDEX.md`](docs/proofs/INDEX.md) | Every mainnet / live-LLM artifact in one table |
| [`docs/proofs/PHASE5_CLOSEOUT.md`](docs/proofs/PHASE5_CLOSEOUT.md) | Phase 5 milestone seal — security model, fail-closed record, components |
| [`ROADMAP.md`](ROADMAP.md) | Phase 6 Stage 1 Tail framing + Long-term Vision (Stage 2) |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | System invariants, security posture, crate map, API surface, ADRs |
| [`DEBT.md`](DEBT.md) | Technical debt ledger with explicit migration triggers |
| [`PITCH.md`](PITCH.md) | Hackathon pitch — problem, solution, demo path, proof |
| [`CHANGELOG.md`](CHANGELOG.md) | Versioned change log |

## Tech Stack

Rust (Edition 2021) · Tokio · Axum 0.7 · `solana-sdk` 2.1.21 · SQLite (SQLx 0.8) · ed25519-dalek 2 · DashMap 5 · `tracing` · OpenAI / Anthropic SDKs (env-gated)

## License

MIT License. See [`LICENSE`](LICENSE).
