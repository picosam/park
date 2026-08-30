# park

**A backlog format for AI coding agents.** One topic per file, machine-checked
status and blockers, and a compiled index your agent reads on demand — instead
of loading your entire TODO into every session.

Markdown with YAML frontmatter. Python standard library only: one file, no
dependencies, no service, nothing to log into. Delete the tool tomorrow and you
still have plain markdown that reads fine.

## The problem it solves

A long backlog is a context tax. Every session pays for all of it to find one
item — so it gets skimmed, and the parts nobody re-reads are exactly the parts
that rot:

- work parked behind a blocker that has **since cleared**, and nobody noticed;
- decisions buried inside entries marked done, findable only by whoever already
  knew they were there;
- the same item filed twice under two names, drifting apart.

A checkbox carries two states. A real backlog has a dozen: parked-with-a-trigger,
waiting-on-someone-else, decided-but-unrecorded, designed-not-implemented,
superseded, done-but-unchecked. `park` gives those states names a machine can
check.

## The format

One file per topic, under `briefs/`:

```markdown
---
id: retire-legacy-exporter     # unique; matches the filename
type: task                     # task | decision | watch | record | procedure
status: parked                 # open | parked | done | superseded
owner: agent                   # agent | user — whose move is it?
gate: [new-export-pipeline]    # blockers, as brief ids
triggers: []                   # prose events, for armed watches
brief: Retire the legacy exporter once the new pipeline carries production.
---

The body: why, what was decided, what would falsify it. Links other briefs
by [[id]]. Cites where it came from.
```

The `brief:` line is what gets compiled into the index. An agent reads the
index, then opens the **one** brief it needs.

## What the tool does

```
park validate        # schema, closed vocabularies, unique ids, gates resolve
park map             # compile briefs/ into the index (TODO.md)
park map --check     # non-zero when the index and the briefs disagree
```

Wire `--check` into CI and the index can never silently rot — which matters
more than it sounds, because an index an agent trusts and that is wrong is
worse than no index at all.

**The check that earns its keep:** `gate` is a list of brief *ids*, not prose.
So when every blocker of a parked brief has closed, the tool says so — the
work is unblocked and nobody noticed. Prose blockers cannot be checked, and
they rot quietly: they name mechanisms that were later deleted, or conditions
that quietly came true months ago. That failure mode is the reason this format
exists.

## Install

Two paths, no package index. Requires Python 3.11+. Either way, park finds
the repository from where you RUN it — the nearest directory at or above
your working directory carrying `briefs/` — never from where it is
installed, so both paths work from anywhere inside the repository.

It is one file with no dependencies: copy `bin/park` from this repository to
`bin/park` in yours — or anywhere on your `PATH` — and make it executable.

```
chmod +x bin/park
bin/park --version
```

Or run it without copying anything — `pyproject.toml` backs this path:

```
uvx --from git+<this repository> park --version
```

## Onboard a repository

There is a ready-to-paste prompt for your coding agent — it inventories what
the repository already parks, maps it onto this schema, migrates in batches,
and retires the old mechanism deliberately rather than leaving two live:

**[prompts/onboard.md](prompts/onboard.md)**

## Conventions, and their limits

Stated plainly rather than implied: `park` reads `briefs/*.md` at the
repository root and writes the index into `TODO.md`, between a generated-region
marker and the end of the file. Those paths are convention, not configuration —
there are no flags to move them yet, and the brief tree is flat, so a
repository that needs scoped subfolders or a differently-named index needs a
change here first. Say so in an issue; it is a small change, and knowing which
shape people actually need is worth more than guessing at one.

`superseded` is applied by a human ruling, never by the tool. Deciding that one
piece of work replaces another is a judgment, and a validator that made it
automatically would quietly retire things nobody agreed to retire.

## Verifying a checkout

```
bin/park --version
bin/park validate
```

## Where this tree comes from

This repository is **published, not developed**: it is generated output,
extracted from a private workbench where the tool is used daily against a real
backlog. Every byte here has a source there, so an edit made here does not
survive the next extraction and the repository takes no pull requests. Report
issues instead — the fix lands in the source and arrives with the next
extraction.

## License

MIT. See [LICENSE](LICENSE).
