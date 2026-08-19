# Mask review

Corrected neuron masks for AIAL, AIAR, AIYL and AIYR: 150 chains, sliced through z.
This repo holds the masks and the metadata that describe them. It does not hold the EM
images you draw on, which arrive separately (see below).

## The split, and why

| | Size | Changes | Lives |
|---|---|---|---|
| Masks and metadata | ~9 MB | constantly | here, in git |
| EM frames | ~1.9 GB | never | on the drive |

The frames are static and regenerable, so putting them in git would add nearly two
gigabytes of history to carry something that never changes. Keeping them out means this
repo stays small enough to clone in seconds and to see a real history of the corrections.

## Setting up, once

You need three things: this repo, the review code, and the frames. The code and frames
both come from the drive.

```bash
# 1. Clone this repo somewhere on your Mac.
git clone <repo-url> ~/mask-review

# 2. Install the review tools (from the code folder on the drive).
cd /Volumes/Expansion/Lucinda_Review/code
python3 -m pip install --user -r requirements-review.txt

# 3. Put the frames into your clone, one bundle at a time.
python3 place_frames.py --clone ~/mask-review/AIY_for_lucinda \
    --from /Volumes/Expansion/Lucinda_Review/bundles/AIY_for_lucinda
python3 place_frames.py --clone ~/mask-review/AIA_for_lucinda \
    --from /Volumes/Expansion/Lucinda_Review/bundles/AIA_for_lucinda
```

Step 3 prints `clone validates clean and is ready to review` when it worked. It is safe
to re-run: chains that already have their frames are skipped, so an interrupted copy
picks up where it stopped.

If the drive is not called `Expansion` on your Mac, run `ls /Volumes` and use the name
you see there.

## Reviewing

```bash
cd /Volumes/Expansion/Lucinda_Review/code
python3 launcher.py
```

Browse to a bundle folder **inside your clone** (`~/mask-review/AIY_for_lucinda`), not
the one on the drive. Tick the neurons you want, leave the mode on **Redraw only**, and
click Launch.

Reviewing out of the clone is the whole point: it is what lets your work be committed
and seen. The drive copy is only the source of the frames.

## Sending work back

Whenever you finish a stretch of work:

```bash
cd ~/mask-review
git add -A
git commit -m "AIYL chains 2 to 8 reviewed"
git push
```

Commit as often as you like. Each commit is a checkpoint you can go back to, and pushing
is what makes the work visible on the other end.

## Getting updates

```bash
cd ~/mask-review
git pull
```

If new chains arrive, run `place_frames.py` again for that bundle to fetch their frames
from the drive.

## Two rules

**Only you edit masks.** New chains may be added from the other side, but nobody else
edits a mask you might be working on. Git cannot merge two versions of a PNG, so
overlapping edits mean one of them has to be thrown away.

**Never commit a jpg.** `.gitignore` already prevents it. If git ever offers to commit
one, stop and ask, because it means the frames ended up somewhere they should not be.

## When something looks wrong

Run this against a bundle in your clone. It checks the structure and says what is
missing:

```bash
cd /Volumes/Expansion/Lucinda_Review/code
python3 -c "from sam2_utils import bundle; print(bundle.validate_bundle('$HOME/mask-review/AIY_for_lucinda') or 'clean')"
```
