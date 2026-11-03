# Upgrading documents and clients to format 2.1

The new tool reads 2.0.x and 2.1.x and writes 2.1.0 only. Reading a 2.0 package does not alter it. Its next successful save through the new writer upgrades it, even if the edit does not use a new feature. Older 2.0-only clients refuse the upgraded package. App build numbers, CLI contract versions and document format versions are separate identities; check the installed build and its supported formats.

## Shared documents

1. Identify every app, CLI, MCP server, bot and manual writer that can change the document, including other machines and copies in separate worktrees or synced folders. Agree on a pause window before upgrading writers or migrating files.
2. Back up the actual working state, including dirty/staged changes, untracked or local-only files and required LFS/submodule content. Save or otherwise retain unsaved app edits. A remote Git mirror does not contain all of this. Verify that backup files are readable; cloud placeholders alone are not a verified backup. Keep personal repositories private.
3. Stop writers and install the matching qualified release for each machine's architecture, including the executable and companion resources. The Mac app's embedded helper updates with the app; a standalone CLI or an explicit MCP path may still point elsewhere. Check the resolved executable, `--version` and archive provenance.
4. Restart the real MCP server and refresh the client session/catalog. Verify that the current tools are visible and that a scratch read/write works. Updating a shell command or restarting only the server does not prove a cached client catalog refreshed.
5. Try the migration on a backed-up copy first, using a supported ordinary edit/save through the new app or tool. Compare document/row/column IDs, CSV values, rowdoc bodies, README, saved views, metrics and other package files. Review any normalization/repair warning instead of silently combining a repair with migration. Do not export/reimport CSV or hand-edit the manifest version to simulate a migration. There is no dedicated public bulk-migration command.
6. With writers still paused, apply the reviewed change to the intended working copy, confirm its version and read it back. Publish the reviewed repository/sync change through the owner's usual process. Preserve unrelated edits. Resume one verified writer first, then the remaining writers for that document. Record deferred or failed documents separately; partial completion is not a completed rollout.

All shipped templates use 2.1. Previously created documents do not change just because their original template was updated. Historical evidence and old-format compatibility fixtures should remain identifiable and preserved rather than being swept into an automatic conversion.

## Recovery

Before any subsequent edits, a verified original working-state backup can restore the document while writers remain paused. Keep both versions and review any shared-history reversal normally.

Once new 2.1 edits exist, do not overwrite them with the old backup or change the version string to make an old client open the package. Preserve the complete current state and use a compatible client to repair/reconcile it. A downgrade would need an explicit conversion plan that retains new work and addresses unsupported definitions; no automatic downgrade is supplied. Rolling operators cannot safely be removed or reinterpreted silently.

The tool does not promise a transaction across documents, repositories or separate calls. Following an I/O failure or lost response, inspect the actual saved state before retrying. Keep per-document outcomes and recovery copies.

Formats older than 2.0 need a separate legacy conversion. Do not claim they were upgraded by a normal 2.1 save or by changing their manifest alone.
