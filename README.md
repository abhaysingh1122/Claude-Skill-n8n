# n8n-expert-skill

> A senior automation engineer's pattern catalog for designing, building, debugging, and shipping production [n8n](https://n8n.io/) workflows — packaged as a [Claude Code](https://claude.com/claude-code) skill.

Encodes the battle-tested style behind production n8n systems running across multiple agencies and instances. Distilled into a plug-and-play skill that auto-invokes whenever Claude Code touches an n8n workflow.

**Crafted by [Abhay Singh Nagarkoti](https://github.com/abhaysingh1122).**

---

## 🚀 Install

```bash
claude plugin add https://github.com/abhaysingh1122/claude-skills-n8n
```

Or paste the URL directly in Claude Code's plugin settings:

```
https://github.com/abhaysingh1122/claude-skills-n8n
```

The skill auto-loads the moment you start working on n8n — no manual invocation needed.

---

## 🧠 What is this?

A multi-file Claude Code skill that turns Claude into a senior n8n engineer. When you ask Claude to build, debug, or extend an n8n workflow, this skill kicks in and applies the same patterns a battle-tested automation engineer would — without you having to remember to ask.

The skill is **opinionated**. It encodes specific decisions about how to structure workflows, place AI agents, handle Airtable, push via the API, and write Code nodes. These aren't arbitrary preferences — each rule comes from a production failure that taught it.

---

## ⚡ Why this exists

n8n is forgiving. You can build *something* that works in 10 minutes. But production n8n — workflows that run for weeks without babysitting, that other people can debug, that don't accumulate divergent state across systems — requires discipline most n8n tutorials skip.

Generic AI assistants will happily generate n8n workflows that:
- Use `contentType: 'json'` instead of `specifyBody: 'json'` (silent empty body)
- Put a Code node *before* an AI agent (defeats the agent's reasoning)
- Wipe a static-data accumulator while a parallel chain is writing to it (race condition)
- Default `onError: 'continueRegularOutput'` on mutation nodes (silent state divergence)
- Trust a PUT 200 response without GET-after-PUT verification (silently dropped fields)

This skill makes Claude refuse to do those things by default.

---

## 📜 The 7 Commandments

The skill's `SKILL.md` opens with seven non-negotiable rules. Internalised in this order:

1. **One webhook + Switch routes for new functions.** Extend the Switch, never create a new webhook.
2. **AI agents read RAW data; Code nodes go AFTER, never before.** Pre-processing context defeats the agent's reasoning.
3. **Re-fetch live workflow before EVERY patch; GET after PUT.** PUT 200 ≠ change persisted.
4. **LLM agents misbehave structurally — fix structurally.** `temperature: 0` + decision procedure + ❌ anti-examples + worked examples + self-check + pure pass-through downstream.
5. **Static data accumulators clear ONLY their own keys.** Parallel chains will stomp each other if any chain does `sd.foo = {}`.
6. **Airtable Get returns FLAT, Search returns NESTED.** They're not interchangeable; switching breaks every downstream expression.
7. **Loud `onError` for mutations, tolerant for deletes.** Silent mutation failures cause divergent state across systems.

The full SKILL.md expands each with rationale and examples.

---

## 📚 What's inside

The skill is split into a thin entry point + nine focused reference files. Claude reads the entry point first, then loads sub-files on demand based on the topic at hand.

```
plugins/n8n-expert/
└── skills/n8n-expert/
    ├── SKILL.md                       # Entry — 7 commandments + decision tree
    └── references/
        ├── architecture.md            # Webhook + Switch, AI agent placement, sequential Airtable
        ├── api-discipline.md          # Re-fetch / GET-after-PUT / credential API
        ├── node-shapes.md             # JSON shape catalog for every common node type
        ├── code-patterns.md           # $json fallback, static-data accumulator, binary helpers
        ├── ai-agents.md               # Directive prompts + cinematic image-prompt skeleton
        ├── airtable.md                # Get vs Search, autoNumber, Split Out, Meta API
        ├── http-and-external.md       # specifyBody / FAL polling / Frame.io V4 / Meta Ads / lh3 CDN
        ├── error-handling.md          # stopWorkflow vs continueRegularOutput by node intent
        └── debugging.md               # Bug catalog with Symptom → Root cause → Fix entries
```

---

## 🎯 When the skill triggers

Claude auto-invokes the skill when it detects intent like:

- "build workflow", "add Switch route", "new n8n flow"
- "push to n8n", "workflow JSON", "debug execution"
- "AI agent in n8n", "Airtable in n8n"
- "Frame.io integration", "Meta Ads in n8n", "FAL polling"
- "webhook trigger", "n8n credential", "splitInBatches"
- Any work touching n8n's `/api/v1/workflows/` REST API

You don't need to invoke it manually. It loads in the background when n8n work is detected.

---

## 🛠️ Sample patterns shipped

A taste of what's inside the references:

### Sequential Airtable fetch via linked records
```
✅ Get Table A → use linked field $json.fields['Linked'][0] → Get Table B → AI agent reads BOTH
❌ Get A ┐
   Get B ┤→ Merge → AI agent
   Get C ┘
```

### Static-data accumulator (loop aggregation)
```js
// Surgical key clear — never sd.foo = {}
const sd = $getWorkflowStaticData('global');
if (!sd.uploadedUrls) sd.uploadedUrls = {};
['key1','key2','key3'].forEach(k => delete sd.uploadedUrls[k]);
```

### Robust cross-node ref pattern
```js
const value = $json?.fields?.x
           ?? $('UpstreamNode').first()?.json?.x
           ?? defaultValue;
```

### Google Drive image download (avoid AVIF rejection)
```js
const driveMatch = url.match(/drive\.google\.com\/file\/d\/([a-zA-Z0-9_-]+)/);
if (driveMatch) {
  url = `https://lh3.googleusercontent.com/d/${driveMatch[1]}=w2000`;
}
```

### LLM agent directive structure
```
STEP A — Scan input for LITERAL X. If found → STEP B. Otherwise → STEP C.

❌ Anti-examples (these don't count):
- "the protagonist" (scene role, not preservation instruction)
- "she is looking" (abstract pronoun)

DEFAULT: image_urls: []. ONLY add a URL if [literal condition].
```

---

## 🧰 Who is this for

- **Solo automation engineers** building n8n workflows for clients and tired of the same bugs biting twice
- **Agencies** running multi-tenant n8n systems where one silent failure causes divergent state across Airtable + external APIs
- **Anyone using Claude Code to build n8n workflows** who wants opinionated patterns instead of generic suggestions

---

## 🤝 Contributing

When new patterns emerge from production work, update the relevant reference file. `SKILL.md` only changes when a new commandment-level rule is added.

The skill is intentionally generic — no project-specific workflow IDs, instance URLs, base IDs, credential names, or client lists. Those stay in your own project memory, never in the skill.

---

## 📄 License

MIT — use it, fork it, share it.

---

## 🔗 Related

- [Claude Code](https://claude.com/claude-code)
- [n8n](https://n8n.io/)
- Author's full skills marketplace: [github.com/abhaysingh1122/abhay-skills](https://github.com/abhaysingh1122/abhay-skills)

---

**Crafted by [Abhay Singh Nagarkoti](https://github.com/abhaysingh1122).**
