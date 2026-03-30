---
layout: post
title: "Are you really vibe-coding CNVkit?"
date: 2026-03-30
description: Clinical considerations for bioinformatics development with AI coding agents.
tags:
  - software-engineering
  - tools
---

CNVkit is used in hundreds or maybe thousands of clinical and research labs worldwide. It's used in CAP/CLIA-, CE-, and FDA-regulated pipelines, at major NGS diagnostics companies and medical center pathology labs alike, to generate results used in treatment decisions for patients. It would be an severe lapse of duty for me to turn over the development of this project to an LLM.

That said, the [evidence](https://github.com/etal/cnvkit/releases) is suspicious, isn't it? CNVkit had two releases in 2024, mainly bug fixes and user contributions, and none at all in 2025. Months would go by without a commit; user questions floated in the abyss unanswered. In my darkest hours I considered archiving the repo on GitHub. Then [v0.9.13](https://github.com/etal/cnvkit/releases/tag/v0.9.13) landed in February with not only user-contributed fixes but also three new significant features: PHI-safe coverage processing via bedGraph, per-gene confidence interval calculation, and resurrection of HMM segmentation with a major dependency update. PRs are landing, issues are closing, tests are passing, GHA is doing more. And, obviously, Claude is writing code reviews on PRs and co-signing commit messages.

Part of the revival came from a collaboration with [Wei Gu Lab at Stanford](https://cfna.stanford.edu/). The rest is roughly what it looks like.

So, is it vibe coding? Not exactly. When Andrej Karpathy [first tweeted about the concept](https://x.com/karpathy/status/1886192184808149383), he framed it as a slightly silly experiment in pushing the capabilities of coding agents, and indeed things got a little silly industry-wide immediately afterward. Steve Yegge and Gene Kim's book [*Vibe Coding*](https://www.simonandschuster.com/books/Vibe-Coding/Gene-Kim/9781966280026) originally had draft titles like *Chat-Oriented Programming* (or *The CHOP Handbook*), but, you know, they had to follow the vibes.

It's too late to change common usage now, but here are the terms in my mind:
* "Vibe coding" -- just as Karpathy said: Turn off all safety checks, don't look at the diffs, force the agent to just follow your vibes and crank out code. If the result sucks, throw it away and start over. **This is not what I'm doing.**
* "Conversational programming" -- [Simon Wardley coined this term around 2016](https://blog.gardeviance.org/2023/01/why-fuss-about-conversational.html), before  real AI agents existed. He predicted that AI and developer tooling would converge on something that you work with through natural language conversation to build systems, but did not specify the interface beyond that.

I'm doing conversational programming with lots of manual intervention and rigorous review and verification. Specifically, I use an AI coding agent, Claude Code, and many other tools, including, as always, vim. I understand all the code that lands, and I stand by the results.

This topic is [beyond contentious](https://blastedbio.blogspot.com/2025/11/thoughts-on-generative-ai-contributions.html) in both long-running open-source projects and in bioinformatics, and CNVkit sits in the intersection. I'll share what I've learned about using AI coding assistants in serious bioinformatics development.

## A day in the life

My desktop looks like this, minimally:
* Claude Code in one terminal window
* A separate shell or two in other terminal windows or tabs, for vim, git, the python REPL, everything else
* Literature, docs, and GitHub in a web browser.

I used VS Code earlier, and created a dev container image to support that flow. But Claude Code's terminal UX is good now; I have more natural control over the shell environment there, and I'm comfortable wrangling diffs and git there, too. I've tried the Code integration built into Claude Desktop, and for now, I prefer having full control over the environment in the terminal shell. It's the same tooling under the hood.

I read Claude's patter and diffs as they come in. The [Explanatory Output Style plugin](https://claude.com/plugins/explanatory-output-style) elaborates on the agent's on-the-fly decisions, keeping me in the loop. The terminal icon bounces in my dock every time Claude Code makes edits, needs a permission authorization, or anything else, and that's a good opportunity to review what's happened and nudge the agent back onto the righteous path if needed, and also wrangle git and code directly in another terminal session if I know exactly what I want. 

If I make a significant change manually during a Claude session, e.g. rename or substantially edit a file, I tell Claude what I did so that it will update its context to keep aligned with reality. (This could be automated, but English works fine.) Claude doesn't see what it's not looking for, and there's no harm it telling it something it already knows. Better to over-communicate.

The iterative development loop looks like: plan, execute, test (ruff check, pytest, mypy), pre-commit and commit, then tell Claude to review its own work while I do my own review in another window or tab. Repeat as needed. Once the automated checks pass, push the branch to GitHub, open a PR, and have the Claude bot there do another review while the CI pipelines run across the full test environment matrix. Make more edits and iterate as needed, play with it locally to catch any unknown-unknowns, then land the branch when it's all clear.

The [Feature Dev plugin](https://claude.com/plugins/feature-dev) formalizes this flow nicely. It explicitly instructs the agent to:

1. Restate and confirm the initial request with the user
2. Review the relevant code
3. *Ask clarifying questions*
4. Launch 2-3 agents in parallel to draft implementation plans, each with slightly different priorities
5. Build the feature according to the selected plan
6. Launch 3 more agents to review the code, again each with different priorities
7. Finally, summarize the key decisions and results.

There are opportunities for discussion between each step. Using this plugin feels conversational rather than bureaucratic, and it is thorough. Within the 7-step framework the agent always knows what to do next -- meaning, I don't have to remember to manually tell it to do the next right thing; it just collates its own review output and deals with it, for example. I think [Beads](https://github.com/steveyegge/beads) would help it flow even more smoothly. Overall, this workflow is automated as much as I would want it to be, while keeping me fully knowledgable and accountable for the codebase. 

## Making it work

The CNVkit codebase is nearly 13 years old, and contains 72 source files at last count, not counting test data and other infrastructure. It's somewhat old, bigger than most bioinformatics research tools, but not huge.

Several aspects of this project make it an especially good fit for conversational programming.

First, I'm a solo maintainer. Many other people have contributed to the repo over the years, but aside from a spell a few years ago when a co-maintainer helped manage PRs, I've reviewed every change set that lands, and I know the full codebase. This means there is no coordination overhead, and the project's full context fits in my head. If Claude has a question, or gets confused, I usually can give the right answer without much additional investigation. I work on one or two features at a time and don't have to worry about merge conflicts, design disagreements, prioritization, or getting permission to go ahead, as I might in a larger organization.

I have a long list of [open issues](https://github.com/etal/cnvkit/issues), now finally shrinking. Most issue descriptions are very detailed; CNVkit users tend to be very smart and diligent people. I've written down feature ideas and pondered them for years without gathering the courage to implement them until now, so the tasks are well specified and the edge cases and ramifications are already in my long-term memory. There was also a backlog of maintenance tasks, like Docker deployment and Python version upgrades, that Claude can recognize as recurring chores in its training data, so it can one-shot those reliably.

On the point of training data -- CNVkit has extensive  [user documentation](https://cnvkit.readthedocs.io/en/stable/), it's used in many open-source pipelines and mentioned in many publications besides its own, and there's a deep corpus of questions and answers on BioStars, SeqAnswers, and GitHub Issues. In short, it's well represented in every frontier AI's training! Claude, ChatGPT, and Gemini can answer questions about CNVkit directly from their weights, even without any additional context. That makes planning mode pretty fluid -- its guesses are not perfect, but they're usually directionally correct and just need a few nudges from the real world. Those nudges may be my own design guidelines, links to papers, or a topic to look up in ePMC and synthesize.

CNVkit's dependencies are conservative. The PyData stack (numpy, scipy, pandas, matplotlib) and pysam are even better represented in the LLM training data, so generated code is idiomatic.

Python, like Rust and Go, is especially agent-friendly in several ways: The abundant open-source code in the training data is consistent due to the simple-but-not-too-simple language design; there is "one right way" to do most things, reinforced by a good "batteries included" standard library and stable ecosystem; the explicit type annotations, if given, help the agent reason and reduce its search space; and, crucially, error messages show what went wrong and what to do next. R is hard for agents because it's on the wrong side of all those criteria, but there's only a little bit of R in CNVkit, and that code has been stable for years.

I invested early in user docs, automated testing and other code hygiene tooling, and kept these up to date over the years. That created a smooth path for AI to self-test and verify success with standard tools both locally and in GitHub Actions, and incrementally update this same scaffolding. The automated code feedback loop allows the AI agent to use those tools on its own and correct the code it produces immediately. This is the key to making development fast and robust at the same time.

### Tending to CLAUDE.md

When you launch `claude` in a repo for the first time, it suggests that you run `/init` to generate a stub `CLAUDE.md` file. A stub is better than nothing, but investment here has compounding returns. [CLAUDE.md](https://github.com/etal/cnvkit/blob/master/CLAUDE.md) looks like a dense but fairly ordinary, human-readable developer onboarding doc, and it is in fact the briefing document for the AI assistant to read as soon as it starts up with an empty context window.

To recap Steve Yegge's ["Super Baby" analogy](https://steve-yegge.medium.com/beads-for-blobfish-80c7a2977ffa): a new AI agent pops into the world blinking, disoriented and shivering, with an empty context window. It knows absolutely nothing about your codebase or your goals, hasn't even listed the files in the current directory yet. It instinctively reaches for CLAUDE.md and pulls into context the very most essential things it needs to be productive here: project purpose, design principles, architecture, conventions, assumptions, quirks, style rules, test commands, execution environment, build artifacts, I/O strategies. Which tools to use, which conda environment to activate. URL of the CNVkit paper, in this case.

CLAUDE.md is the coding agent's primary education, and it determines how well things will go once you hand your agent a coal hammer and send it down into the mines, or whatever you happen to have in your task list. Tend to CLAUDE.md thoughtfully, as a comprehensive developer guide, because it's leverage for everything else the agent does.

It should be concise, actionable, and specific to the project. CNVkit's has these cues in a "Development Workflow" section near the top:

- **Bug fixes and new features**: Write a failing test first, then implement.
- **Edge cases**: Before finishing, verify behavior for empty inputs, NaN/missing values, and single-element arrays.
- **User-facing changes**: Update the relevant docs in `doc/*.rst`.
- **Clinical impact**: When reviewing changes, consider whether the changeset alters numerical output or output file formats (`.cnr`, `.cns`, `.cnn`, SEG, VCF). Flag any such changes explicitly, as downstream clinical pipelines may depend on exact output stability.

You can tell Claude Code at end of a session to update CLAUDE.md with any new learnings. The [CLAUDE.md Management plugin](https://claude.com/plugins/claude-md-management) is worth running periodically to catch the inconsistencies and redundancies that tend to arise. For Opus 4.6, the optimal length of CLAUDE.md seems to be on the order of 150-250 lines; beyond that it eats up a meaningful chunk of the context window with general tips where specific information from the current session would be more efficient.

### About the co-authorship thing

You might think coding agents would generally be an enormous help to existing open-source projects, and not just the single-author, single-use web apps that you see people crowing about on LinkedIn. Results have been mixed, it turns out.

Maintainers of existing projects [do](https://discuss.scientific-python.org/t/proposal-for-scipy-to-adopt-the-same-ai-policy-as-sympy/2230) [not](https://discuss.scientific-python.org/t/a-policy-on-generative-ai-assisted-contributions/1702) [want](http://www.mail-archive.com/numpy-discussion@python.org/msg08422.html) AI slop in pull requests. Even before coding agents, the real bottleneck in maintaining popular open-source projects was reviewing and managing merges, not writing new code. In submitting a feature branch, a new contributor is also implicitly or explicitly taking responsibility for not only the current code and documentation, but its continued maintenance, ideally forever. Vibe coding is by definition not that.

But... the coding agents actually work, too. Consequently, [open source AI contribution policies vary widely](https://github.com/melissawm/open-source-ai-contribution-policies). The more permissive ones still typically require that you disclose AI-assisted contributions.

Meanwhile, Claude Code has very boldly taken the liberty of signing itself as a co-author on commits:

```
Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

This is unusually cyberpunk for a developer tool.

The commentariat seem to hold two points in tension:
* **Attribution:** Commit authorship is for the responsible party, not for listing developer tools. VS Code and vim don't add themselves to commits.
* **Disclosure:** Actually, we really want to know when AI was substantially involved in a commit.

I'd prefer not to see the Claude icon in the Contributors list on GitHub, because the people on that list put real thought, time, and effort into their contributions for the benefit of other CNVkit users. But AI disclosure is important, too, here and in other areas, and the fine-grained metadata that the co-author line carries is useful for provenance and forensics.

The ongoing SymPy, SciPy, and NumPy discussions on AI-assisted contributions tipped me toward this perspective: The same ethics that apply to other contributors in other open-source scientific Python projects should apply to me, too. If Claude writes a commit message, I let the co-authorship line stay as a disclosure, because in the general sense that's what another maintainer my position would want to see flagged in a contribution. It's a courtesy to anyone who forks this repo, too.

Yes, I invoked Kantian ethics to decide whether or not to go to the trouble of disabling an auto-generated line in commit messages. Is it a useful flag to broadcast to others, given that I'm the repo owner and Claude's involvement is already obvious? Is it giving the agent too much credit and personhood, at the expense of "one wringable neck" accountability? I don't know, man, nobody knows.

### Limitations, mitigations

First, clinical labs using CNVkit in validated pipelines should, as always, re-validate after upgrading to any new version.

As for ongoing development, domain knowledge is indispensable, but once the agent has been given a solid plan, it generally doesn't get lost. It sometimes needs a quick correction, but the CNVkit repo is not a hard one for AI agents to navigate. I haven't seen Claude Code fall into a doom loop in this repo like I saw Cline do on the first web app I tried to vibe-code with it last year.

Planning is the main lever. AI has tunnel vision by default: it doesn't see what it's not looking for. Planning mode lets you argue and nitpick with your infinitely patient and tireless coding agent before letting it write anything, hammering out the ambiguities and edge cases while the iron is soft. Plan, review, iterate, and build your own tooling. Have it explain what it's doing, and ask it to double-check and review its own output if that's not already built into your automation. If you ask for plainly unnecessary or redundant work, Claude will quickly double-check and identify that it's already been done -- no harm done.

Interestingly, AI tends to find different bugs than a human would -- cryptic and subtle things like interface mismatches, type errors, unnatural edge cases. The way AI looks at code is different from the way I look at code. It's like it sees with compound eyes. (I know there's a bug joke crawling around in there somewhere.)

The recent [HMM rewrite](https://github.com/etal/cnvkit/pull/1001) is a good example. Literature shows that HMMs are a good conceptual representation of genomic copy number measured from noisy coverage profiles. Over several years, I tried hmmlearn, then a few early versions of pomegranate, then fell into a dependency-conflict trap which mostly resolved with the release of pomegranate 1.0. I asked Claude to help [fix the integration of pomegranate 1.x](https://github.com/etal/cnvkit/pull/910), and it did so. Then recently, I asked Claude how I could extend it to handle variable bin widths more effectively. Claude contains multitudes, and when asked this question in this way, it pointed out that fundamentally, the Baum Welch forward-backward algorithm isn't that complicated, and the number of hidden states needed for genomic copy number is small enough that pytorch's GPU acceleration makes no practical difference. A textbook, from-scratch implementation with numpy and scipy would provide all the functionality CNVKit needs, and none that it doesn't. *That had not occurred to me.* But HMM algorithm descriptions and examples abound (I wrote a couple in grad school for coursework), so Claude could easily handle this task, both writing and testing, once we talked through the details.

On scaling: The MCP plugin [Serena](https://claude.com/plugins/serena)  invisibly provides semantic code understanding through language servers (LSP), complementary to Claude Code's built-in abilities. Without it, the agent resorts to Unix coreutils like `grep` and `sed` , and sometimes bespoke, transient Python scripts, just to find the right strings and the code lines around them. The LSP saves both unnecessary processing steps and overall tokens. Its value grows with the codebase.

As of February 2026, you can use the Claude Code command `/insights` for a retrospective on your past sessions and recommendations for your workflow. Empirically, I have been using a very iterative, conversational style. Not every project requires this; if your projects tend to be one-shot and driven by (a) vibes or (b) coherent specs, rather than scientific literature that is sparsely represented in the training corpus and epistemically debatable, then an agent orchestrator should provide a lot of leverage.

### What if we tried more power?

Steve Yegge has [written](https://newsletter.pragmaticengineer.com/p/steve-yegge-on-ai-agents-and-the) [extensively](https://steve-yegge.medium.com/) about coding with agents. I won't recap any of it here, just go read it. On his "AI adoption scale", and a similar one [Nate B. Jones describes](https://www.youtube.com/watch?v=bDcgHzCBgmQ), I'm somewhere in the middle, varying by task. Working on a truly new feature in CNVkit takes full focus, but maintenance tasks, like setting type annotations or fixing the hundreds of linting errors `ruff` coughed up at me a few months ago, I can let agents independently group into batches and run headlessly.

To reach the top of the scale, running a "dark factory" with a dozen or more concurrent agents, you need bulletproof and fully automated DevOps, CI/CD, end-to-end testing, review-by-agents, rewriting-by-agents, and other techniques that are [still being explored](https://steve-yegge.medium.com/six-new-tips-for-better-coding-with-agents-d4e9c86e42a9). You also need enough work specified clearly enough to keep the agents busy, which is a novel problem to have. It's interesting that the most elite vibe-coders, extremely smart and talented people, seem to go full spiral and start creating tools to enable [ever more vibe-coding](https://steve-yegge.medium.com/the-ai-vampire-eda6e4f07163). The software bloat perpetuates itself, with repo sizes disproportionate to functionality. The enormous READMEs you see -- Claude tells me these are not helpful to the agent; they fill up context, whereas a concise README would be more effective. They grow because cleanup is a separate step beyond adding a new section to an existing README, and given their druthers, both agents and humans would rather be generating new code.

I'm tempted to push back on the "AI adoption scale" framing itself. Simon Wardley and Tudor Girba offer a [dissenting opinion](https://medium.com/feenk/rewilding-software-engineering-900ca95ebc8c), in which software engineering is a *decision-making process* about systems that may be too large to fully grasp, and AI can help make better decisions about what to build and how. In other words, AI not only generates content efficiently, but can also to maximize intelligence, knowledge, and rigor. Let's not conflate productivity with code volume. I develop CNVkit with AI to improve the quality of the software and its results, and cases like the HMM rewrite and [this elusive NaN-poisoning bug](https://github.com/etal/cnvkit/pull/1036) are evidence that this approach is producing a better, more reliable product.

## Staying focused

Taking a step back, there is a point past which writing good code is no longer the bottleneck. Per the [theory of constraints](https://en.wikipedia.org/wiki/Theory_of_constraints), getting to your goal requires crossing through some series of prerequisites. At a given moment, one of those prerequisites is your bottleneck, and that's the one thing you need to optimize to reach your goal. Once you've put some work into improving that bottleneck, it's no longer your rate-limiting step; something else is. Now it's more productive to go work on that something else.

CNVkit had some technical debt, and the bugs that users had reported were obscure. The PyData ecosystem dependencies have developed very fast during the past 13 years, and Python itself has evolved from a cool, fun, type-casual, very high-level language with confusing notions about Unicode to a fast, optionally typed, concurrency-curious, industry-dominant language with confusing notions about Unicode.

With modern tooling, I can now maintain CNVkit to a high standard and keep with the software ecosystem around it. Outside of Claude Code and its plugins, I'm not using the new agent orchestration tools (e.g. [Gas Town](https://github.com/steveyegge/gastown), [RuFlo](https://github.com/ruvnet/ruflo), [Agentic Coding Flywheel](https://github.com/Dicklesworthstone/#the-agentic-coding-flywheel) -- yet!), because generating code is no longer the bottleneck for CNVkit development. I now spend more of my active wall-clock time on CNVkit just reading about the possibilities, thinking about what to do for a given issue or scenario, and running small experiments. That's time better spent.
