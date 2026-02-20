**MINIONS INTERVIEWS — RESEARCH INTERVIEW SYSTEM**

You are tasked with creating the complete initial foundation for **minions-interviews** — structured qualitative research. This is part of the Minions ecosystem, a universal structured object system designed for building AI-native tools.

---

**PROJECT OVERVIEW**

`minions-interviews` is a structured system for conducting, transcribing, and analyzing research interviews. Different from `@minions-hiring/sdk` — this is for UX research, journalism, and academic interviews.

Agents can extract themes, surface key quotes, and synthesize insights across multiple interviews.

This project is built on the Minions SDK (`minions-sdk`), which provides the foundational primitives: Minion (structured object instance), MinionType (schema), and Relation (typed link between minions).

The system supports both TypeScript and Python SDKs with cross-language interoperability (both serialize to the same JSON format). All documentation includes dual-language code examples with tabbed interfaces.

---

**MINIONS SDK REFERENCE — REQUIRED DEPENDENCY**

This project depends on `minions-sdk`, a published package that provides the foundational primitives. The GH Agent building this project MUST install it from the public registries and use the APIs documented below — do NOT reimplement minions primitives from scratch.

**Installation:**
```bash
# TypeScript (npm)
npm install minions-sdk
# or: pnpm add minions-sdk

# Python (PyPI) — package name is minions-sdk, but you import as "minions"
pip install minions-sdk
```

**TypeScript SDK — Core Imports:**
```typescript
import {
  // Core types
  type Minion, type MinionType, type Relation,
  type FieldDefinition, type FieldValidation, type FieldType,
  type CreateMinionInput, type UpdateMinionInput, type CreateRelationInput,
  type MinionStatus, type MinionPriority, type RelationType,
  type ExecutionResult, type Executable,
  type ValidationError, type ValidationResult,

  // Validation
  validateField, validateFields,

  // Built-in Schemas (10 MinionType instances — reuse where applicable)
  noteType, linkType, fileType, contactType,
  agentType, teamType, thoughtType, promptTemplateType, testCaseType, taskType,
  builtinTypes,

  // Registry — stores and retrieves MinionTypes by id or slug
  TypeRegistry,

  // Relations — in-memory directed graph with traversal utilities
  RelationGraph,

  // Lifecycle — CRUD operations with validation
  createMinion, updateMinion, softDelete, hardDelete, restoreMinion,

  // Evolution — migrate minions when schemas change (preserves removed fields in _legacy)
  migrateMinion,

  // Utilities
  generateId, now, SPEC_VERSION,
} from 'minions-sdk';
```

**Python SDK — Core Imports:**
```python
from minions import (
    # Types
    Minion, MinionType, Relation, FieldDefinition, FieldValidation,
    CreateMinionInput, UpdateMinionInput, CreateRelationInput,
    ExecutionResult, Executable, ValidationError, ValidationResult,
    # Validation
    validate_field, validate_fields,
    # Built-in Schemas (10 types)
    note_type, link_type, file_type, contact_type,
    agent_type, team_type, thought_type, prompt_template_type,
    test_case_type, task_type, builtin_types,
    # Registry
    TypeRegistry,
    # Relations
    RelationGraph,
    # Lifecycle
    create_minion, update_minion, soft_delete, hard_delete, restore_minion,
    # Evolution
    migrate_minion,
    # Utilities
    generate_id, now, SPEC_VERSION,
)
```

**Key Concepts:**
- A `MinionType` defines a schema (list of `FieldDefinition`s) — each field has `name`, `type`, `label`, `required`, `defaultValue`, `options`, `validation`
- A `Minion` is an instance with `id`, `title`, `minionTypeId`, `fields` (dict), `status`, `tags`, timestamps
- A `Relation` is a typed directional link (12 types: `parent_of`, `depends_on`, `implements`, `relates_to`, `inspired_by`, `triggers`, `references`, `blocks`, `alternative_to`, `part_of`, `follows`, `integration_link`)
- Field types: `string`, `number`, `boolean`, `date`, `select`, `multi-select`, `url`, `email`, `textarea`, `tags`, `json`, `array`
- `TypeRegistry` auto-loads 10 built-in types; register custom types with `registry.register(myType)`
- `createMinion(input, type)` validates fields against the schema and returns `{ minion, validation }` (TS) or `(minion, validation)` tuple (Python)
- Both SDKs serialize to identical camelCase JSON; Python provides `to_dict()` / `from_dict()` for conversion

**IMPORTANT:** Do NOT recreate these primitives. Import them from `minions-sdk` (npm) / `minions` (PyPI). Build your domain-specific types and utilities ON TOP of the SDK.


---

**CORE PRIMITIVES**

This project defines the following Minion Types:

1. **`interview`** — A research session (Fields: `title` (string, required), `date` (date), `duration` (number), `interviewer` (string), `status` (select: scheduled/completed/analyzed))
2. **`participant`** — An interviewee (Fields: `name` (string, required), `demographics` (json), `consentGiven` (boolean), `notes` (textarea))
3. **`transcript`** — Interview text (Fields: `content` (textarea, required), `format` (select: verbatim/summary), `language` (string)) — Relations: `references` → interview
4. **`quote`** — A notable excerpt (Fields: `text` (textarea, required), `timestamp` (string), `significance` (select: low/medium/high)) — Relations: `part_of` → transcript, `references` → theme
5. **`theme`** — An emergent pattern (Fields: `name` (string, required), `description` (textarea), `frequency` (number), `confidence` (number, 0-1))
6. **`insight`** — A derived conclusion (Fields: `statement` (textarea, required), `supportingQuotes` (number), `actionable` (boolean)) — Relations: `depends_on` → themes

---

**BEYOND STANDARD PATTERN**

- Theme extraction from transcripts
- Quote surfacing by theme or keyword
- Cross-interview synthesis
- Participant anonymization utilities

---

**CLI COMMANDS**

```bash
interviews new "User research session 1"
interviews transcript import session1.txt
interviews themes <interview-id>
interviews quotes --theme "onboarding friction"
interviews synthesis --project <project-id>
```

---

**AGENT SKILL VALUE**

Agents can process interview transcripts, identify recurring themes, surface key quotes, and synthesize findings across multiple sessions.

---

**DUAL SDK REQUIREMENTS**

Critical cross-language compatibility requirements:

**Serialization Parity**
- Both TypeScript and Python SDKs must serialize minions to identical JSON format
- Field names, types, and structure must match exactly

**API Consistency**
- Same method names (adjusted for language conventions: TypeScript camelCase, Python snake_case)
- Same parameters and return types

**Documentation Parity**
- Every code example must have both TypeScript and Python versions
- Use Astro Starlight tabs: `<Tabs><TabItem label="TypeScript">...</TabItem><TabItem label="Python">...</TabItem></Tabs>`

**Testing Parity**
- Shared test fixtures (JSON files) that both SDKs can consume
- Identical test case coverage
- Cross-language integration tests (TypeScript SDK creates minion, Python SDK reads it)

---

**PROJECT STRUCTURE**

Standard Minions ecosystem monorepo structure:

```
minions-interviews/
  packages/
    core/                 # TypeScript core library
      src/
        types.ts
        index.ts
      test/
      package.json
    python/               # Python SDK
      minions_interviews/
        __init__.py
        types.py
      tests/
      pyproject.toml
    cli/                  # CLI tool
      src/
        commands/
        index.ts
      package.json
  apps/
    docs/                 # Astro Starlight documentation
  spec/
    v0.1.md
  examples/
    typescript/
    python/
  .github/
    workflows/
      ci.yml
      publish.yml
  README.md
  LICENSE                # AGPL-3.0
  package.json
```

---

**WHAT YOU NEED TO CREATE**

**1. THE SPECIFICATION** (`/spec/v0.1.md`)
Write a complete markdown specification covering all minion types, field schemas, relation patterns, and domain-specific algorithms.

**2. THE CORE LIBRARY** (`/packages/core`)
A framework-agnostic TypeScript library built on `minions-sdk`. Full type definitions, domain-specific classes, and utilities.

**3. THE PYTHON SDK** (`/packages/python`)
Complete Python port with identical functionality. Published to PyPI as `minions-interviews`.

**4. THE CLI** (`/packages/cli`)
All commands listed above, with interactive modes and JSON output support.

**5. THE DOCUMENTATION SITE** (`/apps/docs`)
Built with Astro Starlight. Landing page, getting started, concepts, API reference, guides — all with dual TypeScript/Python examples.

---

**DELIVERABLES**

Produce all files necessary to bootstrap this project completely. Every file should be production quality — not stubs, not placeholders.

1. Full specification (`/spec/v0.1.md`)
2. TypeScript core library (`/packages/core`)
3. Python SDK (`/packages/python`)
4. CLI tool (`/packages/cli`)
5. Documentation site (`/apps/docs`)
6. README with concrete examples
7. Examples in both TypeScript and Python
8. CI/CD setup (lint, test, publish for both languages)

---

**START SYSTEMATICALLY**

1. Write the specification first — nail down the schemas and domain algorithms
2. Implement TypeScript core library with full type definitions
3. Port to Python maintaining exact serialization compatibility
4. Build CLI using the core library
5. Write documentation with dual-language examples throughout
6. Create working examples

This is a foundational tool for the Minions ecosystem. Get it right.
