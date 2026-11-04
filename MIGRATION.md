# Upgrading documents and clients to format 2.1

The new tool reads 2.0.x and 2.1.x and writes 2.1.0 only. Reading a 2.0 package does not alter it. A document without a persisted time context remains readable using the host calendar, identified as `evaluationContext.contextSource: "legacyHost"`; its ordinary mutations refuse until an explicit context upgrade. Older 2.0-only clients refuse the upgraded package. App build numbers, CLI contract versions and document format versions are separate identities; check the installed build and its supported formats.

## Choose the document's time context

The document owner must select the time zone and week start that its dates, saved views and metrics should use. Do not infer a city from a numeric offset or adopt a migration machine's zone automatically. Supported zones are `UTC` or supported slash-containing IANA names, such as `Europe/London`; bare abbreviations and fixed offsets are refused. The calendar is Gregorian, with four minimum days in the first week. Week start defaults to Monday on upgrade and can be set explicitly.

Read the existing context and current token first. For a document whose owner has selected UTC and Monday, the guarded CLI workflow is:

```sh
structable describe "$PWD/Project.structable" --configuration timeContext --json
structable manage-document "$PWD/Project.structable" \
  --action upgrade_time_context --time-zone UTC --week-starts-on monday \
  --expected-change-token '<current changeToken from describe>' --json
```

Replace UTC and Monday with the owner's chosen rules. The MCP equivalent is `structable_manage_document` with `action: "upgrade_time_context"`, `timeZone`, optional `weekStartsOn` and `expectedChangeToken`. This explicit operation saves the context and the 2.1 package; do it separately from other edits. Missing or invalid zones and stale guards save nothing. If a context already exists, use `set_time_context` with an explicit zone and a fresh token; an omitted week start then retains its existing value.

Context changes preserve stored date values and row Updated timestamps, but can change relative-filter membership and metric buckets. Review the context receipt's `temporalReviewDefinitions`, date-bearing views, metrics and any reported legacy repair. Legacy reads may have differed across hosts; agreement with one machine's old results is not proof of the owner's intended rules. Newly supplied or edited relative-date filters use canonical `datePeriod` windows or boundaries; unrelated saves preserve untouched legacy definitions. See the [date-period contract](README.md#document-time-context-and-date-periods).

A missing context is a readable upgrade state. A malformed present context, unknown condition type or malformed period is an error that refuses loading and preserves the original bytes. Correct a preserved recovery copy using the reported path instead of treating malformed data as a missing context.

## Shared documents

1. Identify every app, CLI, MCP server, bot and manual writer that can change the document, including other machines and copies in separate worktrees or synced folders. Agree on a pause window before upgrading writers or migrating files.
2. Back up the actual working state, including dirty/staged changes, untracked or local-only files and required LFS/submodule content. Save or otherwise retain unsaved app edits. A remote Git mirror does not contain all of this. Verify that backup files are readable; cloud placeholders alone are not a verified backup. Keep personal repositories private.
3. Stop writers and install the matching qualified release for each machine's architecture, including the executable and companion resources. The Mac app's embedded helper updates with the app; a standalone CLI or an explicit MCP path may still point elsewhere. Check the resolved executable, `--version` and archive provenance.
4. Restart the real MCP server and refresh the client session/catalog. Verify that the current tools are visible and that a scratch read/write works. Updating a shell command or restarting only the server does not prove a cached client catalog refreshed.
5. Try the explicit context upgrade on a backed-up copy first, using the owner-selected zone, week start and a fresh change token as shown above. Compare document/row/column IDs, CSV values, rowdoc bodies, README, saved views, metric definitions and results, and other package files. Review the saved context, date-bearing definitions and any normalization/repair warning; metric results may change under the selected context. Do not export/reimport CSV or hand-edit the manifest to simulate a migration. The context action handles one document; there is no public bulk-migration command.
6. With writers still paused, apply the reviewed change to the intended working copy, confirm its version and read it back. Publish the reviewed repository/sync change through the owner's usual process. Preserve unrelated edits. Resume one verified writer first, then the remaining writers for that document. Record deferred or failed documents separately; partial completion is not a completed rollout.

All shipped templates use 2.1 and an explicit UTC authoring context for repeatable sample data and metrics. Creating a document from a template captures the creator process's system zone once before interpreting dates, or uses an explicit `--time-zone`; it replaces the template's authoring context and defaults to Monday weeks. CSV/TSV import into a new document follows the same capture rule. Existing documents retain their saved context. Previously created documents do not change just because their original template was updated. Historical evidence and old-format compatibility fixtures should remain identifiable and preserved rather than being swept into an automatic conversion.

## Recovery

Before any subsequent edits, a verified original working-state backup can restore the document while writers remain paused. Keep both versions and review any shared-history reversal normally.

Once new 2.1 edits exist, do not overwrite them with the old backup or change the version string to make an old client open the package. Preserve the complete current state and use a compatible client to repair/reconcile it. A downgrade would need an explicit conversion plan that retains new work and addresses unsupported definitions; no automatic downgrade is supplied. Date-period definitions and document context cannot safely be removed or reinterpreted silently.

The tool does not promise a transaction across documents, repositories or separate calls. Following an I/O failure or lost response, inspect the actual saved state before retrying. Keep per-document outcomes and recovery copies.

Formats older than 2.0 need a separate legacy conversion. Do not claim they were upgraded by a normal 2.1 save or by changing their manifest alone.
