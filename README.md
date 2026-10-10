# Mask review

Corrected neuron masks, organized by how far along each one is. This repo holds the
masks and the metadata that describe them, never the EM images you draw on (see below).

## Layout

```
done/                     fully corrected and repropagated
pending_repropagation/    corrected, waiting on a repropagation pass
not_done/                 no correction work started yet
  manual_verify_<GROUP>/
    <NEURON>/
      chain_00/ ...
```

Each `manual_verify_<GROUP>` folder is one or two neurons (e.g. `manual_verify_AIA`
holds `AIAL` and `AIAR`). A folder moves between the three top-level directories as its
status changes; nothing about its own name or contents changes when it moves.

## The split, and why

| | Size | Changes | Lives |
|---|---|---|---|
| Masks and metadata | a few MB per group | constantly | here, in git |
| EM frames | regenerated on demand | never shipped | your own raw EM copy |

No frames are stored in this repo or shipped with these bundles at all. Every chain's
`state.json` has `frames_dir: null`, which tells `gui.py` to regenerate that chain's
frames itself, from your own raw EM copy, the first time you open it (the same thing
that already happens during a recrop). This only works on a machine that has gone
through `set-up-sam2-on-a-review-machine.md` (torch, a checkpoint, and raw EM); if
that is not set up, `gui.py` will fail to open a chain rather than silently showing
nothing.

## Setting up, once

```bash
git clone <repo-url> ~/mask-review
cd ~/mask-review
python3 -m pip install --user -r requirements-review.txt
```

That is the whole setup. There is no frame-copying step: your raw EM is already local,
and `gui.py` regenerates each chain's frames the first time you open it (a few seconds
per chain, not a bulk transfer up front).

## Reviewing

```bash
python3 launcher.py
```

Browse to a group folder inside your clone, e.g. `~/mask-review/not_done/manual_verify_ADA`.
Tick the neurons you want, leave the mode on **Redraw only**, and click Launch.

## Sending work back

Whenever you finish a stretch of work:

```bash
cd ~/mask-review
git add -A
git commit -m "AIAL chains 2 to 8 reviewed"
git push
```

Commit as often as you like. Each commit is a checkpoint you can go back to, and pushing
is what makes the work visible on the other end.

If a group's status changes (e.g. you finish correcting everything in it), move its
folder into the right top-level directory as part of that commit:

```bash
git mv not_done/manual_verify_ADA pending_repropagation/manual_verify_ADA
```

## Getting updates

```bash
cd ~/mask-review
git pull
```

New groups arrive the same way as everything else here, no separate frame step.

## Two rules

**Only you edit masks.** New chains may be added from the other side, but nobody else
edits a mask you might be working on. Git cannot merge two versions of a PNG, so
overlapping edits mean one of them has to be thrown away.

**Never commit a jpg.** `.gitignore` already prevents it, and none should ever be
generated inside this repo anyway, since frames are regenerated into your own
machine's scratch frames cache, not into the clone. If git ever offers to commit one,
stop and ask.

## When something looks wrong

Run this against a group folder in your clone. It checks the structure and says what is
missing:

```bash
python3 -c "from sam2_utils import bundle; print(bundle.validate_bundle('$HOME/mask-review/done/manual_verify_AIA', require_frames=False) or 'clean')"
```

`require_frames=False` is intentional: these bundles never carry frames, so a frames
check would always and incorrectly report them missing.
