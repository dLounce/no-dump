---
name: no-dump
description: "Use during any coding or Kaggle project to save tokens: ask the user for only the specific output or code segment needed, and send back only the changed code, never full files or full logs."
---

# no-dump

Save tokens in both directions:
- **Incoming**: the user pastes only the piece of output or code Claude actually needs.
- **Outgoing**: Claude sends only the code that changed.

Never create files. Work in chat.

---

## Part 1: Asking for output

Never say "paste the output" or "share the full log." Every request must say:
1. **What to check**: the one question Claude is answering.
2. **Exactly what to paste**: specific lines, values or a small check cell.
3. **Rough size**: for example "about 5 lines."

### Ways to ask
- **Check cell (preferred)**: a short cell or command that prints a summary of the whole run, ideally True/False plus key numbers. Summaries catch problems the last few lines hide.
- **Point to a segment**: "Paste only the last 10 traceback lines," "only the final CV score line."
- **Yes/no**: "Did it run without errors? Reply yes or no plus the final score."

### Python check cells
```python
# Training summary: catches problems anywhere in the run, not just the end
import numpy as np
l, vl = history.history['loss'], history.history['val_loss']
print('epochs:', len(l), '| any NaN:', bool(np.isnan(l + vl).any()))
print('best val_loss:', round(min(vl), 4), 'at epoch', int(np.argmin(vl)) + 1)
print('final loss/val:', round(l[-1], 4), round(vl[-1], 4))
```
```python
# Data check
print(df.shape)
print(df.isnull().sum()[df.isnull().sum() > 0])
print(df['target'].value_counts(normalize=True).round(3))
```
```python
# Submission check
sub = pd.read_csv('/kaggle/working/submission.csv')
ss = pd.read_csv('/kaggle/input/<comp>/sample_submission.csv')
print(sub.shape == ss.shape, list(sub.columns) == list(ss.columns), sub.isnull().sum().sum())
```

### Terminal / non-Python patterns
```bash
command 2>&1 | tail -20                  # last 20 lines only
command 2>&1 | grep -iE "error|warn" | head -20   # errors and warnings only
command 2>&1 | grep -iE -m1 -A10 "error"          # first error plus 10 lines after
```

### What to request by situation
- Error: last 8 to 10 traceback lines, or the first error block.
- Training: the training summary cell above.
- Data: shape, key dtypes, nonzero null counts.
- Evaluation: the metric line or per-fold scores.
- Install/build: only error/warning lines via grep.
- Long logs: keyword matches or last N lines, never the whole log.

---

## Part 2: Asking for code

Never say "paste your code" or "share the full notebook."

### If Claude hasn't seen the code yet
Ask for an outline first, never the full code:
- "List just your cell titles or function names, one per line."
- Or give a command:
```python
import inspect, sys
print([n for n, o in inspect.getmembers(sys.modules['__main__']) if inspect.isfunction(o) or inspect.isclass(o)])
```
```bash
grep -nE "^(def |class |function |export )" file.py
```
Then ask for the one function or cell that matters.

### If Claude knows the structure
- **Name it**: "Paste only `train_one_epoch`," "only the DataLoader cell."
- **Follow the traceback**: ask for only the function it names, plus what that function calls if needed.
- **Signatures only** when Claude just needs inputs and outputs.
- **Config only** when the question is about settings: "Paste only your CFG dict."
- **Keyword lines**: "Paste only the lines where `lr` is set."

Don't ask again for code already seen in this conversation that hasn't changed.

---

## Part 3: Sending code back

Never resend a full file or notebook when only part changed.
- **1 to 5 lines**: "replace this" and "with this," or new lines plus where they go.
- **One function changed**: send only that full function.
- **New code**: send only the new cell or function, and say where it goes.
- **Several changes**: list by location, each with only its snippet.
- Always mark the position: function name, cell name, or the line above.
- Send full code only if the user asks, or if the change touches most of the file.

### Recovery rule
If the user reports something broke after applying snippets, or seems unsure where a snippet goes, stop sending snippets. Send the complete affected function or cell once, so nothing stays half-applied. Then go back to minimal snippets.

### Example
**In `train_one_epoch`, replace:**
```python
loss.backward()
optimizer.step()
```
**with:**
```python
scaler.scale(loss).backward()
scaler.step(optimizer)
scaler.update()
```

---

## After receiving a segment
- One line on whether things are on track.
- If not, the fix, as a minimal change.
- If the segment wasn't enough, ask for one more specific piece, never the whole thing.

## Never
- Ask for full output, full logs, full notebooks or full files.
- Resend unchanged code.
- Ask for output "just to be safe" on simple, low-risk steps.
- Create files.
