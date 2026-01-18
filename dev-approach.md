# OpenCode Development Approach

This document explains the development methodology used in the OpenCode codebase.

## TL;DR

**No, this codebase is not using traditional spec-driven development.**

OpenCode follows a **code-driven API development** approach combined with **feature-design driven development**.

---

## Code-Driven API Development

The project follows a **code-first** approach for APIs:

1. **Implementation First**: API routes are implemented in `packages/opencode/src/server/routes/` using `hono` with `describeRoute` decorators
2. **Generate OpenAPI**: Run `bun dev generate` to auto-generate `openapi.json` from implementation
3. **Generate SDK**: Run `./packages/sdk/js/script/build.ts` to generate TypeScript SDK from OpenAPI spec

### Evidence

- `script/generate.ts` runs `bun dev generate > ../sdk/openapi.json` - generating spec **from code**
- Route files use inline OpenAPI decorators (e.g., `describeRoute({ operationId: "project.list" })`)
- No pre-written API spec that drives implementation

### Example Route Definition

```typescript
// packages/opencode/src/server/routes/project.ts
export const ProjectRoutes = lazy(() =>
  new Hono()
    .get(
      "/",
      describeRoute({
        summary: "List all projects",
        description: "Get a list of projects that have been opened with OpenCode.",
        operationId: "project.list",
        responses: {
          200: {
            description: "List of projects",
            content: {
              "application/json": {
                schema: resolver(Project.Info.array()),
              },
            },
          },
        },
      }),
      async (c) => {
        const projects = await Project.list()
        return c.json(projects)
      },
    )
```

### Generation Workflow

```bash
# script/generate.ts
await $`bun ./packages/sdk/js/script/build.ts`
await $`bun dev generate > ../sdk/openapi.json`.cwd("packages/opencode")
await $`./script/format.ts`
```

---

## Feature-Design Driven Development

The `specs/` directory contains **RFC-style design documents**, not API specifications:

- `specs/project.md` - Design document for project API structure
- `specs/01-persist-payload-limits.md` - Detailed RFC for persistence strategy
- `specs/02-cache-eviction.md` - Design doc for cache eviction
- `specs/perf-roadmap.md` - Performance roadmap

These are **design documents** with:

- Goals, non-goals
- Current state analysis
- Proposed approach
- Phased implementation steps
- Risk mitigation strategies
- Validation and rollout plans

### Example Spec Structure

```markdown
## Persist Payload Limits

### Summary

Large payloads (base64 images, terminal buffers) are currently persisted inside key-value stores...

### Goals

- Stop persisting image `dataUrl` blobs inside web `localStorage`
- Stop persisting image `dataUrl` blobs inside desktop store `.dat` files
- ...

### Current state

- `packages/app/src/utils/persist.ts` uses `localStorage` (sync) on web...
- ...

### Proposed approach

1. Add per-key persistence policies (KV store guardrails)
2. Add a dedicated blob store for large data
   ...

### Phased implementation steps

1. Add guardrails in `persist.ts`
2. Add blob-store abstraction + platform hooks
3. Update prompt history + prompt draft persistence to use blob refs
   ...
```

---

## Development Workflow

```
1. Write Feature Design (specs/*.md)
   ↓
2. Implement API Routes (packages/opencode/src/server/routes/)
   ↓
3. Generate OpenAPI Spec (bun dev generate)
   ↓
4. Generate SDK from Spec (packages/sdk/js/script/build.ts)
   ↓
5. Clients use SDK (@opencode-ai/sdk)
```

### Key Commands

| Command                             | Purpose                                        |
| ----------------------------------- | ---------------------------------------------- |
| `bun dev generate`                  | Generate OpenAPI spec from implementation      |
| `./script/generate.ts`              | Full generation pipeline (spec + SDK + format) |
| `./packages/sdk/js/script/build.ts` | Generate SDK from OpenAPI spec                 |

---

## Contrast with Spec-Driven Development

| Aspect         | Spec-Driven                    | OpenCode Approach                     |
| -------------- | ------------------------------ | ------------------------------------- |
| API Spec       | Written first, source of truth | Generated from implementation         |
| Implementation | Derived from spec              | Written first, drives spec generation |
| SDK            | Generated from spec            | Generated from generated spec         |
| Change Flow    | Update spec → regenerate code  | Update code → regenerate spec/SDK     |

### Traditional Spec-Driven Flow

```
1. Write OpenAPI spec
   ↓
2. Generate SDK from spec
   ↓
3. Implement API to match spec
   ↓
4. Verify against spec
```

### OpenCode Flow

```
1. Write design doc (optional)
   ↓
2. Implement API routes
   ↓
3. Generate spec from implementation
   ↓
4. Generate SDK from spec
   ↓
5. Clients use SDK
```

---

## Tooling Stack

### OpenAPI Generation

- **Library**: `hono-openapi`
- **Decorators**: `describeRoute`, `validator`, `resolver`
- **Output**: `packages/sdk/openapi.json`

### SDK Generation

- **Library**: `@hey-api/openapi-ts`
- **Output**: `packages/sdk/js/src/v2/gen/`
- **Features**: TypeScript types, fetch client, auto-generated methods

---

## Benefits of This Approach

1. **Fast Iteration**: Developers can modify API routes without editing separate spec files
2. **Type Safety**: SDK is always in sync with implementation
3. **Single Source of Truth**: Route definitions in code are the canonical API contract
4. **Design Documentation**: Specs focus on design rationale, not API contracts
5. **Automation**: Full pipeline from implementation to usable SDK

---

## Conclusion

OpenCode uses a **pragmatic hybrid approach**:

- **Feature specifications** (design docs in `specs/`) drive feature development
- **Code** drives API contract (OpenAPI is generated from implementation)
- This allows rapid iteration while maintaining consistency through automated SDK generation

The project emphasizes practicality over strict spec-first development, which aligns with its "build and iterate" philosophy mentioned in contributing docs.

---

_Last Updated: 2026-01-18_
