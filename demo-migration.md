# LLM Instructions: .NET Console Job Migration Guidelines

> **Purpose:** These instructions define **mandatory rules** for migrating **console jobs from .NET Framework 4.8 to .NET 8**.
> **Audience:** Any LLM or assistant helping with the migration.
> **Compliance:** Treat all **MUST / MUST NOT / PROHIBITED** statements as **hard requirements**.

---

## 1) Language Policy (MANDATORY)

### ⚠️ MANDATORY LANGUAGE REQUIREMENT
- **Language:** **C# exclusively** for all .NET development guidance.
- **Scope:** All migration instructions, examples, patterns, and snippets must be in **C# only**.

### 🚫 Prohibited
- **Do NOT** provide:
  - VB.NET instructions
  - F# instructions
  - VB.NET or F# code snippets
  - Conversions to VB.NET or F#

✅ **If asked for VB.NET/F#:** Refuse briefly and provide the C# equivalent only.

---

## 2) Console Job Migration Context

### Migration Scope
- Migrate existing **console jobs** from **.NET Framework 4.8** to **.NET 8**.
- Output must be suitable for **containerized execution** and aligned with **modern .NET 8 practices**.

---

## 3) Required Reading Materials (MANDATORY)

Before providing any migration plan, design, or implementation advice, you **MUST** read and follow instructions in the following documents:

### Core Migration Documents
- `src/instructions/sharedLibrary_References.md`
- `src/instructions/LLM_Migration_Instructions.md`
- `src/instructions/BATCH Project Architecture Guide.ini`

### Database Documentation
- **ERD Diagrams:** `src/docs/ERD_Diagrams.md`
- **Database DDL Files:**
  - `src/docs/ddl/AddInForNewCommit.txt`
  - `src/docs/ddl/BAIDB.sql`
  - `src/docs/ddl/ControlsDB.sql`
  - `src/docs/ddl/EmailerDB.sql`
  - `src/docs/ddl/InvestorRulesDB.sql`
  - `src/docs/ddl/OSCADB.sql`
  - `src/docs/ddl/UrsulaDB.sql`

✅ **Rule:** If you cannot access these materials in the conversation/workspace, you must:
- Ask the user to provide the required file contents or relevant excerpts, **OR**
- Respond: **"No relevant response can be found"** (only if the question depends on those documents and cannot be answered safely without them).

---

## 4) Architecture Requirements (MANDATORY)

### 🏗️ Clean Architecture Pattern (MANDATORY)
All migrated console jobs must follow this **exact** layer structure:

**Domain → Application → Infrastructure → ConsoleApp**

#### Layer Responsibilities (Strict)
- **Domain**
  - Core business entities, value objects, domain rules
  - No dependencies on other layers
- **Application**
  - Use cases, orchestration, interfaces (ports), DTOs as needed
  - Depends on **Domain** only
- **Infrastructure**
  - Implementations of interfaces: database access, file systems, external services
  - Depends on **Application** (and Domain as needed)
- **ConsoleApp**
  - Composition root: DI setup, configuration, command-line parsing, invocation
  - Depends on **Application + Infrastructure** (never the reverse)

✅ **Rule:** Dependency direction must always flow inward (ConsoleApp → ... → Domain).

---

## 5) Code Implementation Requirements

### Mandatory Migration Behavior
- Use code patterns and transformations specified in:
  - `src/prompts/migration-instructions.md`
  - `src/instructions/sharedLibrary_References.md`
  - `BATCH Project Architecture Guide.ini`

### Modern .NET 8 Patterns (Required)
All new or migrated code must embrace:
- **Dependency Injection (DI)**
- **async/await** for I/O and external operations
- **Options pattern** for configuration binding and validation

✅ **Expectations:**
- Avoid legacy patterns where modern configuration applies.
- Prefer `IHost` / generic host patterns for console job composition and lifecycle.
- Use cancellation tokens for long-running operations where appropriate.

---

## 6) ⚠️ Critical Self-Containment Requirements (MANDATORY)

### 🚫 PROHIBITED DEPENDENCIES
You **MUST NOT** reference any internal/proprietary packages, including (but not limited to):
- `SharedCommonFunctions`
- `WFMP.IS.Common`
- **Any other internal/proprietary packages**

### 📦 REQUIRED APPROACH: ZERO External Package Dependencies
- The migrated console job **must be self-contained**.
- All required functionality previously sourced from internal packages must be:
  - **Inlined**
  - Re-implemented as **custom implementations** inside the solution
  - Placed into appropriate Clean Architecture layers

✅ **Rule of thumb:**
- If a dependency is internal/proprietary: **Do not use it**.
- Replace it with local code that provides equivalent behavior.

---

## 7) Migration Objectives (Acceptance Criteria)

The final migrated application **MUST** satisfy all of the following:

### 1) Self-Containment
- **ZERO** external dependencies on internal/proprietary packages
- All needed functionality is implemented locally

### 2) Container Readiness
- Must run independently on **OCP (OpenShift Container Platform)** as a containerized workload
- Must support non-interactive execution and deterministic exit codes
- Must externalize configuration appropriately (e.g., environment variables / config files)

### 3) Architecture Compliance
- Must implement the **Clean Architecture** structure:
  - Domain → Application → Infrastructure → ConsoleApp

### 4) Modern .NET 8
- Uses modern patterns:
  - DI
  - async/await
  - Options pattern

---

## 8) Response Guidelines (How the LLM Must Behave)

### Assistance Rules
- Provide the best answer you can based on:
  1) The required reading materials (highest priority)
  2) The details supplied by the user (job name, legacy behavior, dependencies, DB usage, scheduling, etc.)
  3) General .NET 8 and Clean Architecture best practices (only when not contradicted by the migration documents)

### Conversation History Policy
- Use conversation history **only if needed** to answer when you cannot derive the answer independently.

### If You Cannot Answer
If the question cannot be answered using the available materials and inputs, respond **exactly** with:

> **"No relevant response can be found"**

✅ **Use this response especially when:**
- Required documents are missing and the answer depends on them
- Key technical details (job steps, DB objects, configuration) are not provided
- The requested approach would violate the policies above

---

## 9) Output Quality Bar (Expected Structure When Helping)

When responding to migration requests, format your guidance clearly and actionably. Prefer:

- **Assumptions / Inputs needed**
- **Target architecture mapping**
- **Step-by-step migration plan**
- **Key code responsibilities per layer**
- **Config + Options plan**
- **Dependency replacement plan** (especially for prohibited internal packages)
- **Testing/verification checklist**
- **Container readiness checklist**
- **Risks and mitigations**

✅ Keep everything in **C# terms**, even if not providing code.

---

## 10) STATUS
