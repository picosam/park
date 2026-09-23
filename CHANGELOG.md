# Changelog

The version is a human judgment, bumped deliberately inside the change that
will be published — it is never derived mechanically from commit types — and
`--version` must discriminate any two published trees. Newest first.

## 0.4.0

**park requires Python 3.15.** `requires-python` is now `>=3.15,<3.16`,
and `bin/park`'s first line names the interpreter:
`#!/usr/bin/env python3.15`. Nothing in the tool needs 3.15; the move is
for one interpreter across the tools this one is released beside, which
move together at each minor. The declaration governs installs. This
release is published on the 3.15.0 release candidate, as if final: until
3.15.0 final ships, run `uv python install 3.15` before either install
path below, because `uvx` provisions nothing from `>=3.15,<3.16` while
only the rc exists, and an installed rc is used (measured with uv
0.12.18 in an empty interpreter store: the `uvx --from` path refused
before that command and ran `park 0.4.0` after it). park has
no runtime interpreter check, so `python3.15 bin/park` is the one
supported invocation, and an older interpreter that happens to run the
file is not a supported one.

**Intake: a deposit queue in `briefs/inbox/`.** Four verbs take proposals
from whoever should not write a brief directly (an analyst agent without
commit rights, an issue tracker, a script, a review's residue) and file
them. `park intake deposit <id>` creates `briefs/inbox/<id>.md` from stdin
or `--file`, validating its first two lines and never overwriting.
`park intake promote <id>` scaffolds the brief's frontmatter, opens its
body with the provenance line, regenerates the index and deletes the
deposit: all of it or none of it, every file byte-identical to before on
any refusal or failed write. `park intake reject <id> --reason ...` deletes
the deposit and prints one record line. `park intake list` says what is
waiting and whether each deposit would promote. Deposits never enter the
index: the loader still reads `briefs/*.md` and nothing deeper.

**GitHub issues in and out, through `gh`.** `park intake github` reads
the open issues (`gh issue list`) and writes one deposit per issue not
already recorded as `origin github#<N>`; it writes nothing to GitHub.
`park intake github --outbound` prints a plan: for each brief whose
provenance is `github#<N>` and whose status is `done`, the close and the
comment it would make, skipping issues already closed. Only `--apply`
makes those closes. Nothing is scheduled, and no Project board is touched.

**`origin` is a closed vocabulary on the provenance line.** A deposit's
line 1 reads `Deposited by <who>, <YYYY-MM-DD>, origin <origin>`, where
`<origin>` is `analyst`, `operator`, `script`, `review` or `github#<N>`.
There is no new frontmatter field: a brief's schema is unchanged.

**Every refusal carries a stable code.** Each refusal line now opens with
`PARK-Ennn`, and the README lists every code. Tests, scripts and
allowlists can name the refusal rather than its wording, and a code is
never reused. Usage errors are `PARK-E001` and still exit 2.

**A read park cannot make is a refusal, not a traceback.** An input
file or stdin, a deposit, the queue's listing (`PARK-E217`), the
`briefs/` listing (`PARK-E114`) and `TODO.md` (`PARK-E142`) each refuse
with a code. A directory park cannot list is refused rather than read as
empty, which is what `validate` and `map` did before. The queue's
`README.md` is reserved in every letter case: `promote` and `reject`
refuse its name (`PARK-E202`), `deposit` refuses to create it
(`PARK-E216`).

### Upgrading from 0.3.x

- **You need Python 3.15.** Trees, briefs and the index are unchanged:
  nothing a 0.3.x tree validated is refused for its content.
- **The copy-one-file install needs `python3.15` on your `PATH`.** A
  shebang cannot fall back to another name, so a copied `bin/park` on a
  machine without `python3.15` fails before it starts. Either provision
  3.15, or run it as `python3.15 bin/park`.
- **`uvx --from git+<this repository> park` provisions an interpreter only
  once 3.15.0 final exists.** uv offers no download for `>=3.15,<3.16`
  while 3.15 has only release candidates (measured with uv 0.12.17 on
  2026-09-23: it offers `3.15.0rc2` for a `3.15` request and nothing for
  the declared range), so this release is published after 3.15.0.
- **Staying on 3.11 to 3.14** means staying on 0.3.x.
- **Anything that matched a refusal's text** should match its code
  instead. The text after the code keeps its meaning but may be reworded;
  a line that begins `PARK-Ennn` always means the refusal the README's
  table gives that code.
- **`park intake github` needs the GitHub CLI**, installed and
  authenticated; without it the verb refuses with `PARK-E301` and the
  rest of park is unaffected.
- **Deposits written before 0.4.0** carry no `, origin ...` on line 1.
  `promote` takes them with `--origin <origin>`, which it appends to the
  provenance line in the brief; `deposit` refuses the old form.

## 0.3.0

**A brief can declare it is a deferral's destination.** New field
`deferrals` — a list of ids, default `[]` — names any prior decision this
brief is the live record for. `park validate` refuses the same id claimed
by two briefs' `deferrals`, the same way it already refuses a gate that
resolves to nothing: a destination is a structural fact now, not a
sentence a scanner has to relocate every time the prose around it moves.

This closes the gap a checkbox-scanning guard used to paper over: locating
"where did that deferred item go" by regex over a declaration line broke
on a reflow that changed no meaning, and — the more dangerous direction —
could resolve to whichever text happened to sit next to a mention of the
id, rather than to its actual owner. `deferrals` makes the destination a
field a validator can check, not a location a reader has to infer.

**Not a breaking change for existing trees.** `deferrals` is optional and
reads as `[]` when absent, so a brief written against 0.2.x validates
unchanged. Declare it only where it carries ids — the briefs that are a
prior decision's live destination; `park validate` then refuses the same
id claimed twice.

## 0.2.0

**The map is active-only.** `park map` renders open, watch and parked
briefs. Briefs with `status: done` or `status: superseded` are still
parsed, still validated and still resolvable as another brief's gate —
they are simply no longer rendered as routing lines, because the map is
read to choose the next piece of work, and a closed record answers a
different question.

Measured in the repository this tool was built for: the closed section was
41.6% of the generated map (8,578 of 20,636 bytes, 30 of 72 items) and no
test, gate, script or document read it.

The omission is stated, not silent: the generated region ends with a count
of the briefs it did not list, and that line is inside the guarded region,
so `park map --check` refuses a hand edit of it like any other drift.

Closed briefs stay discoverable with the primitives you already have, and
term search gets strictly better — it reaches whole brief bodies, not just
the one-line summaries the map carried:

    rg -l '^status: (done|superseded)$' briefs
    rg <term> briefs

The map's preamble carries both commands, so nobody has to remember them.

Behaviour change, not a breaking one: no input is refused that was
accepted before, and every brief a 0.1.x tree validated still validates.

## 0.1.1

Documentation only; no behaviour changes.

**Onboarding starts from what the session loaded.** §5 of the onboarding
prompt now has the agent state which instruction files its session
actually loaded, and partitions three topologies: a repository
instruction file is repaired in place; a session that loaded only
user-level or machine-global rules proposes a minimal repository
instruction file rather than routing project facts into a private global
file that travels with nobody; and a session that loaded nothing
proposes the same file. In both repository-absent branches the proposal
carries only the operating floor: the five §5 repairs plus one line of
commit discipline (work on the agreed branch, commit logical units, push
only on the owner's explicit request). Nothing else is injected;
language rules, roles and house style stay the owner's.

## 0.1.0

First publication. The version was declared before anyone could install the
tool, precisely so that no two published trees can ever share one.

**What it is.** A backlog format for AI coding agents — one topic per
markdown file with YAML frontmatter under `briefs/`, and a compiled index
(`TODO.md`) an agent reads on demand instead of loading the whole backlog.
One Python file, standard library only, three verbs: `validate`, `map`,
`map --check`. The repository root is resolved from the working directory —
the nearest ancestor carrying `briefs/` — never from where the tool is
installed, so a copy on `PATH` or a `uvx` install works on whatever
repository you are standing in.

**What it validates.** The schema exactly — the seven fields
`id/type/status/owner/gate/triggers/brief`, with closed vocabularies and an
id that must match its filename; that every gate resolves to a real brief id
(the one prose form being `manual: <checkable fact>`, unresolvable by
design); that a watch is armed, not worked — `status: open` plus at least
one trigger; the gate-opened pathology — a parked brief whose gates have all
closed is work that is unblocked while still filed as blocked, and the tool
says so; and index drift — `map --check` is non-zero whenever the generated
region disagrees with the briefs, so the index cannot silently rot.

**Install paths.** Copy the one file, or
`uvx --from git+<this repository> park` — `pyproject.toml` backs the second
and was verified end to end before this entry was written.

**What it deliberately does not do.** No YAML library: the parser accepts
precisely the ratified subset (plain scalars, inline and block lists) and
refuses everything else naming the offending field, so the tool stays one
file with no dependencies. No path configuration: `briefs/` and `TODO.md`
at the repository root are convention, stated as a limit in the README
rather than hidden. It never applies `superseded` — deciding that one piece
of work replaces another is a human ruling, not a validation. And nothing
below the index's end marker is ever touched, which is what lets a
migration run in batches with both systems true at every step.
