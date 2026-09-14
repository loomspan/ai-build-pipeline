# AI Build Pipeline

Work with your AI agent to produce a solid ticket, then automate the development of it.

This repository provides a development process expressed as Markdown command files. You stay closely involved in understanding the problem, choosing the intended outcome, and setting constraints. Once that intent is captured in a self-contained ticket, an AI agent chooses a proportionate execution profile and orchestrates the required work.

The pipeline makes ordinary technical decisions from the ticket and repository evidence. It brings you back in when a material decision cannot be resolved from that evidence. A project-specific **design lens** carries durable decisions into planning and review so the work fits your project.

## Getting started

Copy the entire [`ai/`](ai/) directory into the root of your project. Keep its directory structure: the commands refer to shared instructions and artifacts by these paths.

Use an AI coding agent that can read and edit repository files, run your project's build and test commands, and launch subagents in fresh contexts. The full profile also requires continuing the planning agent into test planning in the same context. These files are instructions executed by your agent; this repository does not include a separate executable runner. How you attach or reference files depends on your agent.

Before your first ticket, work with your agent to customize [`ai/thoughts/design-lens.md`](ai/thoughts/design-lens.md). It is fine to leave it empty when there are no durable project-specific decisions to record.

### Session 1: Develop the ticket together

Start a fresh session and tell the agent you are going to work on a new ticket. Spend time understanding the problem, the intended solution, and any directions or guardrails the work needs. Resolve consequential choices about behavior, scope, and acceptance criteria here.

When you are ready, ask:

```text
Use ai/commands/write_ticket.md to write a ticket from this conversation.
```

[`write_ticket.md`](ai/commands/write_ticket.md) captures the outcome, requirements, observable acceptance criteria, and an advisory full, fast-track, or direct execution-profile recommendation under `ai/thoughts/tickets/`. It preserves settled decisions without requiring the next session to recover them from chat. It also distinguishes binding constraints from suggestions that planning can reconsider.

Read the resulting ticket. It should express what you agreed to build and why, with enough context for an agent that has never seen the conversation. Generating the ticket does not start implementation.

### Session 2: Run the pipeline

Open a new session in the project and supply the ticket:

```text
Use ai/commands/0_run_pipeline.md to implement ticket
@ai/thoughts/tickets/2026-09-06-example-feature.md
```

Replace the example with your actual ticket path, using your agent's file-reference syntax. The orchestrator first performs a bounded, read-only current-checkout triage. By default it recommends a profile and waits for your choice before starting expensive work. You may explicitly request `full`, `fast-track`, or `direct` to skip that confirmation when triage finds the choice safe.

- **Full 5-Step Pipeline** (`full`) runs research, implementation planning, test planning, implementation and verification, and independent review.
- **Fast-Track 2-Step Pipeline — Implementation & Review** (`fast-track`) uses the ticket for targeted implementation and verification, then performs an independent review.
- **Direct Implementation — No Independent Review** (`direct`) uses the ticket for implementation and proportionate verification without a separate review.

Recommendations and confirmation prompts use these descriptive labels. Step
counts exclude Step 0 triage; the full route counts implementation planning and
test planning separately even though they share a context. Command values remain
`full`, `fast-track`, and `direct`.

The orchestrator pauses and recommends an upgrade when new evidence makes a
lighter route unsuitable, including during review fixes. It proceeds under
the upgraded route after you choose it, without redundant confirmation. A full
upgrade runs research and planning before returning to implementation; a
direct-to-fast-track upgrade adds independent review. Existing work is
preserved. It never silently downgrades an explicitly requested profile.

At the end, you receive the selected profile, triage rationale, artifact paths, review disposition when applicable, developer decisions, verification results, and optional developer checks. Full and fast-track completion means a fresh reviewer found no actionable issues and performed sufficient verification. Direct completion is explicitly reported as having no independent review. Committing, opening a pull request, and deployment are outside the defined pipeline stages.

## How the pipeline works

Step 0 is a bounded profile gate, not a research stage. Beyond the ticket,
required policies, and Git summaries, it uses at most two locator searches and
five targeted file reads. If material uncertainty remains, it recommends full
instead of expanding the investigation. It makes no edits and runs no tests.
An explicit full request needs no extra research to justify the choice.

The shared protocol owns the profile eligibility table; the orchestrator owns
selection. Triage includes outstanding verification and review of ticket changes
already in the checkout, so a trivial last edit cannot make a substantial
unreviewed change eligible for direct. Unrelated dirty developer changes are
excluded and preserved. Legacy tickets without a recommendation remain valid.

| Stage | Responsibility | Result |
| --- | --- | --- |
| [1. Research](ai/commands/1_research_codebase.md) | Trace existing behavior, relevant code, tests, dependencies, and history. | Evidence-backed research document. |
| [2. Implementation plan](ai/commands/2_create_plan.md) | Choose an approach, apply project guardrails, assess impacts, and map acceptance criteria to code and tests. | Concrete implementation plan with phases and verification. |
| [3. Testing plan](ai/commands/3_testing_plan.md) | Define regression coverage, failing tests where applicable, safe commands, and exit criteria. | Testing plan grounded in the project's existing tools. |
| [4. Implementation](ai/commands/4_implement_plan.md) | Implement the plans, or a ticket-led fast/direct change, and run verification. | Code, tests, supporting changes, and verification results. |
| [5. Independent review](ai/commands/5_code_review.md) | Reconstruct the change, check correctness and requirements, fix actionable issues, and verify again. | Numbered review document and disposition. |

In the full profile, research, planning, implementation, and each independent review begin in fresh contexts. Implementation planning and test planning deliberately share one context so the test strategy builds on the design decisions and risks just established. Fast track starts implementation and review in separate fresh contexts; direct runs only implementation.

**Artifact files carry knowledge across context boundaries.** Important decisions
belong in the ticket, artifacts produced by the selected route, or the
implementation. Ticket-led execution uses a concise `Execution notes` section
in the existing ticket for material decisions, developer answers, scope
attribution, and observations needed later. Checklists stay internal; no
substitute plans are required. Notes preserve context without authorizing a
future run or supplying prior review conclusions. Chat summaries are receipts.
The [orchestrator](ai/commands/0_run_pipeline.md) checks that artifacts required
by the selected route exist and contain substantive content before advancing.

### Full and fast-track review continues until a fresh context is clean

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

Planning and ticket-led implementation read the lens directly. Test planning
and plan-led implementation carry its decisions forward from the plan.
Independent review reads the lens again and checks conformance. Research
documents the current codebase; it does not currently have an explicit
instruction to apply the lens.

Keep the lens current as the project evolves. Explain the scope and exceptions of each decision so agents can exercise judgment, and revisit entries when experience changes the project's needs. If you change the lens's filename or location, update the command references too.

## Artifacts you can inspect

The full profile writes its outputs under `ai/thoughts/`, using the ticket filename stem to keep related work together:

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

The research records source evidence and repository state. The plans connect
acceptance criteria to implementation and executable verification. Review
documents record findings, resolved issues, conformance, verification, and
residual risks. Fast-track adds review artifacts and direct adds no separate
process document; both preserve material execution context in the existing
ticket. Final reports identify the actual verification source: the final fresh
review for full/fast-track, or Step 4 for direct. Stopped runs identify their
last stage and any pending profile choice.

## Origins and attribution

This process began with the ideas and portions of three command files from [HumanLayer](https://github.com/humanlayer/humanlayer):

- [`research_codebase.md`](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/research_codebase.md)
- [`create_plan.md`](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/create_plan.md)
- [`implement_plan.md`](https://github.com/humanlayer/humanlayer/blob/main/.claude/commands/implement_plan.md)

Their separation of research, planning, and implementation provided the foundation. This project's author substantially reworked those commands and developed the surrounding process: conversational ticket creation, automated orchestration, shared handoffs and escalation, dedicated test planning, independent review and fix cycles, and project-specific design guidance.

## License

See [LICENSE](LICENSE) for the Apache License 2.0 text.
