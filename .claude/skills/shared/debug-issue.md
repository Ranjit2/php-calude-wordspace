# Skill: Debug Issue (Universal Methodology)
# For framework-specific commands see skills/{framework}/debug-issue.md

## Methodology — do not skip steps
1. Read the FULL stack trace — start at the originating exception line, not the top
2. Identify the error category — what type of failure is this?
3. State ONE specific hypothesis — not a list of possibilities
4. Find evidence that confirms or refutes it
5. Fix the root cause — not a try/catch that hides it
6. Write a regression test

## Output format
```
## Error
[Exception type and full message]

## Root cause
[One paragraph — what caused this and why]

## Evidence
[Specific line from trace, log entry, or query result that confirms it]

## Fix
[The exact code change needed]

## Regression test
[Test that would have caught this]
```

## Universal common bugs
| Symptom | Root cause | Fix |
|---|---|---|
| Works locally, fails in prod | Missing environment variable | Check all required keys are set on server |
| JSON decode returns null | Malformed JSON | Use `JSON_THROW_ON_ERROR` flag |
| Duplicate processing | No uniqueness guard | Add unique lock by ID before processing |
| Slow page, timeouts | N+1 queries | Eager-load relationships |
| File not found after upload | Path helper mismatch | Use consistent storage abstraction |
| Race condition in queue | Concurrent workers | Use atomic locking on job by resource ID |
