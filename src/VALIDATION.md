# Pre-Output Validation Checklist

**MANDATORY**: Complete this checklist before outputting any shortcut plist code.

## 1. UUID Validation

- [ ] All UUIDs are **UPPERCASE** (e.g., `AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA`)
- [ ] Every `OutputUUID` reference has a matching source action with that `UUID` parameter
- [ ] No duplicate UUIDs (each action must have a unique UUID)
- [ ] UUID format is correct: `XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX`

**Quick check**: Search for all `OutputUUID` values and verify each has a corresponding `<key>UUID</key>` in an earlier action.

## 2. Position Calculation Validation

For every `attachmentsByRange` entry:

- [ ] Count characters **manually** before each placeholder `￼`
- [ ] Include ALL characters: letters, spaces, punctuation, newlines
- [ ] For multiple placeholders, count previous placeholders as 1 character each
- [ ] Verify position matches the actual string content

**Calculation method**:
```
String: "Hello, ￼! Your age is ￼."
         0123456 7  8901234567890 21

First ￼:  "Hello, " = 7 chars → {7, 1}
Second ￼: "Hello, ￼! Your age is " = 7 + 1 + 14 = 22 chars → {22, 1}
```

**Common errors**:
- Off-by-one errors (forgetting the space after punctuation)
- Forgetting newlines count as characters
- Not counting previous placeholders

## 3. Control Flow Validation

For repeat, conditional, and menu actions:

- [ ] Every start action (mode 0) has a matching end action (mode 2)
- [ ] `GroupingIdentifier` is the **same UUID** for all actions in the block
- [ ] `WFControlFlowMode` uses `<integer>` not `<string>`
- [ ] Menu cases have correct `WFMenuItemTitle` matching `WFMenuItems` array

**Structure check**:
```
Repeat/If/Menu Start:  WFControlFlowMode = <integer>0</integer>
Middle (else/case):    WFControlFlowMode = <integer>1</integer>  (optional)
End:                   WFControlFlowMode = <integer>2</integer>  (required)
```

## 4. Parameter Serialization Validation

- [ ] Text with variables uses `WFTextTokenString` serialization
- [ ] Single variable references use `WFTextTokenAttachment` serialization
- [ ] Static text (no variables) can use simple `<string>` values
- [ ] The `string` key contains the placeholder character `￼` where variables go

**Decision tree**:
```
Does the parameter contain variables?
├─ No  → Use simple <string>value</string>
├─ Yes, just one variable with no text → Use WFTextTokenAttachment
└─ Yes, with surrounding text → Use WFTextTokenString with attachmentsByRange
```

## 5. Action Identifier Validation

- [ ] Action identifiers are spelled correctly (case-sensitive)
- [ ] Check irregular mappings (see ACTIONS.md):
  - `is.workflow.actions.repeat.count` (not `repeat`)
  - `is.workflow.actions.repeat.each` (not `foreach`)
  - `is.workflow.actions.conditional` (not `if`)
- [ ] Parameter names match action requirements:
  - Delete Photos uses `photos` (lowercase, not `WFInput`)
  - Show Result uses `Text` (capital T)

## 6. OutputName Validation

- [ ] `OutputName` matches the source action's output type:

| Source Action | OutputName |
|---------------|------------|
| Text (gettext) | `Text` |
| Ask for Input | `Provided Input` |
| Ask LLM | `Response` |
| Get Weather | `Weather Conditions` |
| URL | `URL` |
| Number | `Number` |
| Repeat Each | `Repeat Item` |
| Repeat Count | `Repeat Index` |

## 7. Plist Structure Validation

- [ ] XML declaration present: `<?xml version="1.0" encoding="UTF-8"?>`
- [ ] DOCTYPE present
- [ ] Root `<plist>` tag with `version="1.0"`
- [ ] `WFWorkflowActions` is an array
- [ ] Required metadata keys present:
  - `WFWorkflowClientVersion`
  - `WFWorkflowMinimumClientVersion`
  - `WFWorkflowName`
  - `WFWorkflowIcon`

## Final Check

Before outputting the code:

1. **Read through the entire plist** looking for typos
2. **Trace the data flow**: Can you follow each variable from source to destination?
3. **Count positions again**: Are all `attachmentsByRange` positions correct?
4. **Check control flow**: Does every start have an end?

---

## Quick Reference: Common Bugs

| Bug | Symptom | Fix |
|-----|---------|-----|
| Wrong position | Variable shows wrong value or empty | Recount characters manually |
| Missing UUID | "No output" or action ignored | Add UUID to source action |
| Lowercase UUID | Import fails or variable missing | Convert to UPPERCASE |
| String mode value | Control flow broken | Use `<integer>` not `<string>` |
| Wrong OutputName | Empty variable | Check OutputName table |
| Missing end action | Shortcut crashes or loops | Add mode 2 action |
