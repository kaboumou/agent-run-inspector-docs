# How to read an Agent Run Inspector report

This is the exact workflow and a field-by-field explanation of what the report
shows — and what it cannot show. It applies to the Agent Run Inspector 0.1.x line.

## The workflow

```
opencode export ses_abc123 > session.json     # tested with OpenCode 1.18.x
runinspector session.json                     # writes session-report.html (+ .json)
```

The HTML report is self-contained: open it in any browser, attach it to an
issue, or hand it to a teammate. The program reads only the input file, makes
no network requests, and never modifies your data.

## Findings and what they mean

### Repeated operations

Identical tool calls that appear more than once are flagged as candidates and
classified:

| Classification | Meaning | Typical cause |
|---|---|---|
| Failing retry | The same call repeated with no success between attempts | An external service or environment is down; the agent is backing off and retrying |
| Repeat after success | The call succeeded earlier and keeps being issued | Polling, verification, or a genuine loop |
| Recovered retry | Failures followed by a success | A transient wobble that healed |

The classification is evidence-based: a repeated call is labeled by what
happened around it, not guessed from intent.

### Failures

- **Recovered** — a failure whose operation later succeeded.
- **Unresolved** — a failure with no later success in the export.
- **Unknown** — missing evidence. The report says *unknown* rather than
  inventing a failure or a success.

### Completion state

The report ends with the run's state: *finished*, *finished with unresolved
items*, or *stopped mid-step / unknown* — derived from the final events in
the export, not from wall-clock time.

### Facts table

Tool usage counts, slowest calls, largest outputs, and provider-reported
token/cost figures. The token and cost numbers are **as reported by OpenCode
in the export** — useful for trend-finding, not invoice-grade accounting.

### Attached files, patches and compaction markers

Newer exports also contain non-tool events, and the report shows them in the
timeline:

- **Attached files** appear as `[attached file]` with the file name, MIME type
  and size. The embedded payload is summarized, never copied into the report —
  so the report stays shareable even when the export embeds file content.
- **Patches** appear as `[patch applied — N file(s) changed]` with the paths
  each patch touched. The **Files changed in this run** section aggregates all
  patched paths, so you can see the run's footprint at a glance.
- **Context compactions** appear as `[context compaction]` markers (with
  *automatic* / *context overflow* flags when the export records them). They
  mark points where the agent's context was compacted — useful when a run
  seems to "forget" earlier state.

These events have no status and are never counted as tool failures; they are
context, not findings.

## What the report cannot tell you

- Whether the run's *result* was correct — it reports mechanics, not semantics.
- Why a provider failed — it shows what the session recorded.
- Anything about other sessions: one export, one report. Exports of child
  sessions work the same way and show their parent id.

## Sharing safely

Everything from the export is HTML-escaped in the report, so no content can
execute. But the report *does contain your transcript content* — commands,
messages, paths. Review it before sharing, or share the JSON report and a
screenshot of the sections you need.
