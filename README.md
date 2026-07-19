# AI Framing Skills

Two free skills that run my AI problem framing method on your actual project. **frame-use-case** pressure-tests an AI idea before you build it. **diagnose-use-case** reads the evidence from a system that's already running and makes the persist, pivot, or stop call. MIT licensed, no email wall, nothing to sign up for.

Here's why they exist. Ask ChatGPT "I want to build an image classifier to track which cars are parked in my garage, what advice would you give me?" and you get fifteen sections of correct advice: label taxonomies, camera placement, temporal smoothing. I checked, every word was right. And not one line asks whether a $200 counter at each ramp solves the problem. A chat model answers the question you asked, and you asked about a classifier.

The framing knowledge is already in the model. I ran 50 realistic project questions through the same ChatGPT model twice, bare and with the skill, and blind-scored both on three fundamentals: did it establish the goal, did it surface a metric, did it weigh an alternative. Bare ChatGPT frames well when the question is well-posed. But it reliably skipped the metric, asked the user a question in fewer than a third of runs, and never once said don't build. The skill forces exactly those three. And to be fair about it: the first version of this skill failed the same eval, producing complete, confident frames built on numbers it invented. I rewrote it to ask instead of manufacture, and only then did it beat the baseline on every fundamental.

## The skills

**frame-use-case** interviews you through the AI Framing Worksheet from my course: eight questions, three sections, four quick tests. It names the trap first ("you've named a chatbot, which is a solution"), asks at most three questions per turn, and refuses to invent numbers you didn't give it. The output is the completed worksheet, with genuine unknowns left open and ranked. It is allowed to conclude the project shouldn't exist. That's most of its value.

**diagnose-use-case** works on a system that's already running. It works from evidence, not from what you say about the evidence: your numbers against the original targets, 20 to 50 real failure examples, the assumptions you wrote down when you framed it. Come with those and it ends in a call: persist, pivot, or stop. Come without them and it tells you what to collect first.

The two hand off to each other. If frame-use-case wrote a `FRAMING.md` into your repo, diagnose-use-case starts by checking which of those assumptions held.

## Install

**Claude Code:**

```bash
git clone https://github.com/rajshah4/ai-framing-skills.git
cp -r ai-framing-skills/frame-use-case ai-framing-skills/diagnose-use-case ~/.claude/skills/
```

Then describe your project: "help me frame this AI use case" or "my pilot's metrics are flat, diagnose it." Or skip the terminal entirely and tell Claude Code: install the skills from https://aiframer.dev/setup.md.

**Cursor, Copilot, Gemini CLI, OpenHands, or any tool supporting the [Agent Skills](https://agentskills.io) standard:** copy the two skill folders into that tool's skills directory. Nothing here is Claude-specific.

**No agent tool:** paste the contents of `frame-use-case/SKILL.md` into any AI chat and describe your project. It runs fine as a plain prompt.

## What no skill can check

The limits are structural, not bugs. These skills have no ground truth: they can't tell you what happened to the last twelve companies that tried your architecture. They can't calibrate your numbers: a 75% target might be ambitious or sandbagged, and no model knows which. They don't sit in your org, where the politics that kill projects live. And they will still agree with you more than a skeptical human would, because that's how these models are built. The skills say all of this out loud at the end of every session.

The judgment behind the questions is the course: [AI Problem Framing for AI Practitioners](https://maven.com/rajistics/ai-problem-framing) on Maven, where you practice these calls on your own project, against a database of 250+ verified AI case studies with real outcomes. More at [aiframer.dev](https://aiframer.dev).

## License

MIT. Copy the SKILL.md into your own prompts if you want. That's the point.
