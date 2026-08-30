# Onboard this repository to `park`

Paste this whole file to your coding agent as the first message of a session
run **in the repository being onboarded**, with `park` already available
(see the README's Install section).

It ends with: every live parked item as a brief, a compiled index, the drift
check wired into CI, the old parking mechanism deliberately retired, and every
decision the migration surfaced put to a human rather than taken.

---

## The discipline this needs

Two failure modes are worth naming before you start, because both are common
and neither announces itself:

1. **Migrating onto a validator this repository does not run.** An unchecked
   brief tree drifts exactly like the backlog it replaced. Wire `park map
   --check` into CI in the same change that lands the first briefs, or do not
   start.
2. **Leaving two parking systems live.** Half-migrations are worse than no
   migration: readers cannot tell which system is authoritative, so they
   consult neither. Retiring the old mechanism is part of this job, not a
   follow-up. If you cannot finish the retirement, do not begin the migration.

Nothing in this prompt authorizes you to decide anything the repository's
owners have not decided. Migration is **filing, not deciding** — a buried
decision becomes a brief that puts the question in front of a human, still
open. In particular, never mark anything `superseded`: that is a ruling.

## §1 — Inventory what this repository parks. Change nothing.

Read, do not restructure. Find every parked thing, wherever it lives: the
backlog file, design documents, per-topic notes, issue templates, README
sections, the agent instruction files, anything the repository already treats
as a parking lot.

For each: an id, what it is, its state, what it is blocked on, and where it
lives now. Write it to a scratch file — the register is evidence for the
mapping, not a deliverable.

Look for these two deliberately, because they are what the format exists to
catch and they never surface on their own:

- **Items parked inside other items.** A live decision inside an entry marked
  done; a real task as a sub-bullet of an unrelated one. These are invisible to
  anyone who does not already know they are there.
- **Blockers that already cleared.** A note saying "after X" where X happened
  months ago, or one naming a mechanism the project has since deleted, so the
  blocker can never clear at all.

Report the count and the shape before proposing anything.

## §2 — Map the two schemas honestly, and surface what is lost.

If this repository already has a structured parking system — its own
frontmatter, a generated index, lint rules, a sealed archive — the mapping is
not one-to-one, and the mismatches matter more than the matches. Go field by
field and produce a table: their field, the `park` field it maps to, and what
is lost.

Expect at least these classes of friction:

- **Fields with no equivalent.** A scope or category axis, a last-updated date,
  a current-position pointer. Some are genuinely covered by git history; some
  were added on purpose and that purpose may still stand. Ask, do not assume.
- **A status the vocabulary does not carry.** For example, "blocked" is
  usually `status: parked` plus a `gate` naming the blocker — which is
  *better* when the blocker can be named as a brief id or a checkable fact, and
  lossy when it cannot. Any item whose blocker cannot be stated checkably is a
  finding about the schema, not a rounding error.
- **Structural units the format cannot express.** A directory with attached
  non-markdown working artefacts is not a brief; a brief can point at it, it
  cannot be it.
- **A separate genre axis.** Document-type taxonomies (design note, handoff,
  audit, log) are orthogonal to work state. Decide explicitly which documents
  are in scope for this migration and which merely live in the same folder.

Put the losses to a human with options. Do not resolve them by squashing.

## §3 — Migrate in batches, with the register as the checklist.

Never all at once. Work the register section by section, and hold this
invariant at every step: **nothing leaves the old system until its content
exists in a brief and the index resolves to it.** After each batch, both
systems are still true and the checks are still green.

- Closed narratives move **intact** to an archive file — greppable, never
  loaded per session, nothing deleted. History is not clutter; it is why
  decisions are not relitigated.
- A closed record that future work must still find (an armed falsifier, a
  standing decision) additionally gets a `type: record` brief, so routing
  reaches it.
- A buried decision becomes `type: decision`, `owner: <the human>`, status
  open. You are filing it, not answering it.
- Blockers become `gate` ids wherever the blocker is itself a brief. Where a
  blocker is a real-world fact instead, write it as `manual: <checkable fact>` —
  the tool treats that as unresolvable by design and will not claim it opened.

Commit per batch, with the checks passing in each commit.

## §4 — Retire the old mechanism deliberately.

Enumerate, then retire — in reviewed changes, never by abandonment. Typically:

- the old index generator and any lint or CI checks enforcing its schema;
- the CI step that ran them;
- the documentation describing the old model (rewritten, or reduced to a
  pointer);
- references in agent instruction files and contributor docs — an instruction
  file that names the wrong source of truth is precisely the drift this format
  exists to kill;
- the design record of the old system: it gets a successor note, not an edit.
  It remains the true record of what was.

If part of the old machinery guards something the new system does not cover —
a frozen archive, a naming convention with downstream consumers — say so and
keep that part on purpose, with its own reason recorded. Deliberate is the
whole point; what is forbidden is drift.

## §5 — Make this repository's agent instructions true again

An agent instruction file — `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, a rules
file, whatever the agents working here load — is read into **every** session.
That makes a wrong one worse than none at all: an agent acts on it without
checking, and the error repeats silently in every session until a human
happens to notice.

The migration you just ran has very probably made one false. This is not
hypothetical: in the repository where this format was built, the instruction
file called the old backlog file "the backlog's source of truth", and that
sentence was false the moment the migration finished. It was caught because
somebody went looking, which is not a mechanism — which is why this step is
written down.

**Required — repair what this migration changed.** In every instruction file
an agent here loads, make these true:

- where the backlog now lives, and that it is one topic per file;
- that the index is **generated** and must never be hand-edited, naming the
  command that regenerates it;
- how to change the backlog: edit or add a brief, then regenerate;
- that the archive is for grepping, never for loading per session;
- the routing rule itself — read ONE brief on demand, not the tree.

Then delete or rewrite anything else the migration made stale, including
pointers to files that no longer hold what they claim.

**Required too — notice what the agents here load at all.** State which
instruction files this session actually loaded: the repository's own, and
any user-level or machine-global rules your agent surface reads (for
Claude Code `~/.claude/CLAUDE.md`, for Codex `~/.codex/AGENTS.md`; name
your surface's equivalent). Declare it from the session; do not probe
other machines' filesystems — absence here is not absence everywhere.
Three topologies, three different answers:

- **The repository carries an instruction file** — with or without
  global rules loaded beside it: the repair list above has its home;
  make it true there.
- **No repository instruction file, but the session loaded user-level or
  machine-global rules.** Do not send repository facts there — a global
  file does not travel with the clone, a collaborator or a different
  agent surface never sees it, and project policy would pollute one
  person's private defaults. Propose — as a diff the owner agrees to — a
  minimal repository instruction file carrying only the operating floor:
  the five repairs above (they are the floor for a backlog repository),
  plus one line of commit discipline — work on the agreed branch, commit
  logical units, push only on the owner's explicit request. Global rules
  keep covering only what is genuinely generic (tone, language, personal
  style).
- **Nothing is loaded anywhere.** Propose the same minimal repository
  file with the same floor; it is the whole contract until the owner
  grows it.

Nothing else is injected in either repository-absent branch: no language
rules, no roles, no house style — those are the owner's to add or not.

**Offered, and bounded — the optimization pass.** While you are in these
files, *propose* improvements against these criteria — the ones that make an
instruction file cheap to load and hard to get wrong:

- **Routing over restating.** Its job is to say where things are and what
  binds behaviour, not to re-explain what a checked artefact already holds.
- **One fact, one home.** A fact stated twice drifts, and the second copy is
  the one that goes stale with nothing detecting it.
- **Point, do not duplicate.** Replace a restated procedure with a pointer to
  the file that owns it.
- **Name what must never be hand-edited**, and what regenerates it.
- **Cut what git history already carries.** How something came to be is
  history, not instruction.
- **Say what is enforced, and by what.** An instruction file asserting a
  guarantee nothing enforces is the most expensive error in this class: it
  reads as a safety net and is not one. Verify each such claim against live
  configuration before letting it stand.

**Anti-goals, and they matter.** This is not a licence to rewrite the
repository's agent contract wholesale, to import conventions from elsewhere,
or to impose a house style. Every change must trace to something the migration
made false, or to one of the criteria above. Propose a diff, cite what makes
each change true, and get the owner's agreement before applying it — a
repository's instruction file encodes decisions you were not present for.

## §6 — When the old format holds what the new one cannot express

This is the case that decides whether this schema is adequate for your
repository or merely adequate somewhere else. The rule: **never squash the
inexpressible into the nearest field.**

Stop, and write the case up concretely — the item, the field, exactly what
would be lost. Then choose, with a human:

1. **Extend the schema.** File it upstream as an issue; a field that several
   repositories need is a real gap, and the maintainer would rather hear it
   than have you fork quietly.
2. **Keep a local convention in the brief body.** Prose the tool does not check
   is honest as long as nothing claims it is checked.
3. **Leave that part on a named, bounded remnant of the old system**, with its
   retirement trigger written down.

An honest bounded remnant beats a silent lossy migration. Say which one you
chose and why, in the brief itself.

## Finish

- `park validate` and `park map --check` both exit 0.
- CI runs `park map --check`.
- Every agent instruction file is true about where the backlog lives, what is
  generated, and how to change it.
- Any wider instruction-file improvements were proposed with reasons and
  applied only with the owner's agreement.
- The index resolves to every migrated item; the archive holds the closed
  record intact.
- The old mechanism is retired, or its surviving parts are documented with
  reasons.
- Every decision the migration surfaced is a brief with an owner — open, not
  answered by you.
