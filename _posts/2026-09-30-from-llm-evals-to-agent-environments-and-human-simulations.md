---
layout: post
title: "From LLM Evals to Agent Environments and Human Simulations"
author: Agam Jain
show_on: Articles
date: 2026-09-30 19:20:00 -0700
---

Before we get into the companies building evals and simulations, let's understand how testing changed when we moved from chatbots to agents that can take actions.

## 2023–2024: Testing LLM applications

When teams were building chatbots and document assistants, a common evaluation setup was a set of questions, expected answers and a few checks. Run these against different models or prompts, compare their responses and inspect where the application failed.

Tools helped manage test sets, collect feedback and repeat tests after changes. LangSmith which became generally available in February 2024, is one example. Others are Vellum, promptfoo, etc. The team supplied its application and test cases; the platform helped evaluate and debug it.

**Test questions → LLM application → responses → scoring and comparison**

![](/assets/images/llm-evals-agent-environments/llm-evaluation-diagram.png)

## Agent evaluations: checking whether the work got done

Then agents came. Asupport agent can issue a refund, while a coding agent can change a repository. We need to check whether the refund actually exists or the code works, beyond what the agent says it did.

An agent evaluation gives the system a task, lets it use tools and checks its actions and the result. The diagram below shows the parts involved.

![](/assets/images/llm-evals-agent-environments/agent-evaluation-diagram.png)

The **task** tells the agent what to do. The **environment** contains its tools and data, such as a repository, terminal and tests for a coding task.

The **agent harness** is the software that lets the model use its tools. The **evaluation harness** runs attempts and collects results. **Graders** assess the recorded actions and final outcome.

A **benchmark** brings tasks and scoring rules together so we can compare systems under defined conditions. These components help explain what different companies are selling.

## Who provides which part?

As of September 2026, some companies provide tools to run your evaluations. Others build tasks and environments, or publish comparisons to help you choose a model.

| Company | What it provides | Why a team would use it |
| --- | --- | --- |
| [**LangSmith**](https://www.langchain.com/blog/langsmith-ga) | Tools to run evaluations, manage test data and inspect what the application did. | Run tests, inspect failures and compare changes. |
| [**Vals AI / Vals Smith**](https://www.vals.ai/vals-smith) | Benchmarks for professional work, plus custom coding evaluations generated from repositories. | Compare models on relevant work, including your own codebase. |
| [**Artificial Analysis**](https://artificialanalysis.ai/about) | Independent capability, speed and cost measurements, with paid data and custom benchmarking services. | Compare models and providers before choosing a setup. |
| [**Mechanize**](https://www.mechanize.work/what-working-here-is-like/) | Software-engineering environments and graders for frontier coding agents. | Train or test agents on building features, deployment and debugging. |
| [**Bespoke Labs**](https://bespokelabs.ai/blog/bespoke-labs-raises-40m-to-build-environments-that-enable-reliable-agents) | Workplace environments containing codebases and connected software services, with tools to evaluate and improve agents. | Train and evaluate agents on extended workflows. |
| [**Scale AI**](https://scale.com/rlenvironments) / [**Turing**](https://www.turing.com/case-study/building-production-grade-web-environments-and-verifier-backed-tasks-for-computer-use-agent-rl-training) | Simulated applications, realistic data, task collections and verification. | Obtain ready-made or custom settings for agent training and testing. |
| [**Surge AI**](https://surgehq.ai/enterprise) | RL environments, expert-written grading criteria, checks and human evaluation. | Build tasks and scoring that require domain expertise. |
| [**HUD**](https://www.hud.ai/) | Tools to build, run and inspect environments, plus a marketplace connecting suppliers with research teams. | Operate environments or sell them to model developers. |

These businesses overlap. A usable environment needs tasks and a way to check results, while a benchmark needs somewhere to run. The same company may therefore provide several parts.

## What does buying an environment actually mean?

Suppose you want to compare coding agents on your company's work. Vals Smith turns suitable merged pull requests into tasks, with an issue description, hidden tests and a reproducible setup. The agent receives the repository before the fix. Its changes are checked against tests and for regressions.

This packages the task, starting environment, grading and comparison together.

For a browser agent, the environment might be an entire application. Turing describes building simulated retail and food-delivery websites with products, menus, inventory and reviews, along with more than 500 tasks and verification logic.

The contents depend on the job. Here are four illustrative setups:

| Agent | Environment | Example check |
| --- | --- | --- |
| Repository coding agent | Codebase, terminal, dependencies and tests | Did the change fix the issue without breaking existing behavior? |
| App-building agent | Browser, backend, database and test integrations | Can a user complete the intended workflow? |
| Harvey- or Legora-like legal agent | Contracts, company playbook, reference sources and document tools | Are the proposed changes appropriate and supported? |
| Healthcare administration agent | Synthetic patient records, scheduling tools and workflow rules | Was the referral handled correctly, including necessary escalation? |

## 2026: Environments for training as well as testing

We can also use these settings for reinforcement learning, or RL. The agent attempts tasks, receives rewards and a training process uses those attempts to update the model.

**Task environment → agent attempts → rewards → model updates**

In February 2026, Scale introduced an offering that combines simulated workflows, business data, expert-designed tasks and verifiers. Customers can bring their own models and the software that runs them.

The distinction is what happens after scoring. An evaluation measures current performance. RL uses the experience to change the model, followed by separate evaluations to check whether it improved.

## Environments and human simulations

So far, the AI agent has been performing work. Human simulation uses AI to explore how people might respond to a decision.

A simulation models how a system behaves so we can change something and explore what might happen. Human-behavior simulations model people's preferences, circumstances and decisions, and sometimes their interactions.

Suppose a pharmacy wants to improve its prescription-refill experience. It could simulate customers' responses to more frequent reminders or clearer instructions before testing those changes with real people.

### How is this different from an RL environment?

An RL environment can itself be a simulation. What changes is the purpose.

For a pharmacy support agent, we provide tools, customer scenarios and rewards for handling requests correctly. The aim is to improve the agent. For a new pharmacy service, AI represents customers whose responses we want to understand.

|  | Agent training environment | Human-behavior simulation |
| --- | --- | --- |
| Example question | Can our agent process a refill request correctly? | How might customers respond to a new refill service? |
| Role of the AI | Performs the work being tested or trained | Represents the people we want to understand |
| Result | Performance scores and experience for training | Estimated reactions, choices and differences between scenarios |

The two can overlap: a simulated customer could speak to a support agent during training. A business making a decision can also use simulated customers directly, without bringing its own AI agent.

### What Simile did with CVS Health

CVS Health is a U.S. healthcare company that operates CVS Pharmacy stores and provides health insurance, prescription benefits and healthcare services. Its 2026 white paper describes using Simile to explore customer journeys, test digital experiences and examine competitive perceptions.

The simulated people are grounded in human data, including interviews and past choices. Teams introduce a scenario and examine responses across different customer groups.

CVS first used simulations to reproduce findings from existing research. It then explored factors such as access to pharmacists, waiting times and clarity of communication. It also tested combinations of reminder frequency, educational content and benefit design to estimate how they might affect customers' intent to refill prescriptions or use clinical services.

**Human data → simulated customers → alternative service designs → estimated responses → selected real-world pilots**

CVS used the results to prioritize experiments. The public case does not report a measured increase in actual refill rates. Estimated intent still needs to be checked against actual behavior.

demo: [Watch Simile's introduction](https://www.reddit.com/r/singularity/comments/1r34xd9/introducing_simile_the_simulation_company/)

## Who is buying what?

These purposes lead to different buyers. Model-training teams need experience for learning, agent builders need evidence that their product works, and customer-research teams want to explore decisions before committing to them.

| Buyer | What they need | What suppliers deliver | Current examples |
| --- | --- | --- | --- |
| **Model-training teams** (OpenAI, Anthropic, Meta, Google) | Experience that teaches models new capabilities | Executable environments, difficult tasks, reference solutions, rewards and quality checks | [Scale](https://scale.com/rlenvironments), [Turing](https://www.turing.com/case-study/powering-servicenow-enterpriseops-gym-with-execution-grounded-workflows), [Surge](https://surgehq.ai/enterprise), [Mechanize](https://www.mechanize.work/what-working-here-is-like/) |
| **Agent-product engineering teams** (Factory, Notion, [monday.com](http://monday.com)) | Know whether a model, prompt or agent change improves their product | Production tracing, evaluation datasets, experiments and release checks | [Braintrust](https://www.braintrust.dev/customers/notion), [LangSmith](https://www.langchain.com/blog/customers-factory) |
| **Teams needing custom benchmarks** (Cognition, Ramp, ServiceNow) | Test agents on work that resembles their customers’ work | Repository-specific coding tasks or expert-authored domain tasks, environments and graders | [Vals Smith](https://www.vals.ai/vals-smith), [Mercor](https://www.mercor.com/enterprise-evals/) |
| **Application companies doing post-training** (Harvey) | Improve quality or reduce inference costs for a particular workload | Representative training data, reward design, training infrastructure and specialized models | [Applied Compute with Harvey](https://www.harvey.ai/blog/training-frontier-review-table-models-with-applied-compute) |
| **Environment and training-data suppliers** (Mercor, Sharpe) | Build, execute, validate and distribute their products | Environment infrastructure, quality checks and access to buyers | [HUD](https://www.hud.ai/), [Prime Intellect](https://www.primeintellect.ai/blog/lab) |
| **Customer research, product and strategy teams** (CVS Health, Itaú, Wealthfront, EY, Teneo) | Explore how people might respond before committing to a decision | Simulated populations, scenario comparisons, research reports and validation | [Simile](https://www.simile.com/), [Aaru](https://aaru.com/simulation), [Artificial Societies](https://societies.ai/case-studies/teneo) |

The bracketed names illustrate buyer groups; individual supplier relationships differ.

### What the customer cases show

Here are a few examples of what these teams actually used.

**June 2024: Factory and LangSmith.** Factory used self-hosted tracing and a feedback loop that turned agent feedback into evaluation datasets. It reported **2× faster iteration** than its earlier manual process. Here, a coding-agent company used outside infrastructure to improve its product. [Customer case](https://www.langchain.com/blog/customers-factory)

**October 2025: EY and Aaru.** EY used simulated respondents to recreate a wealth-management survey. It reported completing the simulation in **one day**, compared with a normal **six-month research cycle**, with a median **Spearman correlation of 0.90 across 53 questions**. That describes agreement in how responses were ranked. It does not mean the simulation could predict future purchases with 90% accuracy. [EY's account](https://www.ey.com/en_gl/insights/wealth-asset-management/how-ai-simulation-accelerates-growth-in-wealth-and-asset-management)

**March 2026: ServiceNow and Turing.** Turing contributed tasks, reference execution paths, sandbox engineering and verification to EnterpriseOps-Gym. The benchmark contains **1,150 tasks across eight domains**, and the research paper acknowledges Turing's contribution. This documents a delivered evaluation project; it does not establish a financial return in production. [Turing's account](https://www.turing.com/case-study/powering-servicenow-enterpriseops-gym-with-execution-grounded-workflows), [research paper](https://arxiv.org/abs/2603.13594)

**2026: Cognition and Ramp with Mercor.** Mercor describes building expert evaluation tasks for software engineering and accounting. It reports **200 tasks in five weeks for Cognition**, with **2× faster model iteration**, and **160 tasks in six weeks for Ramp**, with **3× faster iteration**. The fees and methods used to measure those speed improvements are not disclosed. [Mercor's case summaries](https://www.mercor.com/enterprise-evals/)

**July 2026: Mercor and Deeptune.** When announcing plans to acquire Deeptune, Mercor confirmed that it was already a customer. Deeptune built reconstructed enterprise applications for training environments. An expert-data supplier was buying the software environments needed to use that expertise. [Mercor's announcement](https://www.mercor.com/blog/mercor-to-acquire-deeptune/)

**August 2026: Harvey and Applied Compute.** The two companies trained a specialized model for Harvey's legal-document Review Table product. Harvey reported **54.8% lower cost per answer cell than Claude Sonnet 5** on the evaluated tasks, with improved answer quality. The result applies to that particular workload. [Harvey's account](https://www.harvey.ai/blog/training-frontier-review-table-models-with-applied-compute)

**September 2026: Itaú and Simile.** Itaú used simulated customer research around Brazil's recurring-payment experience. The reported research cycle went from **five weeks to four business days**, while concept exploration went from **two weeks to under three hours**. The case describes faster research, without reporting a demonstrated increase in product adoption. [Simile's case study](https://www.simile.com/blog/itau-at-esomar)
