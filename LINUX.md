# The Structable CLI on Linux

The Linux build runs the same commands and MCP tools as the macOS build. This page says what it needs from the file system, and what it does when a change can't finish.

## What it needs

- **Atomic exclusive rename** (`RENAME_NOREPLACE`) to create a document or import into a new one, so the tool never replaces something already at that path.
- **Atomic directory exchange** (`RENAME_EXCHANGE`) to replace a whole document, so the new version takes the old one's place in one step.
- **A readable parent folder.** The tool locks the folder that contains a document while it reads and writes.

Both renames depend on the kernel and the file system. ext4 and overlayfs have both and are tested.

## When a rename isn't available

The tool refuses the change and leaves the destination as it was. Parent folders it created for a new document stay in place.

## After it replaces a whole document

The tool deletes the previous version once the new one is in place. If that cleanup fails, it reports the failure on standard error, and all or part of the previous version stays in a hidden sibling folder named `.<name>.structable.<UUID>.tmp`, with its original permissions and any data you removed. Each failed cleanup can leave another such folder. It's a leftover, not a backup, so don't restore it over the document. Your MCP client may not show standard error, so look beside the document for these folders even when no failure was reported. Delete one once you're sure you don't need what's in it.

## Without the Mac app

There's no Structable app on Linux. `create-document --open-in-app` creates the document and says in its result that there's no app to open it in.
