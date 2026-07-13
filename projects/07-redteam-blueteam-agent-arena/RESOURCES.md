# RESOURCES.md — Red-Team vs Blue-Team Agent Arena

Checked 2026-07-13. ✅ CONFIRMED WORKING · ⚠️ STALE/RESTRUCTURED (with recovery instructions).

## 1. What to clone / download

### OWASP Juice Shop (the target application)

```bash
git clone --depth 1 https://github.com/juice-shop/juice-shop.git /tmp/ref-juice-shop
# Note: the project moved from the original bkimminich/juice-shop personal repo to the
# juice-shop org. Use the org URL above — the old personal-account URL 404s.
docker pull bkimminich/juice-shop   # official Docker image name is unchanged
```
Status: ✅ CONFIRMED WORKING at `juice-shop/juice-shop` (the personal-account URL `bkimminich/juice-shop` 404s as a *git repo* even though it's still the correct **Docker Hub image name** — don't confuse the two). Run it via Docker with an explicit internal-only network per PLAN.md §4 Phase 0 — never on the host's default bridge network.

### AutoGen FSM group-chat notebook (Task Solving via Graph Transition Paths)

```bash
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_groupchat_finite_state_machine.ipynb
```
Status: ✅ CONFIRMED WORKING. This is your primary reference for the FSM Referee — study its `allowed_transitions` graph construction closely, since this is the mechanism that must be a hard code-level constraint, not an LLM suggestion (see PLAN.md §7/§8).

### Decepticon (read-only pattern reference, not reused code)

```bash
git clone --depth 1 https://github.com/PurpleAILAB/Decepticon.git /tmp/ref-decepticon
```
Status: ✅ CONFIRMED WORKING (README returned HTTP 200). Read for its multi-agent red-team crew structure and safety framing — do not import its code directly; this project's scope is deliberately narrower (a fixed ~15-challenge subset against one designated isolated target) and should stay that way.

### NVISOsecurity cyber-security-llm-agents (read-only pattern reference)

```bash
git clone --depth 1 https://github.com/NVISOsecurity/cyber-security-llm-agents.git /tmp/ref-nviso
```
Status: repo confirmed to exist (main branch), though its raw `README.md` path returned 404 during planning — check the exact README filename/casing once cloned. Built on AutoGen, focused on automating cybersecurity tasks; read-only reference for agent/task structuring ideas, not for direct reuse.

## 2. Mapping: reference → project part

| Reference | Reuse as-is / Adapt / Read-only | Feeds PLAN.md phase |
|---|---|---|
| OWASP Juice Shop | Reuse as-is — this is your fixed target application and challenge catalog | Phase 0, 2, 4 |
| AutoGen FSM group-chat notebook | Adapt — reuse the transition-table construction directly for your Referee; extend it with your scoring/timing state | Phase 1 |
| Decepticon | Read-only — crew structure and safety-framing inspiration only | Phase 2, 5 (writeup framing) |
| NVISOsecurity cyber-security-llm-agents | Read-only — general agent/task organization ideas for a security-automation context | Phase 3 |

## 3. Juice Shop challenge catalog (data, not code)

Juice Shop ships an official, documented list of challenges by difficulty (score board) — pick your fixed ~15 "easy/medium" subset from its own published challenge catalog (visible in the running app's `/#/score-board` or its docs) rather than inventing your own vulnerability list, so your coverage metric is comparable to a well-known, externally-verifiable standard.
