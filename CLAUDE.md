# SkillForge

## Setup

- **Backend**: `docker compose up -d` before running
- **Package manager**: use `bun`, not npm/yarn

## Before committing

```
bun run lint && bun run typecheck && bun run test
dotnet format && dotnet test
```

## Code Quality

- Write **human-readable code**: intention-revealing names, small focused functions, no clever tricks.
- **DRY and SOLID apply strictly to production code.** Tests may duplicate setup/data/assertions — never sacrifice test clarity for abstraction.
- Prefer **explicit over implicit**: no magic, no hidden conventions, no surprising side effects.
- Keep the **domain model pure**: no framework annotations, no ORM attributes, no HTTP concerns inside domain or application layers.

## Guidelines

@.claude/architecture.md
@.claude/tdd.md
@.claude/frontend.md
