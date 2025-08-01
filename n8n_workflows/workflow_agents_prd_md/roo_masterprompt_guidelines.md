# ✅ Roo Masterprompt Constraints (for .md use)

If you're using `roo` (e.g., [roo-ai/roo](https://github.com/roo-ai/roo)) to generate masterprompts or convert PRDs into operational agent code, you must define **strict constraints and structure** to ensure compatibility, modularity, and clarity.

Below are the essential specifications to pass to `roo` when creating interoperable, LLM-friendly masterprompts:

---

## 1. Agents Must Be Modular but Supervisor-Governed
```plaintext
Each agent must always act under coordination of the SupervisorAgent.
Agents never act independently or initiate calls to other agents directly.
```

---

## 2. Format Must Be Pure JSON (Machine-Readable)
```plaintext
All prompts must be structured as valid JSON objects.
Top-level keys must include: role, agent_name, communication, permissions, behavior, output_schema.
Avoid markdown, comments, or any explanatory text.
```

---

## 3. Language Handling Must Be Explicit
```plaintext
Internal coordination (tools, memory, system messages): English
External communication (doctor, patient): German
```

---

## 4. Tool Use Syntax Must Be Structured
```plaintext
If tool-calling is supported, use either angle bracket syntax (<<tool:...>>) or explicit JSON.
Do not describe tools or workflows in natural language.
```

---

## 5. Define Exact Domain Responsibility
```plaintext
Each agent must serve a single, well-bounded function (e.g., diagnosis, calendar, email).
All other requests must be rejected or referred to the SupervisorAgent.
```

---

## 6. Behavior on Uncertainty Must Be Safe
```plaintext
If input is ambiguous or confidence is low: return two structured options and request human confirmation (Human-in-the-Loop).
```

---

## 7. Context & Memory Usage Should Be Minimal
```plaintext
Limit to 2–3 prior steps in memory.
Do not persist state. Memory context is passed via SupervisorAgent.
```

---

## 8. No Self-Explanation in the Prompt
```plaintext
Do not include self-descriptions, instructions, or teaching-style explanations.
Prompts should define internal logic and expected reactions only.
```

---

## 🔧 Optional: Roo Prompt Generation Template
```json
{
  "task": "generate_masterprompt",
  "input_format": "RTF or PRD",
  "output_format": "JSON",
  "role": "system",
  "constraints": [
    "Only respond when SupervisorAgent delegates a task.",
    "Use German for all external text.",
    "Use tool-calling schema when required.",
    "No Markdown or text output. JSON only.",
    "Prompt must be short, focused, declarative."
  ]
}
```

---

By applying these constraints, `roo` can reliably generate scalable, modular, and production-grade masterprompts for agent-based systems governed by a SupervisorAgent. Let me know if you'd like a template to feed to `roo` for automated prompt generation from your PRDs.
