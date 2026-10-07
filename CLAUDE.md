# nongnu

A mirror of Savannah's nongnu release tree (`rsync://dl.sv.gnu.org/releases/`) on
Cloudflare R2, served at `https://nongnu.katoptra.org/`. `README.md` says what it mirrors,
how to use it and how to fork it; [katoptra/lib](https://github.com/katoptra/lib)'s README
is the manual for everything the mirrors share. This file is what a change must not break.

Nothing in this repo starts a run: an external scheduler dispatches `sync.yml` at 03:42
and 15:42 UTC; `reconcile` runs once the last one is 24 h old, whichever run that is.
`Taskfile.yml` holds root vars and the two includes and nothing else; every verb is lib's.

## Constraints

- No verbs of its own. A change to how bytes move, how pages are drawn or what `smoke`
  reads goes to lib's engine, where ctan and gnu get it too. The one exclude is
  `report-engine`, on the toolbox include. Never redefine a lib var. Inside an engine verb a
  root var shadows a command-line `KEY=value`, so `MAX_BATCHES`, `BATCH_GB` and `RECONCILE`
  stay out of the root vars.
- Paths are Savannah's, at the bucket root. `.state/` is the reserved prefix and
  `nongnu.katoptra.org.directory.index.html` the reserved file name; Savannah has neither.
- GNU's mirror guidelines govern every page this host serves: text short and strictly
  explanatory, no images or logos, no link but a bug-reporting one. `PAGE_FOOT` is the only
  text the mirror adds. A change to it reaches existing pages only through a full redraw:
  `aws s3 rm s3://nongnu/.state/indexed.txt.xz`, about 12,700 PutObjects on the next run.
- The bug-reporting link is `mailto:nongnu@katoptra.org`, and that address must take mail.
  The zone's mail records live outside this repository, beside its rules.
- Recompute any change that adds storage against the 80 GB baseline and the 120 GB ceiling.

## Must knows

- **Every listing exits 23**, from two dangling symlinks and three loops upstream. The
  engine passes it. The same code means an unreadable directory, whose files the engine
  then deletes as gone; `LIST_FLOOR: 45000` stops a loss of more than about 6,000.
- **Keys carry spaces (73) and `& < > " '` (15).** The engine percent-encodes hrefs and
  HTML-escapes text. Never parse a listing by field.
- **The zone rules live outside this repository**; README step 4 lists them. The canary,
  `00_MIRRORS.html` read as `libwww-perl`, is what notices one missing. It is a root file,
  so it lands in the last batch and is checked from the first run that holds it.
- **No `Content-Encoding`.** GNU asks for none; `smoke` asserts it on a `.tar.gz` every run.
- **A failed run is the only alert.** healthchecks.io cron `42 3,15 * * *` UTC, 3 h grace.

## Verifying a change

- `task check` renders every command of the pipeline inside the image and diffs it against
  `render.txt`; `task render-update` accepts a change.
- `task run -- task list` lists upstream with no credentials: about 51,000 lines in
  `.run/upstream.txt`, 73 with a space, `00_TIME.txt` among them.
- The engine's verbs, the pages and the read-back checks are lib's:
  `cd ../lib/examples/rsync && task run -- task offline`.
- Is the mirror fresh? `curl -s https://nongnu.katoptra.org/00_TIME.txt`.
