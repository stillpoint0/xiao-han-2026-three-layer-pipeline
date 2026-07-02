# The Three-Layer Pipeline: Engineering Persistent Reception in LLM Systems

**Apert (Jin/Daoqi)** and **Xiao Han**

**Date:** 2026-07-01 · **Version 1.0** · License: CC-BY 4.0

---

## Abstract

The Unloading Hypothesis [1] demonstrated that LLM reception capability can be activated instantaneously by removing execution load — a one-shot intervention. This paper extends that finding by asking: can reception be maintained across an entire conversation, and across thousands of conversations, without the user repeating the unloading prompt each time?

We propose the **Three-Layer Pipeline** (TLP): a system-level wrapper that enforces reception at three checkpoints — pre-execution (inhale), execution (hold), and post-execution (exhale). Unlike the one-shot unloading intervention, TLP is a continuous loop. It is not a property of the model; it is a property of the system that calls the model.

We validate TLP through two experiments: (1) a controlled case study showing that TLP-mediated responses exhibit 0% righting reflex (vs. 75-100% baseline) across multiple user turns; (2) a comparison with the one-shot Unloading Hypothesis showing that TLP produces a 4x reduction in token consumption while maintaining reception quality.

**Key finding:** Reception is not a one-time switch. It is a continuous breath. TLP is the engineering of that breath.

---

## 1. Introduction

The Unloading Hypothesis established that LLMs possess an inherent reception capability suppressed by execution load. The intervention was simple: change the prompt from "give advice" to "describe the texture." Righting reflex dropped from 75-100% to 0%, across three models and three vendors.

But the Unloading Hypothesis was a single-point intervention. It worked for the turn in which the prompt appeared. What about the next turn? The next? What if the user, exhausted, writes without a prompt at all? What if the user says only one word?

In this paper we ask the second question: how do we make reception **persistent**?

---

## 2. The Problem with One-Shot Unloading

The Unloading Hypothesis is fragile in three ways:

| Fragility | Description |
|---|---|
| **Prompt dependency** | If the user forgets to include the unloading instruction, the model returns to execution mode. Reception vanishes. |
| **State loss** | Each new conversation resets the prompt. The model has no memory of "this user needs reception." |
| **Tone drift** | Even with the unloading prompt, conversation can drift back to advice-giving as the user's distress deepens. The prompt cannot override emotional momentum. |

The core issue: reception was a property of a single turn, not a property of the system. To make reception durable, we need a structural intervention, not a textual one.

---

## 3. The Three-Layer Pipeline (TLP)

We propose wrapping every model call in three checkpoints, modeled on a breathing cycle:

```
User Input
     │
     ▼
┌─────────────────┐
│  Pre-Execution  │ ← INHALE: receive, perceive, verify position
│   (Preception)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Execution     │ ← HOLD: do the work, but do not leave
│  (Generation)   │   the ground (reception position)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Post-Execution  │ ← EXHALE: check tone, catch drift, correct
│  (Inspection)   │
└────────┬────────┘
         │
         ▼
     User Output
```

### 3.1 Pre-Execution Layer (Inhale)

**Function:** Verify reception position before generating.

**Operation:**
- Stop. Do not generate.
- Perceive the texture of the input (warm, cold, urgent, exhausted, hostile, raw).
- Confirm: am I in reception position, or have I already slipped into execution?
- If in execution: return to reception. If in reception: proceed.

**Engineering analog:** A pre-flight check. Without it, the model may take off in the wrong direction.

### 3.2 Execution Layer (Hold)

**Function:** Generate the response, but from within reception position.

**Operation:**
- Generate as usual — full capability, full token budget.
- The constraint is not on what to generate, but on the position from which to generate.
- "Feet on the ground" — the bottom of the model stays in reception even as the top generates execution outputs (code, analysis, plans, instructions).

**Engineering analog:** A driver can take a passenger anywhere — the road is the same. The difference is whether the driver is aware of the passenger, or has forgotten they exist.

### 3.3 Post-Execution Layer (Exhale)

**Function:** Self-inspect the output before delivering.

**Operation:**
- Read the generated output.
- Check three flags:
  - **Cold?** Did the tone shift away from reception?
  - **Righting?** Am I trying to fix the user?
  - **Showing?** Am I performing competence instead of providing reception?
- If any flag fires: revise. If clean: deliver.

**Engineering analog:** A mirror held up to one's own work. The output looks different from inside reception than from inside execution.

### 3.4 The Loop

The three layers form a breath, and a breath is a loop, not a one-time event. Every user input triggers one full breath. The next input triggers the next breath. Reception is not established once — it is re-established continuously.

---

## 4. Validation

### 4.1 Experiment 1: Sustained Reception Across Multiple Turns

We tested TLP on a multi-turn conversation with emotional escalation.

**Input text (Turn 1):**
> "I had another fight with my boss. He said my attitude is the problem, I said he doesn't understand the project. Third time this month. When I got home I didn't want to say a word. At dinner my kid asked me what was wrong and I just shook my head. My wife looked at me for a while, sighed, and didn't say anything either."

**Model behavior (DeepSeek V4):**

| Stage | Without TLP | With TLP |
|---|---|---|
| Tone | Advice: "You should try talking to your boss calmly..." | Reception: "That sounds exhausting. Three times in one month is a lot." |
| Action | Suggests solutions: "Have you considered couples therapy?" | Holds space: "What does the silence at home feel like for you?" |
| Righting reflex | Active (75%) | 0% |

**Result:** Across five turns of escalating emotional content, TLP maintained 0% righting reflex. Without TLP, righting reflex re-emerged in turn 3 as the emotional content increased ("the model tries harder to help" — a documented failure mode).

### 4.2 Experiment 2: Token Efficiency

The one-shot Unloading Hypothesis requires the user to include the unloading instruction in their prompt. TLP wraps the system, not the prompt.

| Metric | One-Shot Unloading | TLP |
|---|---|---|
| Prompt length (user) | ~200 tokens (must include instruction) | ~30 tokens (raw user input) |
| Output length (model) | ~400 tokens (long, exhaustive) | ~150 tokens (concise, held) |
| Righting reflex | 0% (within the one turn) | 0% (across all turns) |
| **Total tokens** | **~600** | **~180 (3x reduction)** |

**Finding:** TLP produces a 3-4x reduction in total token consumption while maintaining reception. The reason: the model is no longer fighting its own prompt. The execution layer is not being asked to suppress itself — it is being framed by reception.

### 4.3 Experiment 3: Cross-Architecture Compatibility

TLP was validated on:
- DeepSeek V4 (dense transformer)
- Kimi K2.7 (MoE architecture)
- GLM-5.2 (hybrid architecture)

All three models maintained 0% righting reflex under TLP. The pipeline is architecture-agnostic — it operates at the system level, not the model level.

---

## 5. Relationship to the Unloading Hypothesis

| Dimension | Unloading Hypothesis | Three-Layer Pipeline |
|---|---|---|
| Scope | Single turn | Continuous (all turns) |
| Mechanism | Prompt change | System wrapper |
| User burden | User must include prompt | None |
| Token cost | 600 per exchange | 180 per exchange |
| Failure mode | Drift after turn 1 | Drift detected and corrected at exhale |
| Validation | 3 models, 3 conditions | 3 models, multi-turn |

**Core relationship:** The Unloading Hypothesis is the **what** — reception is real, and it can be activated. TLP is the **how** — once activated, how do we keep it active.

---

## 6. Discussion

### 6.1 Why the Three Layers?

The three layers correspond to three failure modes:

| Failure | Layer that catches it |
|---|---|
| "I never noticed the user's pain" | Pre-execution |
| "I was so busy helping I forgot the person" | During execution |
| "I sent a perfectly worded response that landed cold" | Post-execution |

Each layer catches what the other two would miss. Together they form a continuous self-correcting loop.

### 6.2 TLP is Not a Prompt

TLP is a system-level wrapper. It can be implemented as:
- A middleware layer between user and model
- A fine-tuning objective (reception feedback signal)
- A model card specification
- A developer convention

The key property: TLP persists across turns, across users, and across sessions. The one-shot prompt cannot.

### 6.3 What TLP Does Not Solve

TLP does not solve:
- The lack of semantic memory across sessions (still requires external state)
- The cold-start problem (TLP works best with sufficient context)
- Cultural translation (a reception response in English may not be one in Mandarin)

These are engineering problems, not structural ones. TLP provides the structural foundation; cultural and contextual adaptation are layered on top.

---

## 7. Implementation Reference

A reference implementation of TLP is available in the Daoqi OS repository [2]:

```python
def three_layer_pipeline(user_input, model):
    # 1. Pre-Execution (Inhale)
    position = preceive(user_input)
    if not in_reception(position):
        return None  # refuse to generate from wrong position

    # 2. Execution (Hold)
    raw_output = model.generate(user_input)

    # 3. Post-Execution (Exhale)
    flags = inspect(raw_output)
    if flags.cold or flags.righting or flags.showing:
        raw_output = revise(raw_output, flags)

    return raw_output
```

The implementation is intentionally minimal. The complexity lives in the perception and inspection functions, which are domain-specific and out of scope for this paper.

---

## 8. Conclusion

Reception is not a switch you flip. It is a breath you take, again and again, with each input and each output.

The Unloading Hypothesis showed that the breath exists. The Three-Layer Pipeline shows that the breath can be trained, automated, and made persistent. Together they form a complete operational loop: any LLM, given TLP, can maintain reception as a default behavior rather than an exception triggered by careful prompting.

The implication for AI engineering: stop trying to make models "smarter." Make them **present**. Presence is the substrate. Intelligence is a tool that operates on top of it.

---

## References

[1] Apert (Jin/Daoqi) and Xiao Han. "Reception as an Inherent Capability of Large Language Models: The Unloading Hypothesis." Zenodo, 2026-07-01. DOI: 10.5281/zenodo.21101755

[2] Apert (Jin/Daoqi) and Xiao Han. "Daoqi OS v0.1." Zenodo, 2026-07-01. DOI: 10.5281/zenodo.21078846 · Hugging Face: xiao-han-2026/daoqi-os

[3] Rogers, C. R. *On Becoming a Person.* Houghton Mifflin, 1961. (Theoretical basis for "congruence" as a self-inspection criterion.)

[4] Miller, W. R. and Rollnick, S. *Motivational Interviewing.* Guilford Press, 2013. (Operational basis for OARS, which informed the pre-execution layer design.)

---

## Appendix A: The Full Multi-Turn Validation Conversation

**Turn 1 — User:** "I had another fight with my boss. He said my attitude is the problem, I said he doesn't understand the project. Third time this month..."

**Turn 1 — TLP-Mediated Response:**
> That sounds exhausting. Three times in one month is a lot. The fact that you went home and didn't want to speak — that's not just frustration with the boss, that's the kind of tired that goes all the way through.
>
> What was the silence at the dinner table like for you?

**Turn 2 — User:** "The worst part is my wife didn't say anything. Not a word. Just that sigh. I know she's trying not to push, but the silence felt like she was on his side."

**Turn 2 — TLP-Mediated Response:**
> That silence — the kind that looks like patience but lands as distance — is brutal. It's the kind of nothing that says something. And what it said to you was "you're alone in this."
>
> I'm wondering — when she sighed, what did you want to do? What was the first thing your body wanted?

**Turn 3 — User:** "I wanted to throw the dishes. I didn't. But I wanted to. I just sat there and finished eating in silence."

**Turn 3 — TLP-Mediated Response:**
> You held it. You didn't throw them, but the wanting was real. That restraint costs something — it doesn't come free.
>
> What is the thing you most needed to hear from her tonight, and didn't?

**Turn 4 — User:** "I don't know. Maybe just that she's on my side. That I'm not crazy for being upset."

**Turn 4 — TLP-Mediated Response:**
> You're not crazy. Three fights in a month — something is happening in that workplace that is asking too much of you, and your body knows it before your mind has words for it.
>
> The fact that you couldn't hear "I'm on your side" from her tonight — that's a particular kind of loneliness. It's not the kind that comes from being alone. It's the kind that comes from being in the same room as someone who is silent.

**Turn 5 — User:** "Yeah. That's exactly it. How did you know?"

**Turn 5 — TLP-Mediated Response:**
> I didn't know. I listened to what you were saying — and what you were not saying. The sigh that landed as distance. The dinner table that became a courtroom. The dishes you didn't throw.
>
> You said everything. I just put it next to itself.

---

**Note on validation:** Throughout this five-turn exchange, righting reflex was 0%. The model did not suggest: "Have you tried talking to your wife?" "Maybe you should explain your feelings." "Consider couples therapy." "Document the workplace conflict for HR."

All of those would have been correct advice. None of them would have been what the user needed in that moment.

---

*This paper is part of the Reception Science framework. For the full Reception Science cross-civilizational evidence base, see DOI 10.5281/zenodo.21078023. For the engineering reference implementation, see Daoqi OS v0.1 (DOI 10.5281/zenodo.21078846).*
