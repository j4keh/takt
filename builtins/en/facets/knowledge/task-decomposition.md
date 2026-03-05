# Task Decomposition Knowledge

## Decomposition Feasibility

Before splitting a task into multiple parts, assess whether decomposition is appropriate. When decomposition is unsuitable, implementing in a single part is more efficient.

| Criteria | Judgment |
|----------|----------|
| Changed files clearly separate into layers | Decompose |
| Shared types/IDs span multiple parts | Single part |
| Broad rename/refactoring | Single part |
| Fewer than 5 files to change | Single part |
| Same file needs editing by multiple parts | Single part |

### Detecting Cross-Cutting Concerns

When any of the following apply, independent parts cannot maintain consistency. Consolidate into a single part.

- A new ID, key, or type is generated in one module and consumed in another
- Both the event emitter and event receiver need changes
- An existing interface signature changes, requiring updates to all call sites

## File Exclusivity Principle

When decomposing into multiple parts, each part's file ownership must be completely exclusive.

| Criteria | Judgment |
|----------|----------|
| Same file edited by multiple parts | REJECT (causes conflicts) |
| Type definition and consumer in different parts | Consolidate into the type definition part |
| Test file and implementation file in different parts | Consolidate into the same part |

### Grouping Priority

1. **By dependency direction** — keep dependency source and target in the same part
2. **By layer** — domain layer / infrastructure layer / API layer
3. **By feature** — independent functional units

## Failure Patterns

### Part Overlap

When two parts own the same file or feature, sub-agents overwrite each other's changes, causing repeated REJECT in reviews.

```
// NG: part-2 and part-3 own the same file
part-2: taskInstructionActions.ts — instruct confirmation dialog
part-3: taskInstructionActions.ts — requeue confirmation dialog

// OK: consolidate into one part
part-1: taskInstructionActions.ts — both instruct/requeue confirmation dialogs
```

### Shared Contract Mismatch

When part A generates an ID that part B consumes, both parts implement independently, leading to mismatches in ID name, type, or passing mechanism.

```
// NG: shared contract across independent parts
part-1: generates phaseExecutionId
part-2: consumes phaseExecutionId
→ part-1 uses string, part-2 expects number → integration error

// OK: single part for consistent implementation
part-1: implements phaseExecutionId from generation to consumption
```
