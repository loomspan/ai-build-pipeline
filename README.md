# AI Build Pipeline

Work with your AI agent to produce a solid ticket, then automate the development of it.

This repository provides a development process expressed as Markdown command files. You stay closely involved in understanding the problem, choosing the intended outcome, and setting constraints. Once that intent is captured in a self-contained ticket, an AI agent orchestrates research, implementation planning, test planning, implementation, and independent review.

The pipeline makes ordinary technical decisions from the ticket and repository evidence. It brings you back in when a material decision cannot be resolved from that evidence. A project-specific **design lens** carries durable decisions into planning and review so the work fits your project.

## Getting started

Copy the entire [`ai/`](ai/) directory into the root of your project. Keep its directory structure: the commands refer to shared instructions and artifacts by these paths.

Use an AI coding agent that can read and edit repository files, run your project's build and test commands, and launch subagents in fresh contexts. The automated pipeline also requires continuing the planning agent into test planning in the same context. These files are instructions executed by your agent; this repository does not include a separate executable runner. How you attach or reference files depends on your agent.

Before your first ticket, work with your agent to customize [`ai/thoughts/design-lens.md`](ai/thoughts/design-lens.md). It is fine to leave it empty when there are no durable project-specific decisions to record.

### Session 1: Develop the ticket together

Start a fresh session and tell the agent you are going to work on a new ticket. Spend time understanding the problem, the intended solution, and any directions or guardrails the work needs. Resolve consequential choices about behavior, scope, and acceptance criteria here.

When you are ready, ask:

```text
Use ai/commands/write_ticket.md to write a ticket from this conversation.
```

[`write_ticket.md`](ai/commands/write_ticket.md) captures the outcome, requirements, and observable acceptance criteria under `ai/thoughts/tickets/`. It preserves settled decisions without requiring the next session to recover them from chat. It also distinguishes binding constraints from suggestions that planning can reconsider.

Read the resulting ticket. It should express what you agreed to build and why, with enough context for an agent that has never seen the conversation. Generating the ticket does not start implementation.

### Session 2: Run the pipeline

Open a new session in the project and supply the ticket:

```text
Use ai/commands/0_run_pipeline.md to implement ticket
@ai/thoughts/tickets/2026-09-06-example-feature.md
```

Replace the example with your actual ticket path, using your agent's file-reference syntax. The orchestrator runs the stages and passes artifact paths between agents. Answer any material questions it raises; otherwise, it continues through implementation and review.

At the end, you receive the artifact paths, review disposition and count, developer decisions, verification results, and any optional developer checks. Completion means a fresh reviewer found no actionable issues and performed sufficient verification. Committing, opening a pull request, and deployment are outside the defined pipeline stages.

## How the pipeline works

| Stage | Responsibility | Result |
| --- | --- | --- |
| [1. Research](ai/commands/1_research_codebase.md) | Trace existing behavior, relevant code, tests, dependencies, and history. | Evidence-backed research document. |
| [2. Implementation plan](ai/commands/2_create_plan.md) | Choose an approach, apply project guardrails, assess impacts, and map acceptance criteria to code and tests. | Concrete implementation plan with phases and verification. |
| [3. Testing plan](ai/commands/3_testing_plan.md) | Define regression coverage, failing tests where applicable, safe commands, and exit criteria. | Testing plan grounded in the project's existing tools. |
| [4. Implementation](ai/commands/4_implement_plan.md) | Implement the plans, run verification, and update completion checkboxes using actual evidence. | Code, tests, supporting changes, and verification results. |
| [5. Independent review](ai/commands/5_code_review.md) | Reconstruct the change, check correctness and requirements, fix actionable issues, and verify again. | Numbered review document and disposition. |

Research, planning, implementation, and each independent review begin in fresh contexts. Implementation planning and test planning deliberately share one context so the test strategy builds on the design decisions and risks just established.

**Artifact files carry knowledge across context boundaries.** Important decisions belong in the research, plans, ticket, or implementation. Chat summaries are short receipts. The [orchestrator](ai/commands/0_run_pipeline.md) checks that required artifacts exist and contain substantive content before advancing.

### Review continues until a fresh context is clean

A review agent completes its initial review before editing. If it finds actionable issues, it applies safe fixes within scope and re-reviews until it believes the work is clean. Because it changed the work, it returns `fixes-applied`, and the orchestrator starts another fresh reviewer.

Only a reviewer that makes no implementation-artifact changes, finds no actionable issues of any priority, and completes sufficient verification can return `clean`. Each fresh reviewer examines the current repository without consuming prior review documents.

There is no fixed review-cycle limit. Repeated defects or materially identical verification failures across two consecutive fresh cycles without meaningful progress trigger a developer question instead of an indefinite loop.

### Where you stay involved

The [shared automation protocol](ai/commands/shared/automation-protocol.md) defines three step outcomes: `complete`, `needs-developer`, and `failed`. A material ambiguity about behavior, scope, risk, or correctness comes back with evidence and a recommended answer. Routine choices such as naming or helper placement remain with the agent.

Verification uses the project's actual build and test tools. Routine checks must avoid unintended live or destructive operations. Optional manual or configured-environment checks are reported separately and do not block completion; required acceptance criteria still need sufficient evidence.

The numbered stage commands can also be used individually. They default to standalone mode unless explicitly invoked in pipeline mode. Standalone planning supports developer feedback, and standalone review does not apply implementation fixes unless requested.

## Make it project-specific with the design lens

The [design lens](ai/thoughts/design-lens.md) preserves durable, non-obvious decisions that apply across tickets. Its value is capturing something a capable agent could reasonably get wrong even after reading the source and tests.

For example, the Loomspan framework's feature design lens records that technical exposure does not automatically establish a supported compatibility contract. It also distinguishes ordinary business inputs from trusted execution metadata: the runtime owns authenticated identity and authorization information, and the model must not be able to override them.

Those decisions change how features should be designed and reviewed. A public constructor alone cannot settle a compatibility question, and matching input names cannot establish permission to propagate trusted identity.

Here is a compact example adapted from the Loomspan lens into this repository's entry format:

```markdown
## Runtime-owned execution identity

- **Decision:** Authenticated identity, authorization claims, and trusted tenant
  identity come from authoritative runtime sources and must not be
  model-overridable skill inputs.
- **Why this is non-obvious:** Passing identity alongside ordinary business
  inputs can look convenient while giving the model control over trusted data.
- **Applies to:** Skill invocation, nested calls, and tool execution that use
  trusted execution identity.
- **Exceptions:** None.
- **Established by:** Loomspan framework feature design lens, principles on
  business input and trusted execution metadata.
```

This is an example to adapt, not an active policy shipped with the pipeline.

To develop your own lens, ask your agent:

```text
Help me customize ai/thoughts/design-lens.md for this project. Identify durable,
cross-cutting decisions that could not reliably be reconstructed from code and
tests alone. Discuss proposed entries with me, including their rationale, scope,
exceptions, and source, before recording them.
```

Keep entries specific and explain why they matter. Good candidates include who may authorize AI-initiated actions, which data may reach a model, what establishes a supported contract, or what provenance an AI-derived value must retain. General advice such as "write tests" and architecture summaries already visible in the source belong elsewhere.

| Where a decision belongs | Use it for |
| --- | --- |
| Design lens | Durable project decisions that apply across tickets. |
| Ticket requirements | Behavior and constraints required for this particular outcome. |
| Ticket `Pipeline notes` | A narrow intentional exception or constraint a later stage might otherwise misunderstand. |
| Research and plans | Discovered evidence, implementation choices, risks, and verification details. |

Planning reads the lens directly and records applicable guardrails. Test planning and implementation carry them forward from the plan. Independent review reads the lens again and checks conformance. Research documents the current codebase; it does not currently have an explicit instruction to apply the lens.

Keep the lens current as the project evolves. Explain the scope and exceptions of each decision so agents can exercise judgment, and revisit entries when experience changes the project's needs. If you change the lens's filename or location, update the command references too.

## Artifacts you can inspect

The commands write their outputs under `ai/thoughts/`, using the ticket filename stem to keep related work together:

```text
ai/thoughts/
  design-lens.md
  tickets/<ticket-stem>.md
  research/<ticket-stem>.md
  plans/<ticket-stem>.md
  plans/<ticket-stem>-testing.md
  reviews/<ticket-stem>-review-1.md
  reviews/<ticket-stem>-review-2.md
```

The research records source evidence and repository state. The plans connect acceptance criteria to implementation and executable verification. Review documents record findings, resolved issues, conformance, verification, and residual risks. These files make the development process inspectable after the agent session ends.

## Origins and attribution

This process began with the ideas and portions of three command files from [HumanLayer](https://github.com/humanlayer/humanlayer):

- [`research_codebase.md`](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/research_codebase.md)
- [`create_plan.md`](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/create_plan.md)
- [`implement_plan.md`](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/implement_plan.md)

Their separation of research, planning, and implementation provided the foundation. This project's author substantially reworked those commands and developed the surrounding process: conversational ticket creation, automated orchestration, shared handoffs and escalation, dedicated test planning, independent review and fix cycles, and project-specific design guidance.

## License

See [LICENSE](LICENSE) for the Apache License 2.0 text.
