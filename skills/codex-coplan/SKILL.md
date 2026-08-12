---
name: codex-coplan
description: Bring in an external model (Codex) to co-develop a plan — both models draft independently, align on disagreements, and merge into one plan. Load this only when the user explicitly names Codex (e.g. "let's discuss this with Codex," "what does Codex think about this approach") — don't auto-trigger just because a plan has disagreements or could use a fresh angle; think it through yourself first.
---

# Co-developing a plan with an external model

## Process

1. **Prep the brief** — Codex has no access to your conversation history, so assemble what it needs for this scenario: background and goals, hard constraints, judgment criteria, **absolute paths** to relevant files and directories (name the entry points and let it read them itself), what's already been tried and why it didn't work, and environmental facts (schema/API contracts/config values, scale, permissions, timeline — whatever applies). Look up what's missing yourself; only ask the user if you can't find it.

2. **Round one: independent drafts** — write your own version, and dispatch Codex to write its own, **with zero cross-reference on this round only**: if both versions get pulled toward each other, there's no independent second opinion left to compare against. Give it nothing but the brief from step 1. **For anything you already tried and know failed, state the reason** (so it doesn't repeat the mistake); **for anything you're just not satisfied with but haven't actually verified, give it only the label, no reasoning** (a stated reason becomes an anchor). Assign it a plausible professional persona different from your own default framing (skip celebrities).

3. **Classify the differences** (you do this yourself, don't dispatch it): **agreements** (both sides have it — goes straight into the plan), **one-sided points** (only one side has it), **conflicts** (same question, opposite answers).

4. **Follow up** — **starting this round, share your full draft with it**, and keep the thread open from here on; the discussion needs to run on the same shared material going forward. Only follow up on one-sided points and conflicts, with specific questions — never "what do you think of the other version." Use `--resume` to continue the same thread, or it won't remember its own positions:
   - One-sided → "You didn't account for \<approach>. Is it inapplicable, or did you miss it? If inapplicable, which constraint rules it out?"
   - Conflict → "You argued A, the other draft argued B — the disagreement comes down to an assumption about \<something>. What's your assumption, and under what conditions does it break?"

5. **Fold the answers into your draft, then re-classify** — go back to step 3. **Keep following up as long as new one-sided points or conflicts show up**; stop only once a round adds nothing new. Don't let the two models talk each other into agreement — more rounds just means more convergence. **Soft cap: 3 rounds.** If round 3 still surfaces something new, stop waiting for convergence — go straight to step 6 and resolve by checking the facts, and settle whatever's left using your own judgment criteria.

6. **Resolve disagreements by checking premises** — most disagreements trace back to different assumptions about facts. Check what you can check yourself (does this field exist, what's the actual scale, is this permission granted, what's the real timeline) — don't just take either model's account of it.

7. **Make the call, deliver one plan** — whatever you can't verify, settle with your own judgment criteria. Only offer multiple solutions when the conditions for applying them are mutually exclusive, and label each with when it applies. **Never write "Codex thinks X, I think Y" — deliver the plan that's already absorbed both inputs.**
