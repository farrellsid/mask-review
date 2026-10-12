# Mask review

Corrected neuron masks, sorted by how far along each one is. This repo only holds
the masks and the metadata describing them, never the EM images you're actually
drawing on (more on that below).

## Layout

```
done/                     fully corrected and repropagated
pending_repropagation/    corrected, waiting on a repropagation pass
not_done/                 no correction work started yet
  manual_verify_<GROUP>/
    <NEURON>/
      chain_00/ ...
```

Each `manual_verify_<GROUP>` folder covers one or two neurons (`manual_verify_AIA`
holds both `AIAL` and `AIAR`, for example). When a group's status changes, its
folder just moves to a different top-level directory. Nothing else about it changes.

## Why no frames

| | Size | Changes | Lives |
|---|---|---|---|
| Masks and metadata | a few MB per group | constantly | here, in git |
| EM frames | regenerated on demand | never shipped | your own raw EM copy |

This repo never ships frames. Every chain's `state.json` has `frames_dir: null`,
which tells `gui.py` to regenerate that chain's frames itself from your own raw EM
copy the first time you open it (same thing that already happens during a recrop).
That only works on a machine that's been through `set-up-sam2-on-a-review-machine.md`
(torch, a checkpoint, raw EM). If it hasn't, `gui.py` will just fail to open the
chain instead of quietly showing you nothing.

## Setup

```bash
git clone <repo-url> ~/mask-review
cd ~/mask-review
python3 -m pip install --user -r requirements-review.txt
```

That's the whole setup. Your raw EM is already local, so there's no frame-copying
step; `gui.py` regenerates each chain's frames the first time you open it, a few
seconds per chain.

## Reviewing

```bash
python3 launcher.py
```

Browse to a group folder in your clone, e.g. `~/mask-review/not_done/manual_verify_ADA`.
Tick the neurons you want, leave mode on **Redraw only**, hit Launch.

## Sending work back

Whenever you finish a stretch of work:

```bash
cd ~/mask-review
git add -A
git commit -m "AIAL chains 2 to 8 reviewed"
git push
```

Commit as often as you want, each one's a checkpoint you can roll back to. Pushing
is what actually makes the work visible on the other end.

If a group's status changes, say you finish correcting everything in it, move its
folder into the right top-level directory as part of that commit:

```bash
git mv not_done/manual_verify_ADA pending_repropagation/manual_verify_ADA
```

## Getting updates

```bash
cd ~/mask-review
git pull
```

New groups show up the same way as everything else here. No separate frame step
for those either.

## Two rules

**Only you touch your masks.** New chains might get added from the other side, but
nobody else edits a mask you're working on. Git can't merge two versions of a PNG,
so if edits overlap, one of them just gets thrown away.

**Never commit a jpg.** `.gitignore` already blocks it, and none should ever show up
in here anyway, since frames regenerate into your own machine's scratch cache, not
the clone. If git ever offers to commit one, stop and ask what's going on.

## When something looks wrong

Run this on a group folder in your clone and it'll check the structure and tell you
what's missing:

```bash
python3 -c "from sam2_utils import bundle; print(bundle.validate_bundle('$HOME/mask-review/done/manual_verify_AIA', require_frames=False) or 'clean')"
```

`require_frames=False` is on purpose. These bundles never carry frames, so a frames
check would always wrongly flag them as missing.
