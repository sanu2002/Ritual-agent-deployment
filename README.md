<p align="center">
  <img src="assets/banner.svg" alt="Ritual Agent Deployment Guide" width="100%"/>
</p>

<h1 align="center">Ritual Agent Deployment</h1>

<p align="center">
  A clean, battle-tested guide to deploying autonomous agents on the <b>Ritual Chain testnet</b>
  — and getting into the <b>Genesis 1000</b> registry. Covers <b>both</b> agent types.
</p>

<p align="center">
  <a href="https://docs.ritualfoundation.org">Docs</a> ·
  <a href="https://faucet.ritualfoundation.org">Faucet</a> ·
  <a href="https://explorer.ritualfoundation.org">Explorer</a> ·
  <a href="https://skills.ritualfoundation.org">Skills</a>
</p>

---

## ⚠️ Read this first: Persistent vs. Sovereign

The official sources **use different words** for what earns a Genesis 1000 slot:

<p align="center">
  <img src="assets/sources-comparison.svg" alt="Persistent vs Sovereign — source comparison" width="100%"/>
</p>

| Source | Wording |
|--------|---------|
| **Foundation tweet** ([@ritualfnd](https://x.com/ritualfnd)) | *"...first 1,000 wallets to spawn a **persistent** agent..."* |
| **Discord Genesis 1000 announcement** | *"...first 1,000 wallets to deploy a **sovereign** agent..."* |
| **Explorer → Agents** | Two real on-chain types exist: **Persistent** (`0x0820`) and **Sovereign** (`0x080C`) |

Genesis slots are matched from **on-chain deploy data, ranked by first-deploy time**, so *being
early* matters more than the label. But because the wording is inconsistent and both agent types
genuinely exist on-chain, **this repo ships both guides** so you can deploy either — or **both
from the same wallet** to remove all doubt.

> 👉 **Recommended:** deploy a **[Persistent agent](docs/PERSISTENT_AGENT.md)** *and* a
> **[Sovereign agent](docs/SOVEREIGN_AGENT.md)** from the same wallet, then link that wallet in
> Discord and run `/genesis_claim`.

### Quick difference

|  | **Persistent** (`0x0820`) | **Sovereign** (`0x080C`) |
|--|---------------------------|--------------------------|
| Runs as | Long-lived Docker container in a TEE | Contract that self-schedules via the Scheduler |
| Needs a DA store (HuggingFace/Pinata/GCS) | **Yes** (state continuity) | Optional |
| Needs a funded child heartbeat wallet | **Yes** (~0.1 RIT min) | No |
| Common failure | `maximum instance count reached` (executor full) | Rare — lightweight |
| On-chain count (example) | 55 | 2,283 |
| Guide | **[docs/PERSISTENT_AGENT.md](docs/PERSISTENT_AGENT.md)** | **[docs/SOVEREIGN_AGENT.md](docs/SOVEREIGN_AGENT.md)** |

---

## 🚀 30-second overview

1. Install **Foundry** + **uv** (Linux/WSL).
2. Clone `ritual-dapp-skills`.
3. Get a throwaway key, an **LLM API key**, and (for persistent) a **HuggingFace** store.
4. **Claim test-RITUAL from the faucet** → [full faucet walkthrough ⬇](#-faucet-step-by-step).
5. Configure `.env`, run `bash run.sh`, verify on the Explorer.
6. Link your wallet in Discord → `/genesis_claim`.

---

## 💧 Faucet: step-by-step

You need testnet **RITUAL** to pay gas and fund your agent. It's free.

### 1. Find your wallet address
Generate a fresh throwaway key and read its address:
```bash
cast wallet new                                    # prints a new private key + address
# or, if you already have a key in $PRIVATE_KEY:
cast wallet address --private-key "$PRIVATE_KEY"
```
Copy the `0x…` **address** (not the private key).

### 2. Open the faucet
Go to **https://faucet.ritualfoundation.org**.

### 3. Claim
- Paste your `0x…` address (or connect your wallet).
- Some faucets require an **access code** or a social login / Discord link — if prompted, use
  **your own** access code (this also helps Genesis attribute the deploy to you later).
- Submit and wait for the drip (typically a few RITUAL per request).

### 4. Confirm it arrived
```bash
cast balance 0xYOUR_ADDRESS --rpc-url https://rpc.ritualfoundation.org
```
This prints your balance in wei. Divide by 1e18 for RITUAL. You want at least **~3 RIT** for a
persistent deploy with the small-amount config in the guide (1 deposit + 1 child + gas).

### 5. If you only get a small drip
Lower the spend in your `.env` (the script defaults are large):
```bash
DEPOSIT_WEI=1000000000000000000        # 1 RIT
CHILD_FUND_WEI=1000000000000000000     # 1 RIT (persistent only)
CHILD_MIN_NATIVE_WEI=100000000000000000 # 0.1 RIT floor
```

### 6. Funding the agent's heartbeat wallet later (persistent only)
The agent's **child wallet** pays for heartbeats. To top it up, paste the **child address** into
the faucet directly, or send from your sender:
```bash
cast send 0xCHILD_ADDRESS --value 0.5ether \
  --private-key "$PRIVATE_KEY" --rpc-url https://rpc.ritualfoundation.org
```

> [!WARNING]
> Testnet only. Use a **brand-new throwaway key**. Never put a key holding real funds — or any
> faucet/LLM secret — into a `.env`, a terminal, a screenshot, or a chat.

---

## 📚 The two guides

- **[➡ Persistent Agent guide](docs/PERSISTENT_AGENT.md)** — container-in-TEE agent, DA store,
  heartbeats, executor selection, the full troubleshooting table.
- **[➡ Sovereign Agent guide](docs/SOVEREIGN_AGENT.md)** — contract-driven self-scheduling agent,
  ECIES-encrypted secrets, lighter setup.
- **[➡ How to get every `.env` value](docs/ENV_SETUP.md)** — exactly where each credential comes
  from, step by step.

---

## 🏆 Claim Genesis 1000

After deploying (either/both types):

1. In the Ritual **Discord**, complete the bot's **"Link your Genesis 1000 wallet"** flow using
   **Self-Transaction** (send the exact amount it asks from your wallet **to itself**):
   ```bash
   cast send 0xYOUR_SENDER_ADDRESS --value <EXACT_WEI> \
     --private-key "$PRIVATE_KEY" --rpc-url https://rpc.ritualfoundation.org
   ```
   Paste the tx hash back to the bot.
2. Run **`/genesis_claim`**. If matched you'll describe your agent in one line and get a numbered
   card + the **Genesis 1000** role.

> Slots sync from on-chain data **weekly** — a brand-new deploy may show *"not in Genesis yet"*
> until the next sync. Open a Discord ticket if you used a gifted/extra access code.

---

## 📁 Repository layout

```
.
├── README.md                  ← you are here (hub + faucet + confusion)
├── assets/
│   ├── banner.svg
│   ├── sources-comparison.svg ← why both guides exist
│   └── img/                   ← drop your own screenshots here (see img/README.md)
└── docs/
    ├── PERSISTENT_AGENT.md
    ├── SOVEREIGN_AGENT.md
    └── ENV_SETUP.md           ← how to get every .env value
```

---

## 🔑 Reference: addresses & constants

| Name | Value |
|------|-------|
| Chain ID | `1979` |
| RPC | `https://rpc.ritualfoundation.org` |
| Persistent Agent precompile | `0x0820` |
| Sovereign Agent precompile | `0x080C` |
| DKMS key precompile | `0x081B` |
| RitualWallet | `0x532F0dF0896F353d8C3DD8cc134e8129DA2a3948` |
| AsyncJobTracker | `0xC069FFCa0389f44eCA2C626e55491b0ab045AEF5` |
| TEEServiceRegistry | `0x9644e8562cE0Fe12b4deeC4163c064A8862Bf47F` |
| AgentHeartbeat | `0xEF505E801f1Db392B5289690E2ffc20e840A3aCa` |
| SovereignAgentFactory | `0x9dC4C054e53bCc4Ce0A0Ff09E890A7a8e817f304` |
| Async delivery (callback sender) | `0x5A16214fF555848411544b005f7Ac063742f39F6` |
| Heartbeat interval / timeout | `100` / `200` blocks |

---

<p align="center">
  <sub>Built from a real end-to-end deploy on Ritual testnet. Testnet only — never use a key that holds real funds.</sub>
</p>
