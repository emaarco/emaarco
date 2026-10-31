AI is rapidly making routine coding tasks more interchangeable, and that trend is accelerating. But in niche domains like process automation with BPMN and process engines like Zeebe, raw prompts still produce inconsistent, often frustrating results. This happens mainly because the training data is sparse, the domain is specialized, and the variance between engines is high.

Skills could be the answer. Instead of re-explaining your domain, conventions, and edge cases every single time, you encode that knowledge once—and the model picks it up automatically. I’ve tested this approach in real projects, and I’m convinced it works. 

Thus, this post aims to share my learnings with you. It explains what skills are, how to apply them to BPMN process orchestration, and what benefits they offer. It also covers which guardrails to apply to keep quality high while moving faster. Finally, it shares the set of skills I've built and tested in real projects.

## 🚀 Is Coding Becoming Interchangeable?

Lately I’ve been reading a lot about what AI is doing to software development. And even though I’m not one of those thinking that developers will soon be completely replaced — I find some of what’s happening genuinely surprising.

Just days ago, [Spotify](https://techcrunch.com/2026/02/12/spotify-says-its-best-developers-havent-written-a-line-of-code-since-december-thanks-to-ai/) reportedly said their best developers haven't written a single line of code since December—because they're using AI agents to handle the implementation. And in a recent Pragmatic Engineer podcast episode, [the creator of Clawd](https://newsletter.pragmaticengineer.com/p/the-creator-of-clawd-i-ship-code) mentioned that he ships code he doesn’t even read anymore. Just two examples from many. Edge cases, perhaps, but telling ones. I’m not claiming this is universal. But they hint at real potential, especially in well-represented domains.

And that potential is grounded in something concrete: languages like Java, Python, and JavaScript. Constructs like HTML and CSS. Components like REST endpoints and database queries. The kind of code that’s been publicly available for years—sometimes decades. Embedded in source files across countless websites, shared through open source repositories, discussed in forums like [Stack Overflow](https://stackoverflow.com/), and documented in publications and tutorials. LLMs have absorbed it all—the whole knowledge of the web.

For these well-represented domains, AI agents are remarkably capable. With the right prompt, they can combine building blocks into entire services or products—in a fraction of the time it would have taken before. [OpenClaw](https://openclaw.ai/) is a good example: a real product built by essentially one developer, with AI handling most of the implementation.

Without judging this specific product—I’ve never looked at its code—the output isn’t always what an experienced developer would call beautiful. It’s rarely perfectly consistent with an existing codebase’s conventions, and it often misses edge cases. But it works. It’s a significant time-saver, and a good starting point—either for manual continuation or for the next prompt in an iterative loop. And at a smaller scope, with a few rounds of review and feedback, it becomes genuinely clean and production-ready.

Thus, the productivity gains are real: less time on boilerplate, more headspace for what actually matters. That’s genuinely valuable. But there’s a catch, and it becomes obvious the moment you step into a niche domain.

## 🔩 Why Process Orchestration with BPMN Is a Different Story

Working with BPMN engines like Zeebe to automate processes isn’t a mainstream programming domain. It’s a domain within the orchestration-space that is quite specialized, with comparatively little public training data. And that gets visible quite fast when you try to use AI.

When you ask an LLM for instance to generate a job worker, or to configure a service-task in a BPMN file, you’ll often get something that *looks* right but doesn’t work. And the reason for that is simple: The domain is full of concepts that don’t exist elsewhere: job workers, message correlation, variable scoping, incident handling, BPMN file schemas. And to make things harder, these can also vary between engines—even though BPMN itself is a shared standard of [OMG](https://www.omg.org/bpmn/).

Thus, when working with AI you must iterate. You add sample code. You paste documentation into the context. And after a few rounds, or manual resolving, you get something that works. To be fair: That iterative approach is already valuable—but there’s clear room for improvement.

### 🔄 First Steps in the Right Direction

Several tools and patterns have already emerged to narrow this gap.

[Agent files](https://agentskills.io/home) are one approach. They give the model persistent context about your project—like engine-specific conventions. However, their effectiveness drops when the file grows long and unfocused. Then, mixed concerns cut both ways: on one hand, Zeebe-specific context gets drowned out by unrelated project concerns. And on the other hand, engine noise pollutes tasks that have nothing to do with the engine.

Another approach is the Model Context Protocol—short [MCP](https://modelcontextprotocol.io/docs/getting-started/intro). It connects AI agents to external tools and data sources. Tools like [Context7](https://context7.com/) have emerged from this protocol. Through tool calls, they pull current library documentation directly into the model’s context—like the latest Camunda SDK docs. Having these docs available greatly reduces the chance the model generates incorrect or outdated API calls.

I tried both. Both helped. But neither felt sufficient. The context was always one conversation away from being lost, and there was no consistent way to encode engine-specific knowledge that the model could reliably access across sessions.

Until now—because that’s exactly the gap "skills" are designed to fill.

## 🧠 Skills: Reusable Knowledge

A skill is a folder of instructions — packaged once, reused every time. It teaches an AI agent how to handle a specific task or workflow by encoding domain knowledge, conventions, and edge cases explicitly. This way, the model loads it when relevant instead of relying on re-prompting each session.

Skills are built on an [open standard](https://agentskills.io/home). This standard was originally developed by [Anthropic](https://www.anthropic.com/) and released openly — much like MCP. That means the concept isn't tied to a single tool or vendor: it's a shared foundation that's growing across the ecosystem.

### 👩🏽‍💻 How to Build Skills

A skill consists of up to three parts. The `SKILL.md` file is the only required component. It contains a [frontmatter](https://code.claude.com/docs/en/skills#frontmatter-reference) section with metadata—like the skill's `name`, `description`, and `allowed-tools` (tools the agent can use without asking during execution). Other fields are available as well—like `argument-hint`, `model`, and `context`—but those three are the most commonly needed.

Next to this metadata in the frontmatter the markdown also contains a body. It holds the actual instructions: how to structure the output, what patterns to follow, which edge cases to handle, and—critically—when to ask for clarification before proceeding.

All in all, such a skill could, describe exactly how Zeebe job workers should be implemented. In a simplified example this could look somehow like this:

```markdown
---
name: create-worker
argument-hint: "[<path-to-ProcessApi-or-BPMN-file>] [task-type-constant]"
allowed-tools: Read, Write, Glob
description: Create a new @JobWorker class for a BPMN service task. Accepts a
	ProcessApi file, a BPMN model, or a plain description.
---

## Key Rules
- Use @JobWorker annotations from "io.camunda.client"
- Use @Variable annotation to define input variables of the job
- Inject the use-case interface as the only constructor parameter
- ...

## Pattern

```
package ...

import io.camunda.client.annotation.JobWorker
import io.camunda.client.annotation.Variable

@Component
class AbortRegistrationWorker(
	private val useCase: AbortSubscriptionUseCase
) {

	private val log = KotlinLogging.logger {}

	@JobWorker(type = TaskTypes.NEWSLETTER_ABORT_REGISTRATION)
	fun handle(@Variable subscriptionId: UUID) {
		log.debug { "Received job to abort registration for subscriptionId: $subscriptionId" }
		useCase.abort(SubscriptionId(subscriptionId))
	}
}
```

## Instructions

1. Find the required serviceTask in the Process-API or the bpmn-file itself
2. If you can’t resolve the process or task, ask the user to provide one
3. Derive package name and class name for the worker and search for existing files
4. If a file exists, report this to the user and ask what to do
5. Otherwise create a new worker using the camunda-spring-boot-starter patterns and key rules listed above
6. If you are missing any information (which annotations to use or use cases to inject), ask the user for feedback
7. Report the result and remind the developer to run `test-worker`
```

Beyond that, there are two optional folders — not required for every skill.

`references/` holds supporting docs loaded on demand to keep the model's context lean: for a worker skill, this might include additional code examples of complete worker implementations or zeebe-specific variable handling patterns.

`scripts/` contains executable helpers for deterministic checks. For a worker skill, this could be a script that parses a BPMN model, filters out the relevant service task types, and extracts the process variables for a given task, eliminating any ambiguity before the model starts generating code. All in all, a skill therefore looks something like that:

```
my-skill/        # The parent folder
├── SKILL.md     # Required: The skill description with frontmatter & body
├── references/  # Optional: supporting docs loaded on demand
└── scripts/     # Optional: executable helpers for deterministic checks
```

If you want to know more about how to write effective skills, Anthropic's [Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) is a solid reference.

### 🕵🏽 How a Skill Gets Detected

When a skill is defined accordingly, agents can detect and use it automatically. At startup, only the `name` and `description` are pre-loaded into context. When you make a request, the model matches it against those descriptions to decide which skill to load. The full `SKILL.md` content is only pulled in once invoked, keeping context lean by default.

This makes the `description` field critical: it must clearly state both *what* the skill does and *when* to use it. The mechanics behind this are documented in detail in the [official skills documentation](https://code.claude.com/docs/en/skills), which covers discovery, loading behavior, and suggestions for writing effective skill descriptions.

### 🗂️ Where Skills Apply

The scope of skills is broader than you might expect. Two patterns stand out where they consistently add value.

The first is domains with sparse-training. Areas that are either new or niche enough that public examples are scarce: This includes BPMN and process-orchestration but also the Model context protocol named earlier. In these cases, skills supply directly what the model can't infer from training.

The second is highly individual output. Any LLM can create a REST controller. But creating one that consistently follows your team's specific structure, naming conventions, and error handling patterns? That requires adaptability neither training nor prompting can reliably deliver alone. The same applies to analyzing processes against a custom style guide—or any artifact that must conform to a specific shape every single time.

All of this is because LLM output is best-guess and non-deterministic by nature. The same prompt can yield different results across runs. Skills encode the rules explicitly, giving the model the best possible chance of producing consistent, structurally predictable output.

### 📦 How Skills Can Be Categorized

So far the focus has been on *when* skills help. But it's equally useful to think about *what* they actually do — because that shapes how you design them. Based on Anthropic's observations, use cases for skills tend to cluster into three categories:

- **Output generation** — creating consistently shaped artifacts: code, documentation, diagrams. The `create-worker` example above is a direct case: same class structure, annotation usage, and naming conventions on every run, regardless of who invokes it.
- **Workflow automation** — encoding repeatable, multi-step processes. A `review-process` skill, for example, could walk through a BPMN model systematically: checking compliance against a style guide, verifying that each service task has a corresponding worker, confirming that each message has an adapter wiring it to the engine, and checking that process tests exist for each relevant path — then producing a structured report of what's in place and what's missing.
- **MCP enhancement** — skills that incorporate external tool access as part of their workflow. Rather than hardcoding Zeebe API details into the `create-worker` skill, you could extend it: the skill first identifies what needs to be built, then uses an MCP to load the current Camunda SDK docs into context, and only then generates the code — always based on the latest API, without any manual copy-pasting of documentation.

These categories are useful for framing skills, but they shouldn't be treated as strict boundaries. In practice, many skills blend multiple categories — and as the ecosystem evolves, new patterns will likely emerge that don't fit neatly into any of them.

### 🌐 The Skill Ecosystem

The ecosystem around skills is very active & growing fast. By February 2026, the [Agent Skills standard](https://agentskills.io/home) is already supported by many agents like Claude Code, Gemini CLI, Cursor, GitHub, and more.

Companies are publishing their own skill collections — [Anthropic's skills repo](https://github.com/anthropics/skills) being a good example. Community-curated lists are growing alongside those. And dedicated marketplaces and directories are also starting to emerge. [LobeHub](https://lobehub.com/) and [findskill.ai](http://findskill.ai/) being early ones worth knowing about.

In the process automation space, though? Comparatively little skills yet exist. That's part of why I wrote this post — to fill the gap and share initial examples from my work.

### 🔬 What This Looks Like in Practice

While building and refining skills in a real Zeebe project, I collected the most useful ones and moved them into the [bpmn-to-code skills plugin](https://github.com/Miragon/bpmn-to-code/tree/main/bpmn-to-code-skills).

Since I'm using Claude Code, the skills live in the `.claude/skills/` folder. But because agent-skills it's an open standard, that's not limiting. You can use them as-is with any other agent. The current collection looks like this:

```
.claude/skills/
├── automate-process/
├── create-process-adapter/
├── create-worker/
├── review-process/
├── test-process-adapter/
├── test-process/
├── test-worker/
└── ...
```

It contains a mix of generation skills — like implementing workers, adapters, and controllers. But also some quality-assurance skills like reviewing processes. Feel free to copy the ones you need, fork the whole repo, adapt them to your setup, learn from them - and gain the benefits they can offer to you.

And if you run into gaps or discover patterns that work better, feel free to share them - no matter whether you’re working with Zeebe or a completely different engine. My goal is to create a collection that helps everyone in this space move faster and build with more confidence.

## 📋 My Personal Experience

I’ve been experimenting with skills for a couple of weeks now, and I’m genuinely convinced. It started with Zeebe and BPMN. But I quickly found myself writing skills for other domains too: creating and testing GraphQL or REST controllers, each tailored to the team’s specific structure and conventions.

The output became more consistent, the quality improved, and I spent less time fixing things the model got subtly wrong. More importantly: I started to trust the output — not blindly, but enough to review rather than rewrite. That’s a meaningful shift in how you work.

But the more interesting change was what that freed-up time made possible. I found myself doing things I simply hadn’t had time for before: auditing the state of our ADRs, checking whether they were actually being followed, and closing the gaps. I even wrote a skill for that as well — getting it done faster than it would have taken otherwise.

I think this effect can be even more impactful in projects where teams are mostly stuck keeping the product running — buried in tech debt, with little room for anything else. The time won could be exactly what creates space to finally get ahead of things, instead of just surviving them.

Personally, what also keeps standing out to me: skills make it much faster to experiment and iterate. Less friction between idea and working implementation means you can validate more, adjust earlier based on feedback, and spend more of your time on things that create real value for users — rather than just chasing tech debt or keeping the lights on.

A good example of this comes from a colleague of mine: He recently wanted to test MCP as a new interaction layer for a product used in the construction industry for planning inbound orders. The existing workflows in there - for example to contact a customer - was tedious: open the customer list, search, navigate to the detail page, copy the email address, switch to your mail client, write the message. With an MCP server wired to the product, an AI agent could handle all of it. Just ask it to find the customer’s email and draft the message. He built it from an existing REST API specification in a single session. Using a simple prompt and the [mcp-builder skill](https://github.com/anthropics/skills/tree/main/skills/mcp-builder). From idea to working prototype in one go.

But let’s be honest about the limits.

## 👨‍💻 Human in the Loop — Still Non-Negotiable

Skills are not a magic fix. The output quality still depends on the model and how well the skill as well as your prompt is written. Vague or incomplete instructions still lead to assumptions, and not always the ones you want. And crucially: they still don’t change the fact that LLMs are non-deterministic by nature. 

Thus, you should always consider a few practices that make the difference:

- **Ask before proceeding** — Encourage the model asking clarifying questions in case of uncertainty before it proceeds. Without that signal, LLMs tend to fill gaps with assumptions. And those often look plausible until you review the output. A clarifying question takes seconds; untangling wrong implementations takes much longer.
- **Think in iterations, not commands** — Structure complex skills inspired by features like Claude Code's planning mode: the model proposes, you review and refine, then it executes. This back-and-forth catches misalignment early, before it becomes a bug or tech debt.
- **Treat skills as living artifacts** — When a skill produces bad output, fix the skill. Every correction is an investment that pays forward.
- **Always review the output** — Skills narrow the gap between what you asked for and what you get, but they don’t close it entirely. The goal is to shift from rewriting to reviewing, not from reviewing to blind trust.

## 🪴 Excursion: Quality Assurance Beyond Skills

But skills alone aren’t enough — especially at scale. The hard truth is that neither AI nor the humans using it (or writing code themselves — *how boring*) can be fully trusted to stay within the lines. And in a world where code can be generated faster than ever, tech debt can compound just as fast. Thus, the more productive your team becomes, the more critical it is to keep things maintainable. Therefore, some guardrails that help are:

- **Quality gates** (e.g., SonarQube) — If code can be generated faster than ever, quality expectations need to keep pace. Enforce test coverage thresholds and code quality rules automatically, before anything lands on your main branch.
- **Linting and formatting** (e.g., ktlint, Prettier) — Consistent style isn’t just aesthetics — it keeps the codebase navigable and onboarding manageable, especially when multiple people are generating code at speed.
- **Architecture tests** (e.g., ArchUnit, Konsist) — Good software lives within a structure: clear layers & explicit dependencies. Consistent patterns that make it easy to reason about, extend, and onboard. Those decisions are worth documenting — ADRs are a natural home for them. But documentation doesn’t enforce itself. A developer takes a shortcut, an AI generates code without knowing your rules — and layer boundaries quietly erode. Automated architecture tests catch these violations before they land - preventing costly refactoring’s.

The [zeebe-native blueprint](https://github.com/Miragon/process-engine-blueprints/tree/main/examples/zeebe-native) contains working examples of these guardrails — specifically an architecture test setup based on [ArchUnit](https://www.archunit.org/) and [Konsist](https://docs.konsist.lemonappdev.com/) — if you want a concrete starting point.

## ✨ What Becomes Possible

So to sum it, up: using skills well isn’t friction-free. But in my experience, none of the challenges above are blockers. They are easily manageable with the right practices in place. And once they are, a lot becomes possible. From my perspective, the benefits include — among others:

- **AI becomes usable in domains with sparse training data** — Skills don’t wait for better training data. They supply what’s missing directly — engine-specific patterns, constraints, and working examples. That’s what makes AI assistance viable in niche domains today, not at some undefined point in the future.
- **More consistent output** — The same task yields similar results because the skill snaps context, examples, and constraints into place every time. Include a code snippet of your service task worker and nearly every worker looks identical. Same structure, same naming conventions, same layer boundaries.
- **Feedback-based work instead of guessing** — Follow-up questions surface uncertainty early. You've prompted the AI to send a signal to a process and it doesn't know which API to use? It asks — and you clarify — before a single line gets written. The result: no hallucinated API calls, just a quick clarification that perhaps prevents a rewrite.
- **More speed = more focus on what matters** — Skills cut the time spent on repetitive work significantly. Workers, adapters, tests. All of this gets implemented faster and is wired to your architecture. What used to take a few hours may now just require minutes. And the time you win back flows into the work that actually requires human judgment: process design, domain modeling, and architecture decisions - things that shouldn’t be automated away.
- **Faster prototyping** — Want to test whether a new capability — like an MCP server — would add real value for your users? A skill can generate a working prototype from your existing REST spec in a single session. Less friction between idea and working prototype means you can validate more ideas, earlier.
- **Shareable team knowledge** — Skills aren't just for individuals—they're versionable artifacts you commit to your repository. One person defines how a worker should be structured, and the entire team inherits it automatically. The main benefit: output stays consistent across the team. No per-person drift, no repeated explanations. Everyone follows the same conventions, aligned with what you've agreed on together.
- **Cumulative improvement** — Each correction you make can improve the skill. A skill with 80% accuracy today could reach 85% next week. And if your skills are shared, that improvement isn't just yours—it's the whole team's. Everyone benefits.

## 🎬 Wrapping Up

In short: Skills won't make your AI a perfect developer. But they can make it a genuinely useful collaborator. One that understands your domain, follows your conventions, and has access to whatever context you give it: project files, documentation, live data via MCP, and more. Its your colleague & pair-programming buddy. Even in domains where the model would otherwise fall short.

The pattern is straightforward: encode what the model doesn't know, keep a human in the loop, and let guardrails catch what slips through. Done consistently, the benefits compound — more consistent output, faster iteration, and a team that gets a little better with every run.

For Zeebe and BPMN, my skills live in the [bpmn-to-code skills plugin](https://github.com/Miragon/bpmn-to-code/tree/main/bpmn-to-code-skills). Feel free to copy individual skills or fork the whole repository and adapt them to your needs. If you run into issues or have suggestions for improvements, open an issue on the repository. It helps make the collection better for everyone. 

Two things worth noting: skills are not agent-specific. Since they're an [open standard](https://agentskills.io/home), any agent that supports them can use them—not just Claude Code. They're also not engine-specific. Swap out the code examples and conventions, and the same approach works with any process engine. The concept matters more than the specific implementation.

Beyond that plugin, I'd suggest browsing the growing number of skill repositories and marketplaces. There's a good chance someone has already built something useful for your domain. And when you combine skills—yours and theirs—the effect compounds: each one adds context the model can lean on, making the output sharper and more reliable.

Finally, don't stop at skills—explore what else your agent can do. In Claude Code, for example, [sub-agents](https://docs.anthropic.com/en/docs/claude-code/sub-agents) are a natural next step: isolated contexts where the model operates in a specialized role, powered by a skill. Think of a reviewer that checks BPMN consistency, or an architecture guard that verifies hexagonal boundaries. I've experimented with these as well, and combined with skills, they unlocked even more productivity—but that's a topic for another post.

## 🔭 What’s Next — and What I’m Curious About

Together with a few colleagues at [Miragon](https://miragon.io/), I'm continuing to build out skills for Zeebe and exploring the same approach for other process engines. The vision: a shared skill library for process automation—versionable, iteratively improved, and usable across teams. One that helps everyone move faster without sacrificing consistency or quality.

And if you'd like to bring this approach into your own team, we also run a hands-on training on exactly this: [AI in Process Automation](https://miragon.io/trainings/ai-tooling-workshop/). Across the BPM lifecycle, it covers how to set up AI agents, build skills for your domain, and establish the guardrails that keep quality high.

I'm curious about your experience. Have you tried using AI assistance in the context of process automation? If so: what works for you—and what doesn't? Are there specific tasks where the output consistently falls short? Or where you've found an approach that works well? 

👉 Feel free, to drop a comment or reach out!