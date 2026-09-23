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
deferrals: []                  # ids of prior decisions this brief is the
                                # live destination for (at most one owner)
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
park intake ...      # the deposit queue, briefs/inbox/, and GitHub issues
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

The same reasoning covers the other direction — not "what blocks this" but
"where did that go." `deferrals` is a list of ids too, so `park validate`
refuses two briefs claiming the same one: a destination that could resolve
to either is not a destination. Deciding "where did X go" by scanning prose
for X's name has the opposite failure mode from a stale gate — it can
resolve to whatever text happens to sit near a *mention* of X, silently,
which is worse than not resolving at all.

## Intake: the deposit queue

Not everyone who finds work should write a brief. An analyst agent with no
commit rights, an issue tracker, a script, the residue of a review: each
can leave a **deposit** in `briefs/inbox/`, and a person or their agent
promotes it into a brief or rejects it. Deposits are proposals. They never
enter the index, because the loader reads `briefs/*.md` and nothing deeper.

A deposit is one markdown file, `briefs/inbox/<id>.md`:

```
Deposited by <who>, <YYYY-MM-DD>, origin <origin>
<the one-line map entry the promoted brief would carry>

<the finding: what was observed, where, what it would change,
what would falsify it>
```

`<origin>` is a closed vocabulary: `analyst`, `operator`, `script`,
`review`, or `github#<N>` for issue N. It lives on that line only; there is
no frontmatter field for it. A line 1 without `, origin ...` is the form
written before 0.4.0; `promote` accepts it with an explicit `--origin`.

```
park intake list                       # every deposit, and whether it promotes
park intake deposit <id> < text.md     # create briefs/inbox/<id>.md (or --file)
park intake promote <id> [--type T] [--trigger TEXT]... [--origin O]
park intake reject <id> --reason "..." # delete it, print one record line
```

- `deposit` validates line 1 (a real date, an origin from the vocabulary)
  and line 2, and only ever CREATES: an existing deposit, an existing brief
  of the same id, or an issue number already deposited or promoted is
  refused.
- `promote` writes `briefs/<id>.md` with a scaffolded frontmatter
  (`type` from `--type`, default `task`; `status: open`, `owner: user`,
  `gate: []`, `triggers` from `--trigger`, `deferrals: []`, `brief:` from
  line 2), opens its body with the provenance line, regenerates the index
  and deletes the deposit. It is all of that or none of it: on any refusal
  or failed write, every file is left byte-identical to before. It never
  overwrites a brief, and it refuses a promotion the tree would not
  validate.
- `reject` deletes the deposit and prints one line naming it, its
  provenance and your reason. It writes no record file: keep the line
  where you keep decisions (a commit message, a chat). A deposit it
  cannot read is still rejected, since clearing what cannot be promoted
  is its job; the line then says the provenance is unknown.
- `briefs/inbox/README.md` is the queue's own file and never a deposit,
  in any letter case, on any filesystem: `list` skips it, `promote` and
  `reject` refuse its name (`PARK-E202`) and `deposit` refuses to create
  it (`PARK-E216`). The rule is by name (`readme`, compared case-folded)
  and by identity (whatever the filesystem resolves to that file, a link
  included). A case-sensitive filesystem refuses the same spellings as a
  case-insensitive one, so a stray `readme.md` beside `README.md` there is
  yours to remove by hand.
- A read park cannot make is a refusal, never a traceback: an input file
  or stdin (`PARK-E217`), a deposit or the queue's listing (`PARK-E217`),
  a brief or `briefs/`'s listing (`PARK-E102`, `PARK-E114`), and
  `TODO.md` (`PARK-E142`). A directory park cannot list is refused, not
  read as empty.

### GitHub issues

With the GitHub CLI (`gh`) installed and authenticated:

```
park intake github [--repo OWNER/NAME] [--label L] [--limit N]
park intake github --outbound [--repo OWNER/NAME]            # plan only
park intake github --outbound --apply [--repo OWNER/NAME]    # make it so
```

- **Inbound** reads the forge and writes only local files: each open issue
  not already recorded as `origin github#<N>`, on a deposit's line 1 or a
  brief's first body line, becomes one deposit, `gh-<N>-<title words>.md`,
  its map line the issue's title. Nothing is written to GitHub.
- **Outbound** looks for briefs whose provenance is `github#<N>` and whose
  status is `done`, reads each issue's state, and prints the plan: which
  open issues it would close, with the comment it would leave. Only
  `--apply` closes them (`gh issue close --reason completed --comment`),
  and an issue already closed is skipped, so a second run does nothing.
- Nothing is scheduled and there is no Project-board sync: every run is
  someone's command.

Limits, stated: `github#<N>` names an issue in the repository `gh` is
pointed at, so one briefs tree follows one repository. A rejected issue
comes back on the next import unless it is closed or excluded with
`--label`. If a close fails partway through `--apply`, the closes before
it stay made; the refusal names them.

## Refusal codes

Every refusal line opens with a stable code. Name the code, not the prose,
in anything that reacts to a refusal: messages may be reworded, codes are
never reused.

| code | refusal |
|---|---|
| `PARK-E001` | usage: an unknown verb, flag or combination |
| `PARK-E101` | `briefs/` does not exist at or above the working directory |
| `PARK-E102` | a brief cannot be read as UTF-8 text |
| `PARK-E103` | a brief has no frontmatter (its first line is not `---`) |
| `PARK-E104` | a brief's frontmatter is never closed with `---` |
| `PARK-E105` | a frontmatter line is not `key: value` |
| `PARK-E106` | an unknown frontmatter field |
| `PARK-E107` | a duplicate frontmatter field |
| `PARK-E108` | a list item outside a list field |
| `PARK-E109` | an empty list item |
| `PARK-E110` | a list field that is not a list |
| `PARK-E111` | a required field is missing |
| `PARK-E112` | the id does not match the filename stem |
| `PARK-E113` | a duplicate id |
| `PARK-E114` | `briefs/` cannot be listed |
| `PARK-E120` | `type` outside its vocabulary |
| `PARK-E121` | `status` outside its vocabulary |
| `PARK-E122` | `owner` outside its vocabulary |
| `PARK-E123` | the `brief` line is empty |
| `PARK-E124` | a watch whose status is not open |
| `PARK-E125` | a watch without a trigger |
| `PARK-E130` | a `manual:` gate carrying no checkable fact |
| `PARK-E131` | a gate that references its own brief |
| `PARK-E132` | a gate that resolves to no brief |
| `PARK-E133` | parked, but every gate has closed: the gate has opened |
| `PARK-E134` | one deferral id claimed by more than one brief |
| `PARK-E140` | `TODO.md` carries no generated region (`map --check`) |
| `PARK-E141` | `TODO.md` is stale against the briefs (`map --check`) |
| `PARK-E142` | `TODO.md` cannot be read as UTF-8 text |
| `PARK-E201` | a deposit id that is not a lowercase kebab slug |
| `PARK-E202` | no such deposit in `briefs/inbox/` (the queue's README, in any spelling, is none) |
| `PARK-E203` | the deposit already exists (deposits are create-only) |
| `PARK-E204` | a brief with that id already exists |
| `PARK-E205` | a deposit that is empty or not UTF-8 text |
| `PARK-E206` | line 1 is not a provenance line |
| `PARK-E207` | an origin outside the closed vocabulary |
| `PARK-E208` | the provenance line declares no origin |
| `PARK-E209` | `--origin` given for a line that already declares one |
| `PARK-E210` | line 2, the map line, is missing or blank |
| `PARK-E211` | a scaffolded value is blank, multi-line, padded, or does not read back as itself |
| `PARK-E212` | a rejection reason that is blank or not one line |
| `PARK-E213` | that `github#N` origin is already deposited or promoted |
| `PARK-E214` | a write failed; every file was restored |
| `PARK-E215` | promotion refused: the tree would not validate |
| `PARK-E216` | a deposit id reserved for the queue's README, in any letter case |
| `PARK-E217` | an intake input cannot be read: the `--file` or stdin text, a deposit, or the `briefs/inbox/` listing |
| `PARK-E301` | `gh` is not on PATH |
| `PARK-E302` | `gh` failed or timed out |
| `PARK-E303` | `gh` answered something other than the expected JSON |
| `PARK-E304` | a deposit file for an issue exists without its origin |
| `PARK-E305` | an outbound close failed; the ones before it were made |

## Install

Two paths, no package index. Requires Python 3.15. Either way, park finds
the repository from where you RUN it — the nearest directory at or above
your working directory carrying `briefs/` — never from where it is
installed, so both paths work from anywhere inside the repository.

Until 3.15.0 final ships, provision the interpreter first: `uv python
install 3.15` installs the release candidate, which both paths then use.
Without it, `uvx` finds no interpreter for `>=3.15,<3.16` (a release
candidate is not offered for that range), and a copy has no `python3.15`
to run under.

It is one file with no dependencies: copy `bin/park` from this repository to
`bin/park` in yours — or anywhere on your `PATH` — and make it executable.
Its first line is `#!/usr/bin/env python3.15`, so `python3.15` must be on
your `PATH` for the copy to run; `python3.15 bin/park` works the same way.

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
change here first. The one subfolder park reads is `briefs/inbox/`, the
deposit queue, and only through `park intake`. Say so in an issue; it is a small change, and knowing which
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
