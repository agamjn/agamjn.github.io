---
layout: post
title: "harness hosting"
author: Agam Jain
show_on: Articles
category: Technical
date: 2026-09-24 21:22:00 -0700
permalink: /harness-hosting/
description: "the components, durability, access boundaries, and hosting choices behind a cloud agent."
---

<style>
.harness-table { overflow-x: auto; margin: 1.5rem 0; }
.harness-table table { width: 100%; min-width: 620px; margin: 0; font-size: .9rem; }
.harness-table th, .harness-table td { vertical-align: top; }
.harness-table:focus-visible { outline: 2px solid #d9531e; outline-offset: 3px; }
.harness-diagram { margin: 1.5rem 0; padding: 1rem; background: #fff; border: 1px solid #ececec; border-radius: 6px; }
.harness-diagram img { display: block; width: 100%; height: auto; }
@media (max-width: 600px) { .harness-diagram { padding: .25rem; } }
</style>

## what is harness?

harness is a piece of code that turns model responses into an ongoing process of observing, acting, and checking results.

- the model returns text or structured tool calls.

- the harness supplies context, executes permitted calls, records their results, and decides whether to continue, pause, or stop.

hosting a harness means operating this program and the resources it depends on. the central decisions are where work executes, what survives a failure, and what the agent is allowed to access.

let’s use one example throughout: “fix issue #482: checkout becomes negative when the discount exceeds the subtotal. add a regression test and prepare a pull request.”

## components of agent loop

these are logical responsibilities. they do not each need a separate server, and the supporting services are not all part of the harness runtime.


<div class="harness-table" role="region" aria-label="comparison table 1" tabindex="0"><table><thead><tr><th scope="col">component</th><th scope="col">what it contains</th><th scope="col">example in our cloud agent</th></tr></thead><tbody><tr><td><strong>harness runtime</strong></td><td>agent loop, context builder, instruction/skill loaders, model client, tool registry, session logic</td><td>coordinates the checkout fix</td></tr><tr><td><strong>model inference service</strong></td><td>hosted model api or self-hosted model serving</td><td>receives context and returns text or tool calls</td></tr><tr><td><strong>agent execution environment</strong></td><td>vm/container, bash, git, node, file tools, application processes</td><td>executes <code>npm test</code></td></tr><tr><td><strong>persistent storage</strong></td><td><strong>1. workspace;<br>  2. instructions and skills;<br>  3. session/workflow state;<br>  4. reusable memory (mem0, letta, etc)</strong></td><td>project files plus recorded job progress</td></tr><tr><td><strong>external tool gateway / mcp servers</strong></td><td>source control, monitoring, business systems, communication, deployment, knowledge/data integrations</td><td>reads a github issue and creates a pr</td></tr><tr><td><strong>identity and credential service</strong></td><td>secrets, tokens, workload identities, scoped credentials</td><td>authenticates access to github and the model api</td></tr><tr><td><strong>policy and approval service</strong></td><td>authorization rules, action scopes, approval enforcement</td><td>checks whether the job may publish this change</td></tr></tbody></table></div>


### harness runtime and model inference

the harness builds the request from the user query, applicable project instructions, available skills, tool definitions, and conversation history. source files enter the context when loaded or read through tools.

for the checkout task, project instructions might require integer cents and specify the test command. the model can then request the issue details, inspect the relevant files, propose edits, and ask to run tests. the harness executes permitted requests and returns observations for the next model call.

the inference service generates those responses. it can be a hosted api or a model deployment you operate. hosting the harness does not require hosting the model on the same machine.

### execution environment and external tools

the execution environment is where file operations and commands actually happen. it contains the project directory, dependencies, utilities, and any running application processes. in this example, it edits `checkout.ts` and executes `npm test`.

external integrations give the agent access to systems beyond that environment. a github integration reads the issue and creates a pr. other categories include monitoring, business systems, communication, deployment, and knowledge services.

these integrations can be custom functions or mcp servers. they may run alongside the harness or behind a remote service. their location determines where authenticated requests happen and which network connections are required.

### persistent storage

storage holds four different kinds of data:


<div class="harness-table" role="region" aria-label="comparison table 2" tabindex="0"><table><thead><tr><th scope="col">category</th><th scope="col">example</th><th scope="col">what it preserves</th></tr></thead><tbody><tr><td>workspace</td><td><code>/agent-data/workspaces/job-482/</code></td><td>repository, edits, dependencies, outputs</td></tr><tr><td>instructions and skills</td><td>project <code>AGENTS.md</code> and <code>/agent-data/skills/</code></td><td>guidance the harness loads</td></tr><tr><td>session and workflow state</td><td>transcript files and a sqlite database (jsonl files)</td><td>conversation, tool results, job status, approvals</td></tr><tr><td>memory (mem0, letta, etc)</td><td><code>/agent-data/memory/checkout.md</code></td><td>reusable knowledge for later tasks</td></tr></tbody></table></div>


these can start on one persistent disk. workspace describes a directory, while instructions and memory describe content stored in files or other records. `AGENTS.md` can live inside the project; session and approval state can live in a database on the same disk.

sharing storage does not mean sharing write permissions. the agent may edit project files without being allowed to rewrite its approval records or another customer's session.

as workers scale, active workspaces can stay on worker disks, structured state can move to a shared database, and recovery copies can move to object storage. persistence must survive the failures the system promises to recover from.

### identity, credentials, policy, and approvals


<div class="harness-table" role="region" aria-label="comparison table 3" tabindex="0"><table><thead><tr><th scope="col">responsibility</th><th scope="col">question</th><th scope="col">example</th></tr></thead><tbody><tr><td>identity</td><td>who is acting?</td><td>job 482, belonging to acme</td></tr><tr><td>credential</td><td>how is access authenticated?</td><td>acme's github app token</td></tr><tr><td>policy</td><td>is this operation permitted?</td><td>read issues, but never merge prs</td></tr><tr><td>approval</td><td>is this exact action authorized?</td><td>user approved publishing this patch</td></tr></tbody></table></div>


a valid github token does not mean the proposed pr has been approved. approval does not provide the credential needed to execute it.

the integration should authenticate the job, check its scope, and use the appropriate credential. an approval should identify the repository, operation, and exact proposed change. changing the patch requires reevaluating the authorization.

this affects hosting: keeping credentials and approval enforcement outside the environment running project code creates a clearer boundary. a policy written only in the model's instructions cannot enforce that boundary.

## requirements that shape hosting

before choosing a layout, define durability, access, and resource requirements.

### durability


<div class="harness-table" role="region" aria-label="comparison table 4" tabindex="0"><table><thead><tr><th scope="col"></th><th scope="col">durable session</th><th scope="col">durable execution</th></tr></thead><tbody><tr><td>what is preserved?</td><td>conversation, tool results, session metadata</td><td>recorded execution progress, completed steps, pending waits, recovery information</td></tr><tr><td>what does it enable?</td><td>restart a harness and continue the conversation</td><td>recover the job and determine what should happen next</td></tr><tr><td>example</td><td>recover the issue details and previous observations</td><td>remain paused until the proposed pr is approved</td></tr><tr><td>what is not automatic?</td><td>restoration of workspace files or running processes</td><td>preservation of arbitrary ram, live shell processes, or exactly-once external effects</td></tr></tbody></table></div>


a session can be recoverable while its workspace is lost. a workflow can remember that a test started while the actual test process has died. these states need to be preserved or reconstructed together.


<div class="harness-table" role="region" aria-label="comparison table 5" tabindex="0"><table><thead><tr><th scope="col">crash happens</th><th scope="col">what recovery requires</th></tr></thead><tbody><tr><td>after reading the issue</td><td>saved session, or fetching the issue again</td></tr><tr><td>after editing <code>checkout.ts</code></td><td>persisted workspace or patch, plus enough execution state to continue correctly</td></tr><tr><td>halfway through <code>npm test</code></td><td>detect the interrupted attempt and usually restart the tests</td></tr><tr><td>after creating a pr but before recording success</td><td>inspect github before retrying the action</td></tr></tbody></table></div>


for the checkout agent, recovery therefore needs the conversation, the relevant workspace version, and enough execution state to decide what should happen next. external writes also need reconciliation or idempotency so retries do not blindly repeat an action.

### access and isolation

a sandbox defines what execution can access. it may use a vm, a container, or other isolation mechanisms. setting the working directory to `/workspace/shop` does not itself prevent access to other files or the network.

choose filesystem, network, identity, and resource boundaries around the task. the agent needs access to its repository and test dependencies; it does not automatically need production credentials or another job's files.

the restrictions must still allow the work to complete. browser testing, private package registries, and local databases can all change the required environment.

### compute

calling a hosted model api does not require a gpu on the harness machine. most additional resource demand comes from the execution workload.

these are illustrative starting allocations for one active agent, not benchmarks or guarantees:


<div class="harness-table" role="region" aria-label="comparison table 6" tabindex="0"><table><thead><tr><th scope="col">workload</th><th scope="col">cpu</th><th scope="col">ram</th><th scope="col">disk</th></tr></thead><tbody><tr><td>harness with remote tools</td><td>1–2 vcpu</td><td>1–2 gib</td><td>5–10 gib</td></tr><tr><td>harness plus modest repository and tests</td><td>2–4 vcpu</td><td>4–8 gib</td><td>20–50 gib</td></tr><tr><td>harness plus browser, builds, local services</td><td>4–8 vcpu</td><td>8–16+ gib</td><td>50–100+ gib</td></tr></tbody></table></div>


measure peak usage on representative tasks. browsers, builds, parallel tests, and dependency caches can dominate the harness itself. when execution is remote, those costs move to the execution host.

## hosting options

the main placement decision is whether the harness and execution environment share a machine. all four layouts can support durable sessions and execution, provided their state and recovery mechanisms survive worker failure.


<div class="harness-table" role="region" aria-label="comparison table 7" tabindex="0"><table><thead><tr><th scope="col">layout</th><th scope="col">how it works</th><th scope="col">main benefit</th><th scope="col">main cost</th></tr></thead><tbody><tr><td><strong>same vm</strong></td><td>harness reads local files and launches local commands</td><td>simple integration; compatible with local coding tools</td><td>harness and tools share host failure and resource pressure</td></tr><tr><td><strong>separate containers, same vm</strong></td><td>harness calls an execution container through an api</td><td>separate process lifecycles and resource limits</td><td>host failure remains shared; mounts and privileges need care</td></tr><tr><td><strong>harness service plus remote execution</strong></td><td>harness calls read/edit/exec endpoints on another environment</td><td>independent replacement, scaling, and access boundaries</td><td>remote tool adapters, network failures, process tracking</td></tr><tr><td><strong>coordinator plus complete agent workers</strong></td><td>each worker has its own harness and execution environment</td><td>natural fit for agents with their own computers</td><td>per-agent state, artifact exchange, conflicting edits, coordination</td></tr></tbody></table></div>


### harness and execution together

one vm is the most direct setup for a harness that expects a local shell and filesystem. the project, tools, and harness share an environment. a replacement worker restores the session and workspace before continuing.

separate containers on that vm add independent process lifecycles and resource limits. they can reduce interference, but they still share host failure. shared mounts and privileges determine how much isolation they actually provide.

### harness and execution separate

a dedicated harness service calls remote file and command interfaces. execution machines can be replaced or resized without moving the harness, and trusted credentials can stay outside them.

the trade-off is integration work. remote calls can fail ambiguously, long-running commands need identities and status tracking, and built-in local tools may need adapters. separating the processes does not automatically solve recovery.

### complete agent workers

for multiple agents with their own computers, a coordinator can assign tasks to workers that each contain a harness and execution environment.

this keeps local tooling intact and allows different worker environments. it also requires per-agent sessions, ownership of files or branches, artifact exchange, and a way to reconcile conflicting changes.

the distinction is whether the central service controls individual tools or delegates work to another agent.

### operating model and lifecycle

these layouts can run on vms you manage, a container platform, kubernetes, or managed execution infrastructure. a managed harness service also operates the loop, with its own limits on customization, tools, networking, and recovery.

choose the lifecycle separately from the placement:

- **always running:** fastest continuation, with idle compute cost.

- **per job:** create an environment for the task and stop it when complete.

- **stop and restore:** persist state during idle periods and reconstruct the session when needed.

- **warm pool:** keep prepared workers available to reduce startup time.

an overnight approval requires a durable pending decision and a way to resume work. it does not necessarily require keeping the vm alive overnight.

## how to choose

start with the task and the harness you intend to use.

- **choose one vm** when a job needs one development environment and the harness assumes local tools.

- **choose separate containers on one host** when independent resource limits and process lifecycles are enough.

- **separate harness and execution hosts** when workloads need different compute, independent replacement, or stronger access boundaries, and the harness supports remote tools.

- **use complete agent workers with a coordinator** when subtasks require independent agents and workspaces.

then check private network access, storage recovery, approval waits, concurrency, and total cost per completed task. a cheaper worker can be more expensive overall if it repeatedly loses work or cannot run the required tests.

before calling the system durable, kill a tool process, replace a worker, restore a workspace, and interrupt the response to an external write. verify both the recovered job state and the external outcome.

the limit of harness hosting is the boundary of its recovery and access guarantees. a service may resume a conversation without restoring files, or restore a sandbox without knowing whether github accepted the last request.

the question to ask is: “if this worker disappears immediately after an external action succeeds, what happens next?”

## appendix: end-to-end agent loop

time moves from top to bottom. the diagrams are consecutive parts of the checkout task. “tools” represents the configured execution and integration layer, wherever it is hosted.

**part 1: user query to first model call**


<div class="harness-diagram"><a href="/assets/images/harness-hosting/agent-loop-1.svg"><img src="/assets/images/harness-hosting/agent-loop-1.svg" alt="part 1: user query, instruction and skill loading, then the first model call" loading="lazy"></a></div>

[open diagram 1 at full size](/assets/images/harness-hosting/agent-loop-1.svg)



**part 2: tool calls to verified result**


<div class="harness-diagram"><a href="/assets/images/harness-hosting/agent-loop-2.svg"><img src="/assets/images/harness-hosting/agent-loop-2.svg" alt="part 2: permission checks, approvals, tool execution, durable state, and final result" loading="lazy"></a></div>

[open diagram 2 at full size](/assets/images/harness-hosting/agent-loop-2.svg)


the durable state writes represent recovery mechanisms the application must implement. a basic harness may save only a transcript. recording a pending action does not make it atomic with an external service.

a final model response is not proof that the task passed its checks. the application should evaluate actual test results and other acceptance evidence before marking the job complete.

