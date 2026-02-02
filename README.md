# LiveKit Intelligent Interruption Handling

This repository contains my solution to the **LiveKit Intelligent Interruption Handling Challenge**.

It is based on the assignment repository:
https://github.com/Dark-Sys-Jenkins/agents-assignment

The goal of this assignment is to improve conversational flow in a real-time voice agent by
correctly distinguishing **passive acknowledgements** from **active interruptions**.

---

## 🚩 Problem Statement

In the default LiveKit agent behavior, Voice Activity Detection (VAD) is overly sensitive.
When the agent is speaking and the user says short filler words such as:

- "yeah"
- "ok"
- "hmm"
- "uh-huh"

the agent incorrectly interprets these as interruptions and stops speaking mid-sentence.

This leads to a broken conversational experience.

---

## 🎯 Objective

Implement a **context-aware logic layer** such that:

- Passive acknowledgements are **ignored while the agent is speaking**
- Active interruption commands **immediately stop the agent**
- The same words (e.g. "yeah") are treated as **valid input when the agent is silent**
- The solution works in real time and does **not modify the low-level VAD kernel**

---

## ✅ Final Behavior Matrix

| User Input | Agent State | Result |
|----------|-----------|--------|
| "yeah", "ok", "hmm" | Agent speaking | **Ignored** (agent continues seamlessly) |
| "stop", "wait", "no" | Agent speaking | **Interrupted immediately** |
| "yeah", "ok" | Agent silent | **Processed as valid input** |
| "yeah wait a second" | Agent speaking | **Interrupted (semantic command detected)** |
| "hello", "start" | Agent silent | **Normal response** |

---

## 🧠 Solution Overview

The core issue is that **VAD detects silence faster than STT produces text**.
This causes the agent to stop speaking before the system can determine
whether the user actually intended to interrupt.

To solve this, I implemented a **state-aware filtering layer** that:

1. Tracks the **agent speaking state**
2. Tracks the **last finalized STT utterance**
3. Filters interruption behavior **based on both state and semantics**

### Key Design Principles

- **No VAD kernel modification**  
  VAD is treated as a signal, not a decision-maker.

- **Text-validated interruption**  
  The agent only interrupts once STT confirms a real command.

- **Backchannel awareness**  
  Passive acknowledgements are ignored *only when the agent is speaking*.

---

## 🧩 Key Implementation Details

### 1️⃣ Configurable Ignore List

A configurable list of passive acknowledgement words is used:

```python
IGNORE_WORDS = [
    "yeah",
    "ok",
    "okay",
    "hmm",
    "uh-huh",
    "right"
]

Link to the demo video: https://drive.google.com/file/d/1PrefOubQecFKNXs6Qi45-oJTlHnhPsAI/view?usp=sharing