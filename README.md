# 🚫 no-dump

**Stop dumping full logs into Claude.**

A Claude skill that saves tokens during coding and Kaggle projects. Claude asks for **only the output or code segment it actually needs**, and sends back **only the code that changed**. No more full logs, full notebooks or full files.

## Why

When you work on a project with Claude, it often says "paste the output" to check progress. Full outputs and full files:

- waste tokens
- bury the useful lines in noise
- fill the context window faster, so Claude forgets earlier parts of your project

This skill makes Claude ask for the smallest piece that answers its question.

## Before / After

**Before**

> Claude: Run the training and paste the output.
> *(you paste 400 lines of epoch logs)*

**After**

> Claude: Run this and paste the 3 lines it prints:

```python
import numpy as np
l, vl = history.history['loss'], history.history['val_loss']
print('epochs:', len(l), '| any NaN:', bool(np.isnan(l + vl).any()))
print('best val_loss:', round(min(vl), 4), 'at epoch', int(np.argmin(vl)) + 1)
print('final loss/val:', round(l[-1], 4), round(vl[-1], 4))
```

## Features

- **Minimal output requests:** specific lines, summary check cells or yes/no questions instead of full logs
- **Minimal code requests:** asks for an outline first, then only the relevant function or cell
- **Minimal code replies:** "replace these lines with these" instead of resending whole files
- **Recovery rule:** if a snippet breaks something, Claude sends the full affected function once
- **Works beyond Python:** terminal patterns like `tail` and `grep`
- **Kaggle-aware:** submission checks, `/kaggle/input` paths, data checks

## Installation

### Claude app (web / desktop)

1. Download this repo as a ZIP, or zip the `no-dump/` folder.
2. Go to **Settings → Capabilities → Skills**.
3. Upload the ZIP.

### Claude Code

```bash
git clone https://github.com/dLounce/no-dump.git
cp -r no-dump/no-dump ~/.claude/skills/
```

## Repository structure

```
no-dump/
├── README.md
├── LICENSE
└── no-dump/
    └── SKILL.md
```

## When to paste full output anyway

Segments work best when the question is specific. Paste the full output once when:

- the problem is unknown ("score is bad, no error")
- it's the first run of a new pipeline
- two rounds of segments haven't solved the issue

**Rule of thumb:** known question → segment; unknown problem → full output once, then back to segments.

## Contributing

Issues and pull requests are welcome. If Claude still asks for too much output in some situation, open an issue with the example.

## License

MIT License. See [LICENSE](LICENSE).
