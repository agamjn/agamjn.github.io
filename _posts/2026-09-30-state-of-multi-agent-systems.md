---
layout: post
title: "State of Multi-Agent Systems"
author: Agam Jain
show_on: Articles
date: 2026-09-30
---

Before we get into multi-agent systems, let's first understand what an agent does and what changes when we ask several of them to work together.

## From chat to agents

An AI agent works toward a task by choosing an action, observing what happened, and deciding what to do next. A coding agent might read a file, change it, run a test, and use the error to decide what to fix.

Three parts make this possible: the **model** reasons about the task; the **harness** manages context, tool calls and the execution loop; and the **environment** contains the tools and resources it can use, such as a repository, browser or email account.

**November 2022: Chat.** With ChatGPT, you could ask for code, copy it into an editor, run it and bring errors back. You managed the feedback loop.

**September–November 2024: App builders.** Replit Agent, [Bolt.new](http://Bolt.new) and Lovable brought generation, execution and previews into one interface. You described an app and guided its development through conversation.

**February–April 2025: Interactive coding agents.** Claude Code and Codex CLI could work inside your project, reading files, editing code and running tests while you directed and reviewed the work.

**May–October 2025 onward: Delegated cloud agents.** Cloud Codex and Claude Code on the web let you assign work in separate cloud environments and return later to review it.

## What makes a system multi-agent?

One agent can use several tools, run for hours or work on a persistent computer. A multi-agent system adds other agents that make decisions within their own assignments and coordinate their contributions.

For example, a Codex CLI agent can read files, edit code and run tests by itself. In Cursor's January 2026 experiment, planners created tasks, workers implemented them, and a judge decided whether another cycle was needed. The work was distributed across several decision-making agents. [Cursor's architecture](https://cursor.com/blog/scaling-agents#planners-and-workers)

“Local” and “cloud” describe where the system runs. “Single-agent” and “multi-agent” describe how the work is organized.

## When does adding agents help?

It helps to start with the difficulty one agent is facing. Is there too much to investigate? Does the work need an independent review? Or is the project large enough that planning and implementation compete for attention?

These lead to three common arrangements. They overlap, and predate the recent interest in swarms. **January 2024:** LangGraph already described shared workspaces, supervisors and hierarchical teams. [Early collaboration patterns](https://www.langchain.com/blog/langgraph-multi-agent-workflows)

### A coordinator divides a broad investigation

**June 2025:** Finding board members across S&P 500 technology companies required checking many sources. A single agent searched sequentially and failed to complete the answer. A lead agent with parallel researchers found the answers.

Each worker had its own context and tools and could follow useful leads. The coordinator combined the findings. The human did not need to prescribe every search. [Parallel research](https://www.anthropic.com/engineering/multi-agent-research-system)

### One agent builds; another checks

**March 2026:** In an application-building experiment, a planner expanded a short request into a specification, a generator built the application, and an evaluator tested it through a browser. Broken interactions became feedback for the next attempt. [Building and evaluating applications](https://www.anthropic.com/engineering/harness-design-long-running-apps)

**April 2026:** Devin used a separate reviewer with fresh context to examine code changes. The coding agent then assessed the findings against the task. The extra agent contributed another check on work the builder might otherwise consider finished. [Devin's review loop](https://cognition.com/blog/multi-agents-working)

### Agents keep planning as a project develops

**January 2026:** Building a browser meant discovering new problems while implementing earlier decisions. Cursor separated continuous planning from implementation so workers could focus on individual tasks.

**February 2026:** Anthropic's compiler experiment used a shared repository and task locks, without a central coordinating agent. So a swarm does not imply one fixed structure, direct communication between every agent, or a separate computer for each. [Cursor's project experiment](https://cursor.com/blog/scaling-agents), [Parallel compiler agents](https://www.anthropic.com/engineering/building-c-compiler)

For a bounded investigation, the difficulty is often coverage. For an evolving project, it also includes keeping changes compatible as the plan develops. In either case, the work needs to be divided, checked and brought back together effectively.

## What do the results show?

The studies measure different things. Benchmark success, a playable application and bugs found during review should not be read as the same measure of accuracy.

| Study and arrangement | Reported result | Cost and comparison limits |
| --- | --- | --- |
| **June 2025: Anthropic research.** Opus 4 lead with Sonnet 4 workers. | **90.2% improvement** over single-agent Opus 4 on its internal research evaluation. | No matched dollar comparison. This is a relative performance improvement, not a 90.2% success rate. [Study](https://www.anthropic.com/engineering/multi-agent-research-system) |
| **January 2026: Google's Finance-Agent results.** Coordinator and workers. | Mean success rose from **34.9% to 63.1%**. | The authors report matched reasoning-token budgets and standardized tools and prompts. Equal dollar cost is not established. [Paper](https://arxiv.org/html/2512.08296v2) |
| **January 2026: Google's PlanCraft results.** Coordinator and workers. | Success fell from **56.8% to 28.2%**. | Each action changed the state needed for the next. Extra coordination consumed effort without helping the sequential work. [Paper](https://arxiv.org/html/2512.08296v2) |
| **March 2026: Anthropic game-making application.** Planner, generator and evaluator. | Solo: **$9 and 20 minutes**, with broken controls. Full harness: **$200 and six hours**, with a playable core and remaining problems. | Both used Opus 4.5, but scope and supporting software also changed. This does not isolate the effect of adding agents. [Experiment](https://www.anthropic.com/engineering/harness-design-long-running-apps) |
| **April 2026: Cognition code review.** Coding agent and separate reviewer. | Average of **two bugs detected per PR**, approximately **58% severe**. | No controlled single-agent baseline, comparative cost or final correctness rate. [Report](https://cognition.com/blog/multi-agents-working) |
| **July 2026: OpenAI command-line tasks.** Four coordinated agents. | Terminal-Bench 2.1 rose from **88.8% to 91.9%**. | Used additional tokens; an equal-compute advantage was not established. [Evaluation](https://openai.com/index/gpt-5-6/) |

*The Google overview appeared in January 2026; its paper was published in December 2025.*

Cost needs its own comparison. The research-system report estimated ordinary agents at about **4× the tokens of chat**, and multi-agent systems at **15×**. Those figures were not costs for completing the same task. [Token use](https://www.anthropic.com/engineering/multi-agent-research-system)

**July 2026:** Cursor's SQLite swarms cost between **$1,339** using Opus 4.8 with Composer 2.5 and **$10,565** using GPT-5.5 throughout. All new configurations eventually passed the held-out test suite. The solo runs were graded informally, so this compares swarm configurations without establishing savings over an equally capable single agent. [Swarm economics](https://cursor.com/blog/agent-swarm-model-economics#model-economics)

These results give us reasons to use multiple agents for particular tasks. They do not establish a general improvement in quality, cost and completion time together.

## What goes wrong, and what has helped?

Adding workers also adds opportunities for misunderstanding. The reported failures make this concrete.

**June 2025: Unclear assignments led to duplicated research.** In a semiconductor investigation, one worker researched the 2021 shortage while two repeated current supply-chain research. Early versions also launched 50 subagents for simple queries. Clearer task boundaries, output requirements and effort budgets helped address both problems. [Delegation failures](https://www.anthropic.com/engineering/multi-agent-research-system)

**January 2026: Shared work became a queue.** Cursor's agents held locks too long or failed to release them, leaving 20 agents with the effective throughput of two or three. Equal-status agents also favored small, safe changes while difficult work went unclaimed. Separating planning and implementation gave those responsibilities clearer owners. [Coordination and ownership](https://cursor.com/blog/scaling-agents#learning-to-coordinate)

**February–July 2026: Parallel work could converge on the same blocker or diverge into incompatible designs.** Compiler agents repeatedly attempted the same problem and interfered with one another's changes; better test partitioning made independent work possible. An older Cursor swarm produced three separate SQL packages. Its later design gave planners responsibility for shared decisions. [Compiler coordination](https://www.anthropic.com/engineering/building-c-compiler), [Conflicting designs](https://cursor.com/blog/agent-swarm-model-economics#split-brain-design)

**April 2026: An agent did not always know when to ask for help.** In Cognition's “Smart Friend” experiment, a weaker model could consult a stronger one, but deciding when to consult it and what context to send remained difficult. Access to a stronger agent was only useful if the first agent could make good use of it. [Escalation problems](https://cognition.com/blog/multi-agents-working)

## Who is building, and for which market?

By September 2026, these approaches were appearing in developer platforms, coding products and specialist applications. The companies below operate at different layers; a model launch and a finished agent product are different offerings.

| Company and publication date | Market and approach |
| --- | --- |
| **Anthropic · 28 September 2026** | Coding and knowledge work. Sonnet 5.5 targets faster, cheaper execution for everyday agent tasks. [Model announcement](https://www.anthropic.com/claude-sonnet-5-5) |
| **OpenAI · 10 September 2026** | Agent developers. Agents API combines managed sandboxes with parallel subagents, each with its own context. [Agents API](https://openai.com/index/introducing-the-agents-api/) |
| **Cursor · 10 September 2026** | Software teams. Projects combines a persistent coordinator, shared context and local or cloud subagents for features and migrations. [Projects](https://cursor.com/blog/projects) |
| **Cognition / Devin · 11 September 2026** | Software teams. Fusion pairs a frontier lead for planning and review with a cheaper coding agent, each keeping its own context. [Fusion](https://cognition.com/blog/local-fusion) |
| **Harvey · 2 September 2026** | Legal teams. Playbook Review assigns contract rules to agents working on separate document branches; a lead resolves conflicting edits. [Playbook Review](https://www.harvey.ai/blog/rebuilding-playbook-review-as-a-multi-agent-system) |
| **Legora · 15 September 2026** | In-house legal teams. Intake gathers context, handles routine requests and prepares complex work for counsel. [Legal intake](https://legora.com/blog/the-real-cost-of-legal-intake) |
| **Lovable · 15 September 2026** | Business app builders. Its Salesforce and Slack integrations bring app and agent creation into existing workflows. [Product announcement](https://lovable.dev/blog/salesforce-partnership) |

The Legora and Lovable posts describe agent-related products without establishing a multi-agent architecture. This is a view of product direction, rather than a ranking of performance.

Other companies to follow include Replit, LangChain, CrewAI, Basis, Sierra and Decagon.

## What still needs to improve?

The open questions follow from these failures: how much work should the coordinator delegate, which decisions must remain shared, and how does the team recover when an agent fails? Work across different systems also needs to preserve permissions and the meaning of the original task. [Scaling research](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/), [Communication across systems](https://cloud.google.com/blog/products/ai-machine-learning/unlock-ai-agent-collaboration-convert-adk-agents-for-a2a)

This gives evaluations and simulations a specific job. Recreate a coordination failure, change one mechanism, and check whether the team completes the work more reliably, cheaply or quickly. Compare it with the earlier team and a capable single agent. That is how we can tell whether adding agents is helping.
