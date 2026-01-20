# Testing Procedures

Step-by-step procedures to validate shortcuts before delivery.

## Quick Test Checklist

For every generated shortcut:

- [ ] XML validates without errors
- [ ] Signing completes successfully
- [ ] Import into Shortcuts app works
- [ ] Shortcut runs without errors
- [ ] All variables display correct values
- [ ] Control flow behaves as expected

---

## Step 1: XML Validation

Verify the plist XML is well-formed before signing.

```bash
# Check XML syntax
xmllint --noout MyShortcut.shortcut

# If no output, the XML is valid
# If errors, fix them before proceeding
```

**Common XML errors**:
- Missing closing tags
- Unmatched angle brackets
- Invalid UTF-8 characters
- Unescaped special characters

---

## Step 2: Plist Validation

Verify the plist structure is valid.

```bash
# Validate plist format
plutil -lint MyShortcut.shortcut

# Expected output: "MyShortcut.shortcut: OK"
```

**If plutil fails**:
- Check that all keys have values
- Verify integer/string types are correct
- Look for missing dict/array closures

---

## Step 3: Signing

Sign the shortcut for import.

```bash
# Sign for anyone (recommended for testing)
shortcuts sign --mode anyone \
    --input MyShortcut.shortcut \
    --output MyShortcut_signed.shortcut

# Check exit code
echo "Exit code: $?"
# 0 = success, non-zero = failure
```

**Signing modes**:
- `anyone` - Anyone can use (recommended)
- `people-who-know-me` - Only contacts

**If signing fails**:
1. Run `plutil -lint` to check structure
2. Verify `WFControlFlowMode` uses `<integer>` not `<string>`
3. Check all required metadata keys exist

---

## Step 4: Import Test

Import the signed shortcut.

```bash
# Open the signed file (triggers import)
open MyShortcut_signed.shortcut
```

**Expected behavior**:
- Shortcuts app opens
- Import dialog appears
- "Add Shortcut" button is enabled

**If import fails**:
- "Cannot open" → Re-sign the file
- No dialog → File may not be recognized as shortcut

---

## Step 5: Functional Testing

Test the shortcut functionality in Shortcuts.app.

### 5.1: Visual Inspection
1. Open the imported shortcut in edit mode
2. Check for red warning icons (⚠️) on actions
3. Verify variable pills show correct names
4. Confirm control flow structure looks correct

### 5.2: Run Test
1. Click the play button (▶️) to run
2. Provide test inputs when prompted
3. Verify output matches expectations

### 5.3: Variable Flow Test
For shortcuts with variables:
1. Add temporary "Show Result" actions after each variable-producing step
2. Run and verify each shows the expected value
3. Remove debug actions when confirmed working

---

## Step 6: Edge Case Testing

Test boundary conditions:

### Input Edge Cases
- Empty input (if applicable)
- Very long input
- Special characters in input
- Unicode characters

### Control Flow Edge Cases
- For loops: Test with 0 items, 1 item, many items
- Conditionals: Test both true and false paths
- Menus: Test each menu option

---

## Automated Test Script

Save this as `test-shortcut.sh`:

```bash
#!/bin/bash

FILE="$1"

if [ -z "$FILE" ]; then
    echo "Usage: test-shortcut.sh <shortcut.shortcut>"
    exit 1
fi

echo "=== Testing: $FILE ==="

# Step 1: XML validation
echo -n "XML validation... "
if xmllint --noout "$FILE" 2>/dev/null; then
    echo "PASS"
else
    echo "FAIL"
    xmllint --noout "$FILE"
    exit 1
fi

# Step 2: Plist validation
echo -n "Plist validation... "
if plutil -lint "$FILE" >/dev/null 2>&1; then
    echo "PASS"
else
    echo "FAIL"
    plutil -lint "$FILE"
    exit 1
fi

# Step 3: Signing
SIGNED="${FILE%.shortcut}_signed.shortcut"
echo -n "Signing... "
if shortcuts sign --mode anyone --input "$FILE" --output "$SIGNED" 2>/dev/null; then
    echo "PASS → $SIGNED"
else
    echo "FAIL"
    shortcuts sign --mode anyone --input "$FILE" --output "$SIGNED"
    exit 1
fi

# Step 4: File size check (signed files are ~19KB larger)
ORIG_SIZE=$(stat -f%z "$FILE")
SIGNED_SIZE=$(stat -f%z "$SIGNED")
SIZE_DIFF=$((SIGNED_SIZE - ORIG_SIZE))

echo -n "Signature added... "
if [ $SIZE_DIFF -gt 10000 ]; then
    echo "PASS (+${SIZE_DIFF} bytes)"
else
    echo "WARN (only +${SIZE_DIFF} bytes - may not be properly signed)"
fi

echo ""
echo "=== All automated tests passed ==="
echo "Next: Run 'open $SIGNED' to test import and functionality"
```

Usage:
```bash
chmod +x test-shortcut.sh
./test-shortcut.sh MyShortcut.shortcut
```

---

## Verification Checklist Template

Copy this for each shortcut:

```
Shortcut: [Name]
Date: [YYYY-MM-DD]

[ ] XML validation passed (xmllint)
[ ] Plist validation passed (plutil)
[ ] Signing successful
[ ] Import to Shortcuts.app worked
[ ] Visual inspection: no warning icons
[ ] Run test: completed without errors
[ ] Variables display correctly
[ ] Control flow works as expected
[ ] Edge cases tested:
    [ ] [Case 1]
    [ ] [Case 2]

Notes:
_______________________
```
