# nongnu

This repository has the pipeline of a mirror of Savannah's nongnu release tree,
`rsync://dl.sv.gnu.org/releases/`, in Cloudflare R2, at `https://nongnu.katoptra.org/`.
`README.md` identifies the upstream, and it tells how to use the mirror and how to fork it.
The README of [katoptra/lib](https://github.com/katoptra/lib) is the manual for all the
parts that more than one mirror uses. This file gives the rules that each change must obey.

Nothing in this repository starts a run. An external scheduler dispatches `sync.yml` at
03:42 and 15:42 UTC. A run does `reconcile` if it is 23.5 hours or more since a run did
the last reconcile. This can be the run at 03:42 or the run at 15:42. `Taskfile.yml`
contains only the root vars and the two includes, and each verb is lib's.

## Constraints

- **Where a change goes.** `Taskfile.yml` has no verbs. Make all changes to verbs in lib: in
  the toolbox or in the rsync engine. Then each mirror that includes that file gets the
  change. An example is a change to the movement of bytes, to the pages or to the checks of
  `smoke`. The includes have no `excludes:`.
- **Root vars.** Root vars hold only the values of this mirror. Do not put an engine default
  in a root var, because then the command line cannot set it
  ([lib README, Rules a mirror keeps](https://github.com/katoptra/lib#rules-a-mirror-keeps)).
  Do not set a lib var again. In an engine verb, the engine uses a root var, not a
  `KEY=value` from the command line. Thus, `MAX_BATCHES`, `BATCH_GB` and `RECONCILE` are
  not root vars.
- **Paths are Savannah's, at the root of the bucket.** Only the mirror uses the prefix
  `.state/` and the file name `nongnu.katoptra.org.directory.index.html`. Savannah has no
  path with this prefix or with this file name.
- **GNU's mirror guidelines are applicable to each page that this host serves.** The text
  must be short, and it must only give an explanation. Images and logos are not permitted.
  The only permitted link is a link for bug reports.
- **`PAGE_FOOT` is the only text that the mirror adds.** The pages in the bucket show a
  change to `PAGE_FOOT` only after the engine makes all the pages again. To make all the
  pages again, run `aws s3 rm s3://nongnu/.state/indexed.txt.xz`. The next run then sends
  approximately 12,700 PutObjects.
- **The bug-report link is `mailto:nongnu@katoptra.org`.** That address must get e-mail.
  The mail records of the zone are not in this repository. They are in the same location as
  the zone rules.
- **Storage.** For each change that adds storage, calculate the new storage. Compare it with
  the 80 GB baseline and the 120 GB ceiling.
- **Writing.** Obey ASD-STE100 and the rules in the
  [Writing section of the org CONTRIBUTING](https://github.com/katoptra/.github/blob/main/CONTRIBUTING.md#writing).
  Read that section before you write.

## Must knows

- **rsync gives the exit code 23 for each listing.** In the upstream tree, two symlinks
  point to no file, and three symlinks make a loop. The engine accepts the exit code 23.
  rsync also gives this code for a directory that it cannot read. Then the engine deletes
  the files of that directory
  ([lib, The rsync engine](https://github.com/katoptra/lib#the-rsync-engine)).
  `LIST_FLOOR: 45000` stops a run if the listing decreases by more than approximately 6,000
  files.
- **Keys contain spaces (73 keys) and `& < > " '` (15 keys).** The engine percent-encodes
  each href and HTML-encodes the text. Do not parse a listing by field.
- **The zone rules are not in this repository.** README step 4 gives them. The canary finds
  a missing rule: `smoke` reads `00_MIRRORS.html` as `libwww-perl`. The canary is a file at
  the root. Thus, it is in the last batch, and `smoke` reads it from the first run that has
  it in the state.
- **No `Content-Encoding`.** GNU recommends no `Content-Encoding` header. On each run,
  `smoke` reads a `.tar.gz` and makes sure that it has no `Content-Encoding`.
- **A run failure is the only alert.** The healthchecks.io check has the cron
  `42 3,15 * * *` UTC and a grace time of 3 hours.

## Verifying a change

- `task check` makes a render of each command of the pipeline in the image. Then it compares
  the render with `render.txt`. `task render-update` accepts a change. On each pull request,
  the check workflow does the same `task check`, with no secrets.
- `task run -- task list` gets a listing of upstream with no credentials. It is the one
  check of upstream without credentials. `.run/upstream.txt` then has approximately 51,000
  lines. Of these lines, 73 have a space, and one is `00_TIME.txt`.
- `task plan` does the read-only part of a run with the bucket of the mirror: `clock`,
  `list`, `state`, `diff` and `split`. It uses the secrets from the vault.
- The verbs of the engine, the pages and the read-back checks are lib's. To do a test of
  them, use `cd ../lib/examples/rsync && task run -- task offline`.
- To read the clock of the mirror (freshness), use
  `curl -s https://nongnu.katoptra.org/00_TIME.txt`.
