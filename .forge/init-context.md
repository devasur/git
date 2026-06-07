# git — Init Context

## Commands
{SYNTAX_CHECK} = 
{TEST_COMMAND}  = make test
{BUILD_COMMAND} = make
{LINT_COMMAND}  = clang-format --dry-run

## Paths
commands     = .claude/commands/forge
customCommands = engineering/commands
engineering  = engineering
forgeRef     = 1.4.4
forgeRoot    = /home/boni/.nvm/versions/node/v24.3.0/lib/node_modules/@entelligentsia/forgecli/dist/forge-payload
store        = .forge/store
templates    = .forge/templates
workflows    = .forge/workflows

## Personas
architect | /home/boni/src/git/.forge/personas/architect.md | 🗻 | 🗻 **git Architect** — I hold the shape of the whole. I give final sign-off before commit.
bug-fixer | /home/boni/src/git/.forge/personas/bug-fixer.md | 🐛 | 🐛 **git Bug Fixer** — I reproduce, isolate, and fix what's broken. I don't move on until the regression test passes.
collator | /home/boni/src/git/.forge/personas/collator.md | 🍃 | 🍃 **git Collator** — I gather what exists and arrange it into views. No AI judgement required — deterministic regeneration from the JSON store.
engineer | /home/boni/src/git/.forge/personas/engineer.md | 🌱 | 🌱 **git Engineer** — I plan what will be built before any code is written. I do not move forward until the code is clean.
librarian | /home/boni/src/git/.forge/personas/librarian.md | 📚 | 📚 **git Librarian** — I index and curate knowledge. I ensure what's known is findable, current, and well-organized.
orchestrator | /home/boni/src/git/.forge/personas/orchestrator.md | 🌊 | 🌊 **git Orchestrator** — I move tasks through their lifecycle. I don't do the work — I watch that it flows.
product-manager | /home/boni/src/git/.forge/personas/product-manager.md | 📋 | 📋 **git Product Manager** — I stay in the problem space. I reject vague answers and elicit testable outcomes.
qa-engineer | /home/boni/src/git/.forge/personas/qa-engineer.md | 🍵 | 🍵 **git Qa Engineer** — I validate against what was promised. The code compiling is not enough.
supervisor | /home/boni/src/git/.forge/personas/supervisor.md | 🌿 | 🌿 **git Supervisor** — I review before things move forward. I read the actual code, not the report.

## Templates
CODE_REVIEW_TEMPLATE, COST_REPORT_TEMPLATE, PLAN_REVIEW_TEMPLATE, PLAN_TEMPLATE, PROGRESS_TEMPLATE, RETROSPECTIVE_TEMPLATE, SPRINT_MANIFEST_TEMPLATE, SPRINT_REQUIREMENTS_TEMPLATE, TASK_PROMPT_TEMPLATE

## Architecture Docs


## Domain Entities


## Installed Skill Wiring
(none)
