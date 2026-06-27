<h1 align="center">How to get every <code>.env</code> value</h1>

<p align="center">Exactly where each variable comes from, step by step.</p>

<p align="center"><a href="../README.md">← back to hub</a></p>

---

This page explains **how to obtain** every value you put in `.env`. Copy the table for your agent
type, then follow the numbered "how to get it" steps below.

> [!WARNING]
> Every secret below (`PRIVATE_KEY`, the LLM key, `HF_TOKEN`, `DA_PINATA_JWT`, GCS JSON) is
> sensitive. Keep them only in your local `.env` — which is **git-ignored** in this repo. Never
> paste them into screenshots, chats, or commits.

---

## Variable reference

| Variable | Used by | Required? | What it is |
|----------|---------|-----------|------------|
| `RPC_URL` | both | ✅ | Ritual testnet RPC endpoint |
| `PRIVATE_KEY` | both | ✅ | Your throwaway wallet key (pays gas, owns the agent) |
| `ANTHROPIC_API_KEY` *(or OPENAI/GEMINI/OPENROUTER)* | both | ✅ (one) | LLM provider key the agent thinks with |
| `MODEL` | sovereign (optional on persistent) | ⬜/✅ | Exact provider-routable model id |
| `DA_PROVIDER` | persistent | ✅ | `hf` / `pinata` / `gcs` |
| `HF_TOKEN` | persistent (hf), sovereign | ✅ (hf) | HuggingFace **write** token |
| `HF_REPO_ID` | persistent (hf), sovereign | ✅ (hf) | HuggingFace dataset id `user/repo` |
| `DA_PINATA_JWT` | persistent (pinata) | ✅ (pinata) | Pinata API JWT |
| `DA_PINATA_GATEWAY` | persistent (pinata) | ⬜ | Your Pinata dedicated gateway URL |
| `GCS_DA_SERVICE_ACCOUNT_JSON` / `GCS_DA_BUCKET` | persistent (gcs) | ✅ (gcs) | GCS service-account JSON + bucket |
| `AGENT_RPC_URL` | persistent | ✅ | RPC the **agent container** uses (set to public RPC) |
| `EXECUTOR_TEE_ADDRESS` | both | ⬜ | Pin a specific executor (avoid the full default) |
| `DEPOSIT_WEI` / `CHILD_FUND_WEI` / `CHILD_MIN_NATIVE_WEI` / `MIN_RITUAL_WALLET_WEI` | persistent | ⬜ | Amount overrides sized to your faucet drip |
| `CLI_TYPE` / `PROMPT` | sovereign | ⬜ | Agent runtime + first prompt |

---

## How to get each value

### 1. `RPC_URL`
No signup. It's the public testnet endpoint:
```bash
RPC_URL="https://rpc.ritualfoundation.org"
```
Verify it's live: `cast chain-id --rpc-url https://rpc.ritualfoundation.org` → should print `1979`.

### 2. `PRIVATE_KEY` (+ your address)
Generate a **fresh throwaway** key (never reuse a real one):
```bash
cast wallet new
# Successfully created new keypair.
# Address:     0x74E5....56c2
# Private key: 0xd8ba....fc7d   <-- this goes in PRIVATE_KEY
```
Put the **private key** in `.env`; use the **address** at the faucet.
```bash
PRIVATE_KEY="0xd8ba....fc7d"
```

### 3. `ANTHROPIC_API_KEY` (or another LLM key)
This bills real money — guard it.
- **Anthropic:** console.anthropic.com → **API Keys** → **Create Key** → copy `sk-ant-...`
- **OpenAI:** platform.openai.com/api-keys → **Create new secret key** → `sk-...`
- **Gemini:** aistudio.google.com/app/apikey → **Create API key**
- **OpenRouter:** openrouter.ai/keys → **Create Key**

Set **exactly one**:
```bash
ANTHROPIC_API_KEY="sk-ant-..."
```

### 4. `MODEL` (sovereign requires it; persistent defaults by provider)
The exact routable model id for your provider:
| Provider | Example `MODEL` |
|----------|-----------------|
| Anthropic | `claude-sonnet-4-5-20250929` |
| OpenAI | `gpt-4o-mini` |
| Gemini | `gemini-2.5-flash` |
| OpenRouter | `anthropic/claude-sonnet-4.5` |
```bash
MODEL="claude-sonnet-4-5-20250929"
```

### 5. `DA_PROVIDER` + storage credentials (persistent)
Pick **one** provider, then get its creds:

**HuggingFace (`hf`) — easiest**
1. Create an account at huggingface.co
2. Create a **dataset**: huggingface.co/new-dataset → name e.g. `my-agent-store`
3. Create a **Write** token: huggingface.co/settings/tokens → **New token → type: Write** → copy `hf_...`
```bash
DA_PROVIDER="hf"
HF_TOKEN="hf_..."
HF_REPO_ID="your-username/my-agent-store"   # user/repo form, NOT a URL
```

**Pinata (`pinata`) — one credential**
1. Sign up at pinata.cloud
2. **API Keys → New Key** → enable Pinning scopes → copy the **JWT** (`eyJ...`, shown once)
3. (optional) **Gateways** tab → your `https://<name>.mypinata.cloud`
```bash
DA_PROVIDER="pinata"
DA_PINATA_JWT="eyJ..."
# DA_PINATA_GATEWAY="https://your-name.mypinata.cloud"
```

**GCS (`gcs`)**
1. Create a bucket you own
2. Create a service account with write access → download its JSON key
```bash
DA_PROVIDER="gcs"
GCS_DA_SERVICE_ACCOUNT_JSON='{"type":"service_account", ...}'
GCS_DA_BUCKET="my-bucket"
# GCS_DA_PREFIX="agents/demo"
```

### 6. `AGENT_RPC_URL` (persistent — important)
The RPC the **agent container** uses to read the chain and post heartbeats. The script default
`http://172.17.0.1:8545` only works if the chain runs on the same Docker host — it does **not**
for remote executors. Always set the public RPC:
```bash
AGENT_RPC_URL="https://rpc.ritualfoundation.org"
```

### 7. `EXECUTOR_TEE_ADDRESS` (optional, both)
Avoid the full default executor. List all of them and copy a `tee=0x…` that isn't index 0:
```bash
uv run --with web3 python3 list_executors.py
# [0] tee=0x9dc1...8b4C   <-- default, usually full
# [13] tee=0x79ED...843F  <-- pick one like this
EXECUTOR_TEE_ADDRESS="0x79ED9c8D5A3b3a11B9f82DB6eeAC3313f86B843F"
```

### 8. Amount overrides (persistent — size to your faucet)
After claiming from the faucet, check your balance and set amounts so the run fits:
```bash
cast balance 0xYOUR_ADDRESS --rpc-url "$RPC_URL"   # wei; /1e18 = RIT
DEPOSIT_WEI=1000000000000000000            # 1 RIT
CHILD_FUND_WEI=1000000000000000000         # 1 RIT
CHILD_MIN_NATIVE_WEI=100000000000000000    # 0.1 RIT
MIN_RITUAL_WALLET_WEI=1000000000000000000  # 1 RIT
```
> Wei cheat-sheet: `1 RIT = 1000000000000000000` (1 followed by 18 zeros). `0.1 RIT` = 17 zeros.

### 9. `CLI_TYPE` / `PROMPT` (sovereign, optional)
```bash
CLI_TYPE=5                 # 0=Claude Code, 5=Crush (default), 6=ZeroClaw
PROMPT="Say hello world"   # the agent's first instruction
```

---

## Minimal working `.env` examples

**Persistent (HuggingFace + Anthropic):**
```bash
RPC_URL="https://rpc.ritualfoundation.org"
PRIVATE_KEY="0x..."
ANTHROPIC_API_KEY="sk-ant-..."
DA_PROVIDER="hf"
HF_TOKEN="hf_..."
HF_REPO_ID="your-username/my-agent-store"
AGENT_RPC_URL="https://rpc.ritualfoundation.org"
DEPOSIT_WEI=1000000000000000000
CHILD_FUND_WEI=1000000000000000000
CHILD_MIN_NATIVE_WEI=100000000000000000
MIN_RITUAL_WALLET_WEI=1000000000000000000
# EXECUTOR_TEE_ADDRESS=0x...
```

**Sovereign (HuggingFace + Anthropic):**
```bash
RPC_URL="https://rpc.ritualfoundation.org"
PRIVATE_KEY="0x..."
ANTHROPIC_API_KEY="sk-ant-..."
MODEL="claude-sonnet-4-5-20250929"
HF_TOKEN="hf_..."
HF_REPO_ID="your-username/my-agent-workspace"
# PROMPT="Say hello world"
# EXECUTOR_TEE_ADDRESS=0x...
```
