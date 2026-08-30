# Changelog

The version is a human judgment, bumped deliberately inside the change that
will be published — it is never derived mechanically from commit types — and
`--version` must discriminate any two published trees. Newest first.

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
