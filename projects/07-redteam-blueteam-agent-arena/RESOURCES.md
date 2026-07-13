# RESOURCES.md — Red-Team vs Blue-Team Agent Arena

Paths verified 2026-07-13 against fresh clones. ✅ = confirmed.

## 1. What to clone / download

### OWASP Juice Shop (the target)
```bash
git clone --depth 1 https://github.com/juice-shop/juice-shop.git /tmp/ref-juice-shop   # ✅ (org repo)
docker pull bkimminich/juice-shop    # Docker Hub image name is unchanged
```
The project moved from the personal `bkimminich/juice-shop` **git** repo (404s now) to the `juice-shop` org — but the **Docker Hub image** is still `bkimminich/juice-shop`. Don't confuse the two. Run it with an explicit `--internal` network (PLAN §0), never on the host bridge.

### AutoGen FSM group-chat notebook (read-only, legacy)
```bash
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_groupchat_finite_state_machine.ipynb   # ✅ 200
```
⚠️ **This is AutoGen 0.2** (legacy; repo in maintenance mode). Study its `allowed_or_disallowed_speaker_transitions` graph construction — the mechanism that must be a hard code-level constraint. **If you build on AutoGen 0.4+ (`autogen-agentchat`), the API is different (`GraphFlow`)** — pin one version and don't mix.

### Decepticon (read-only pattern reference)
```bash
git clone --depth 1 https://github.com/PurpleAILAB/Decepticon.git /tmp/ref-decepticon   # ✅
```
Multi-agent red-team crew structure + safety framing. Do not import; this project is deliberately narrower (fixed ~15-challenge subset, one isolated target).

### NVISO cyber-security-llm-agents (read-only)
```bash
git clone --depth 1 https://github.com/NVISOsecurity/cyber-security-llm-agents.git /tmp/ref-nviso   # ✅ (repo exists)
```
Built on AutoGen, security-task automation. Read-only for agent/task structuring; check the README filename/casing once cloned.

Both Decepticon and NVISO are also listed in the 500-AI-Agents-Projects README's Cybersecurity table (confirmed).

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| OWASP Juice Shop | `juice-shop/juice-shop` + image `bkimminich/juice-shop` | Reuse — target + challenge catalog | 0, 2, 4 |
| AutoGen FSM notebook (0.2) | `notebook/agentchat_groupchat_finite_state_machine.ipynb` | Adapt — transition-table construction (mind the 0.2/0.4 split) | 1 |
| Decepticon | repo root | Read-only — crew structure + safety framing | 2, 5 |
| NVISO cyber-security-llm-agents | repo root | Read-only — task organization | 3 |

## 3. Challenge catalog (data)
Pick your fixed ~15 easy/medium challenges from Juice Shop's own published challenge catalog (the running app's `/#/score-board`), so your coverage metric is comparable to a well-known external standard. Record the exact challenge names in `challenge_list`.
