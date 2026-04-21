---
title: "Twitter/X thread — The Born-rule classifier"
byline: "@sam_sammane / @qgidev"
venue: "Twitter / X"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
---

# Thread (8 tweets)

---

**1/**
You are softmaxing cosine similarities and calling the result a probability.

The result is not a probability. It's a ranking, recalibrated by a free parameter you guessed.

There's a better primitive. It's 100 years old. Born rule. 🧵

---

**2/**
Replace this:

```python
p = softmax(cosine(doc, class_embeds) / temperature)
```

With this:

```python
amp = class_embeds @ doc
p = (amp ** 2) / (amp ** 2).sum()
```

Three lines. No temperature. No hyperparameter. No training.

---

**3/**
The Born rule: $P(c | \psi) = |\langle c | \psi \rangle|^2$.

Max Born, 1926. Nobel Prize 1954. Specifies how inner products in a Hilbert space become probabilities.

Embedding spaces ARE Hilbert spaces. The rule transfers without modification.

Not "quantum inspired". Actually quantum probability.

---

**4/**
Empirical on three public benchmarks, three public embeddings:

• Cosine-softmax ECE: 0.15 (70-85% calibrated)
• Born-rule ECE: 0.04 (~90-96% calibrated)
• Top-1 accuracy: unchanged (±0.3%)

Drop-in replacement. Better calibration for free.

---

**5/**
On Q-Prime, where class axes are orthogonal by training, Born-rule classifier hits ECE < 0.01.

That's the regime where a downstream threshold ACTUALLY means what it says.

"P(fact) > 0.9 → promote to memory" — the 0.9 is a reliable 90%, not a nominal 90%.

---

**6/**
When does calibration matter? Whenever you gate:

• Content moderation (flag if P(toxic) > 0.8)
• Auto-reply (respond if P(canned-applies) > 0.95)
• Agent memory admission (promote if P(fact) > 0.9)
• Compliance triage (escalate if P(risky) > 0.7)

Cosine-softmax thresholds are brittle. Born-rule thresholds hold.

---

**7/**
Caveat: Born rule normalises over a complete basis.

If your class prototypes don't cover the relevant semantic space, you need the open-set extension (residual norm for "not in any class"). Covered in §4 of the preprint.

---

**8/**
Preprint: *The Born-Rule Classifier* [QGI-TR-2026-04]

~10 pages. Self-contained. Free code (3 lines). Try it on your pipeline this week.

One of 6 preprints released today from QGI.

research@qgi.dev for evaluation conversations.

/fin
