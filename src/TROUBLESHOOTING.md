# Troubleshooting Guide

Common problems and solutions when creating shortcuts.

## Import Failures

### "The shortcut cannot be opened"

**Cause**: The file is not properly signed or the XML is malformed.

**Solutions**:
1. Verify XML is well-formed:
   ```bash
   xmllint --noout MyShortcut.shortcut
   ```
2. Re-sign the shortcut:
   ```bash
   shortcuts sign --mode anyone --input MyShortcut.shortcut --output MyShortcut_signed.shortcut
   ```
3. Check for common XML errors:
   - Missing closing tags
   - Unescaped special characters (`<`, `>`, `&`)
   - Invalid UTF-8 characters

### "Signing failed" or signing command hangs

**Cause**: The plist structure is invalid or has type errors.

**Solutions**:
1. Verify all `<integer>` values are numbers, not strings
2. Check `WFControlFlowMode` uses `<integer>`, not `<string>`
3. Ensure all required keys are present in the root dict
4. Try converting to binary plist first:
   ```bash
   plutil -convert binary1 MyShortcut.shortcut
   ```

---

## Variable Problems

### Variable shows as empty

**Possible causes**:
1. **Wrong OutputUUID**: The UUID doesn't match any source action
2. **Wrong position**: The `attachmentsByRange` position is incorrect
3. **Missing UUID on source**: The source action has no UUID parameter

**Debugging steps**:
1. Find the `OutputUUID` value in the consuming action
2. Search for that UUID in the source action's `<key>UUID</key>`
3. If not found, add the UUID to the source action
4. Verify the position calculation is correct

### Variable shows wrong value

**Cause**: Position in `attachmentsByRange` is incorrect.

**Solution**: Recount characters manually:
```
String: "Result: ￼"
Count:   01234567 8

Position = 8 (count "Result: " = 8 characters)
```

Remember:
- Spaces count as characters
- Newlines count as 1 character each
- Previous placeholders count as 1 character each

### "Repeat Item" or "Repeat Index" not working

**Cause**: Missing UUID on the repeat start action.

**Solution**: Add a UUID to the repeat start action:
```xml
<dict>
    <key>WFWorkflowActionIdentifier</key>
    <string>is.workflow.actions.repeat.each</string>
    <key>WFWorkflowActionParameters</key>
    <dict>
        <key>UUID</key>
        <string>AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA</string>
        <key>GroupingIdentifier</key>
        <!-- ... -->
    </dict>
</dict>
```

Then reference that UUID with OutputName `Repeat Item` or `Repeat Index`.

---

## Control Flow Problems

### Repeat/If/Menu doesn't work correctly

**Possible causes**:
1. Missing end action (mode 2)
2. Mismatched GroupingIdentifiers
3. WFControlFlowMode is a string instead of integer

**Debugging steps**:
1. Find the start action (mode 0)
2. Find the end action (mode 2)
3. Verify both have the **exact same** GroupingIdentifier
4. Verify mode values use `<integer>` tags:
   ```xml
   <!-- CORRECT -->
   <key>WFControlFlowMode</key>
   <integer>0</integer>

   <!-- WRONG - will fail silently -->
   <key>WFControlFlowMode</key>
   <string>0</string>
   ```

### Menu options don't match

**Cause**: `WFMenuItemTitle` in case actions doesn't match items in `WFMenuItems` array.

**Solution**: Ensure exact string match:
```xml
<!-- Start action -->
<key>WFMenuItems</key>
<array>
    <string>Option A</string>
    <string>Option B</string>
</array>

<!-- Case action - must match exactly -->
<key>WFMenuItemTitle</key>
<string>Option A</string>
```

---

## Action Problems

### Action does nothing

**Possible causes**:
1. Wrong action identifier (typo or wrong mapping)
2. Missing required parameters
3. Wrong parameter names

**Common identifier mistakes**:
| Wrong | Correct |
|-------|---------|
| `is.workflow.actions.if` | `is.workflow.actions.conditional` |
| `is.workflow.actions.repeat` | `is.workflow.actions.repeat.count` |
| `is.workflow.actions.foreach` | `is.workflow.actions.repeat.each` |

### Delete Photos doesn't delete

**Cause**: Using wrong parameter name.

**Solution**: Use lowercase `photos`, not `WFInput`:
```xml
<!-- CORRECT -->
<key>photos</key>
<dict>
    <key>Value</key>
    <!-- variable reference -->
</dict>

<!-- WRONG - common mistake -->
<key>WFInput</key>
```

---

## Text and String Problems

### Special characters cause XML errors

**Cause**: Unescaped XML special characters in strings.

**Solution**: Escape these characters:
| Character | Escape |
|-----------|--------|
| `<` | `&lt;` |
| `>` | `&gt;` |
| `&` | `&amp;` |
| `"` | `&quot;` (in attributes) |
| `'` | `&apos;` (in attributes) |

### Newlines not appearing correctly

**Cause**: Using `\n` literal instead of actual newline.

**Solution**: Use actual newlines in the string:
```xml
<!-- CORRECT -->
<string>Line 1
Line 2</string>

<!-- WRONG - shows as literal \n -->
<string>Line 1\nLine 2</string>
```

---

## Debugging Tips

### 1. Test incrementally
Build the shortcut in stages:
1. First create a simple version that shows a static result
2. Add one action at a time
3. Test after each addition

### 2. Use Show Result for debugging
Add `is.workflow.actions.showresult` actions to display intermediate values:
```xml
<dict>
    <key>WFWorkflowActionIdentifier</key>
    <string>is.workflow.actions.showresult</string>
    <key>WFWorkflowActionParameters</key>
    <dict>
        <key>Text</key>
        <string>Debug: Got to this point</string>
    </dict>
</dict>
```

### 3. Compare with working examples
If something isn't working:
1. Find a similar working example in EXAMPLES.md
2. Compare structure line by line
3. Look for differences in:
   - Parameter names
   - Serialization types
   - UUID references

### 4. Check the Shortcuts app directly
1. Import the shortcut
2. Open it in Shortcuts.app
3. Look for red warning indicators on actions
4. Check if variables show correct names in the UI
