# AI Framing Skills

Two skills that run my AI problem framing method on your actual project. 

**frame-use-case** pressure-tests an AI idea before you build it. 

**diagnose-use-case** reads the evidence from a system that's already running and makes the persist, pivot, or stop call. MIT licensed, no email wall, nothing to sign up for.

My motivation was the limits of ChatGPT. I found that while it gave impressive answers, it didn't help you properly frame your problem, think through all the steps, and push back on you. To ensure this skill works well, I did build an eval set of 50 realistic project questions. But even with that, please share feedback if you see gaps or places the skills can improve.

## The skills

**frame-use-case** interviews you through the AI Framing Worksheet from my course: eight questions, three sections, four quick tests. It names the trap first ("you've named a chatbot, which is a solution"), asks at most three questions per turn, and refuses to invent numbers you didn't give it. The output is the completed worksheet, with genuine unknowns left open and ranked. It is allowed to conclude the project shouldn't exist. 

**diagnose-use-case** works on a system that's already running. It works from evidence, not from what you say about the evidence. It wants your numbers against the original targets, 20 to 50 real failure examples, the assumptions you wrote down when you framed it. If you bring those, it will help you with the decision around persist, pivot, or stop. If you don't have that, it tells you what to collect first.

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

These skills have no ground truth. They can't tell you what happened to the last twelve companies that tried your architecture. They can't calibrate your numbers. A 75% target might be ambitious or sandbagged, and no model knows which. They don't sit in your org, where the politics that kill projects live. And they will still agree with you more than a skeptical human would, because that's how these models are built. The skills say all of this out loud at the end of every session.

The judgment behind the questions is the course: [AI Problem Framing for AI Practitioners](https://maven.com/rajistics/ai-problem-framing), where you practice these calls on your own project, against a database of 250+ verified AI case studies with real outcomes. More at [aiframer.dev](https://aiframer.dev).

## License

MIT. Copy the SKILL.md into your own prompts if you want. That's the point.
