# Structable CLI

**Better CSV files.**\
Typed columns. Stable row IDs. Markdown rowdocs for every row. Saved views and metrics in the file.\
Still plain text. Still in your repo. Still diffs clean.\
People get a native Mac app. Agents get MCP. The file needs neither, so the work carries on.

The free `structable` command and MCP server for Structable Documents, on macOS and Linux.

A Structable Document is a folder of plain files: `data.csv` for the rows, `schema.json` for the column types, Markdown rowdocs under `Rowdocs/`, `views.json` for saved views and `Metrics/` for metrics. Structable is the native Mac app. This tool lets scripts and AI agents read and write the same documents without the app, through typed values, stable row IDs and change tokens instead of raw file edits.

This repository holds the release downloads and these instructions. The tool's source code is not published here.

## What you get

One executable, `structable`, with three ways in:

- **Commands** for scripts and terminals: `describe`, `query`, `add-rows`, `upsert-rows`, `update-rows`, `rowdocs`, `transfer` (CSV and TSV import and export) and the rest, 18 in all. Add `--json` for machine-readable output.
- **`structable mcp`**, a local stdio MCP server with the same 18 capabilities as tools, for AI agents.
- **`structable shell`**, an interactive session for exploring a document.

Reads and writes are free. They don't need the Structable app or a Structable Pro subscription; Pro is about creating and editing in the Mac app.

## Requirements

| Download | Runs on |
| --- | --- |
| `structable-macos-arm64.tar.gz` | macOS 27 or later on Apple silicon |
| `structable-linux-x86_64.tar.gz` | 64-bit Intel or AMD Linux with glibc 2.34 or later, libstdc++ from GCC 11 or later (`GLIBCXX_3.4.29`) and libcurl, for example Ubuntu 22.04 or later, Debian 12 or later, Fedora, Red Hat Enterprise Linux 9 and compatible distributions, and Amazon Linux 2023 |
| `structable-linux-arm64.tar.gz` | 64-bit ARM Linux with the same glibc, libstdc++ and libcurl requirements |

On Linux, `libcurl` comes with `curl`, which the install steps below already need, and `libstdc++` comes with nearly every distribution. A minimal container may lack them: install `libcurl4` and `libstdc++6` (Debian and Ubuntu) or `libcurl` and `libstdc++` (Fedora, Red Hat and Amazon Linux). Alpine Linux and other musl-based systems aren't supported. Use the archive for the architecture of the environment where the tool will run; a cloud agent needs its own installation.

Each archive holds the `structable` executable, its companion resources folder, `BUILD-INFO.txt` (the release, source commit and compiler), `THIRD-PARTY-NOTICES.txt` and `TERMS.md`.

## Document compatibility

This release reads package formats **2.0.x and 2.1.x** and saves **2.1.0**. Reading an older document does not change it. A document without a persisted time context remains readable using the host calendar, reported as `legacyHost`; ordinary writes refuse until its owner selects a time zone and explicitly upgrades it. Older 2.0-only clients refuse 2.1 documents, so update all app, CLI and MCP writers before resuming shared editing. There is no 2.0 write-compatibility mode. See [Upgrading documents and clients](MIGRATION.md) for the guarded upgrade workflow.

## Install

All downloads follow the same steps: download the archive and its checksum file, verify the archive, then copy the executable **and its resources folder** into the same directory. Commands that load bundled templates, including `guide` and template-based document creation, require that folder. Other commands can work without it.

The Mac and Linux x86_64 archives use `SHA256SUMS`. The Linux arm64 archive uses `structable-linux-arm64.tar.gz.sha256`. The commands below select the matching checksum file automatically.

Stop running CLI and MCP processes before installing or replacing the tool. These steps copy the executable and resources folder separately, so replacement is not atomic. A failed download or checksum check leaves the installation untouched; a later failure during replacement can leave an incomplete installation. Complete the installation before restarting the tool.

These steps install into `~/.local/bin`. The commands download the latest release; to pin one, replace `latest/download` with `download/<tag>`, using a tag from the [Releases](https://github.com/cynapse/structable-cli/releases) page.

### macOS

```sh
(
set -e
case "$(uname -s):$(uname -m)" in
    Darwin:arm64) ;;
    *) echo "This download requires macOS on Apple silicon." >&2; exit 1 ;;
esac
cd "$(mktemp -d)"
base=https://github.com/cynapse/structable-cli/releases/latest/download
curl -fsSLO "$base/structable-macos-arm64.tar.gz"
curl -fsSLO "$base/SHA256SUMS"
awk '$2 == "structable-macos-arm64.tar.gz" { if (NF != 2 || length($1) != 64 || $1 ~ /[^0123456789abcdefABCDEF]/) exit 1; printf "%s  %s\n", $1, $2; count++ } END { if (count != 1) exit 1 }' SHA256SUMS > selected.sha256
shasum -a 256 -c selected.sha256
tar -xzf structable-macos-arm64.tar.gz

mkdir -p "$HOME/.local/bin"
install -m 0755 structable-macos-arm64/structable "$HOME/.local/bin/structable"
rm -rf "$HOME/.local/bin/StructableToolchain_StructableCore.bundle"
cp -R structable-macos-arm64/StructableToolchain_StructableCore.bundle "$HOME/.local/bin/"
)
```

If you use the Structable app, you may not need this download: the app includes the same tool and its setup can install a `structable` command for you. The app's copy updates with the app and asks you to allow each work folder once; this standalone copy uses your account's file permissions. Keep one of them first on your `PATH`.

### Linux

```sh
(
set -e
case "$(uname -s):$(uname -m)" in
    Linux:x86_64) platform=linux-x86_64 ;;
    Linux:aarch64|Linux:arm64) platform=linux-arm64 ;;
    *) echo "This download requires Linux on x86_64 or arm64." >&2; exit 1 ;;
esac
payload=structable-$platform
asset=$payload.tar.gz
checksums=SHA256SUMS
if [ "$platform" = linux-arm64 ]; then checksums="$asset.sha256"; fi
cd "$(mktemp -d)"
base=https://github.com/cynapse/structable-cli/releases/latest/download
curl -fsSLO "$base/$asset"
curl -fsSL "$base/$checksums" -o SHA256SUMS
awk -v file="$asset" '$2 == file { if (NF != 2 || length($1) != 64 || $1 ~ /[^0123456789abcdefABCDEF]/) exit 1; printf "%s  %s\n", $1, $2; count++ } END { if (count != 1) exit 1 }' SHA256SUMS > selected.sha256
sha256sum -c selected.sha256
tar -xzf "$asset"

mkdir -p "$HOME/.local/bin"
install -m 0755 "$payload/structable" "$HOME/.local/bin/structable"
rm -rf "$HOME/.local/bin/StructableToolchain_StructableCore.resources"
cp -R "$payload/StructableToolchain_StructableCore.resources" "$HOME/.local/bin/"
)
```

The Linux steps choose x86_64 or arm64 from `uname -m` and refuse other architectures. A pinned older release may not contain the selected archive; a failed download leaves the existing installation untouched. These steps do not provide emulation or convert an installation for a different machine.

For every user of a machine, or in a container or cloud agent environment's setup script, run the same steps as root with `/usr/local/bin` in place of `$HOME/.local/bin`.

If you share the tool with other people, for example in a container image, keep `TERMS.md` and `THIRD-PARTY-NOTICES.txt` from the archive with it. The [terms](TERMS.md) require both.

### Check the install

```sh
structable --version
structable guide | head
```

If the shell can't find `structable`, add `~/.local/bin` to your `PATH` or use the full path. `structable --version` reports the CLI/MCP contract version with the packaged build, for example `0.1.0+1222`. Without valid build metadata, it reports the contract version alone, such as `0.1.0`. `BUILD-INFO.txt` records the release and source identity.

## Connect an AI agent over MCP

Register the server with its absolute path. The client starts `structable mcp` itself whenever it needs it.

**Claude Code**

```sh
claude mcp add --scope user structable -- "$HOME/.local/bin/structable" mcp
```

**Codex**

```sh
codex mcp add structable -- "$HOME/.local/bin/structable" mcp
```

**Claude desktop and other MCP clients** that launch local stdio servers and read structured results or their JSON text mirror can use a configuration like this one. Replace the path with yours (on Linux it starts with `/home/`), since most clients don't expand `~`:

```json
{
  "mcpServers": {
    "structable": {
      "command": "/Users/you/.local/bin/structable",
      "args": ["mcp"]
    }
  }
}
```

Every MCP tool result comes back in `structuredContent` and as full JSON text in `content`, so clients that forward only text can read it too. After you configure a client, restart it, start a new session and ask the agent to call `structable_guide` and summarize it. A summary of the guide confirms the agent can read results; seeing only a title means the setup needs checking.

After an upgrade, check the actual server executable as well as your shell command. Some clients cache tool catalogs: restart the server and refresh the client session/catalog, then verify the new tools are visible and call the guide. A current server with an old client catalog is not a verified upgrade.

The agent works where it runs. An agent on your computer can open the documents your account can open. A cloud agent needs the tool installed in its own environment, and it works on the files there, not on your computer.

## First commands

```sh
structable create-document --output-path "$PWD/Bugs.structable" --template builtin.bug-issue-tracker
structable add-rows "$PWD/Bugs.structable" --rows '[{"Title":"Export drops the last row","Status":"Inbox","Severity":"High"}]'
structable query "$PWD/Bugs.structable" --search export --select '["Title","Status","Severity"]'
structable describe "$PWD/Bugs.structable"
```

`structable guide` lists the templates, value encodings and filter shapes; `structable --help` and `structable <command> --help` explain each command. Paths must be absolute (a leading `~` is expanded).

## Exact RowID upsert

`upsert-rows` updates existing exact IDs and creates missing exact IDs in one request:

```sh
structable upsert-rows "$PWD/Bugs.structable" --rows '[{"rowID":"issue001","values":{"Title":"Check imported rows","Status":"Inbox"}},{"rowID":"issue002","values":{"Title":"Review saved filters"}}]' --json
```

The MCP tool is `structable_upsert_rows`. Supply 1–1,000 unique `{rowID, values}` entries, inline or as a JSON file with `--rows @/absolute/rows.json`. IDs must be canonical lowercase alphanumeric strings; new IDs must also fit the canonical `ID.md` filename. Existing row order stays unchanged; new rows append in request order. The result includes ordered `rowOutcomes` (`created`, `updated`, `unchanged`) and `upsertCounts`.

Omitted cells stay unchanged, `null` clears ordinary cells, and an empty `values` map ensures the row exists. New rows receive the normal creation defaults and managed timestamps. Use `add-rows` for generated IDs. ID renaming, timestamp writes and rowdoc fields are refused; read/write rowdoc bodies separately with `rowdocs`. A later rowdoc failure does not undo a completed upsert.

Use the current `--expected-change-token` when the write depends on a read. Validation failure saves none of the request; save failures may leave partial or uncertain results, so reread and reconcile before retrying. Splitting larger work across calls does not make it one transaction.

CSV/TSV `transfer` imports still create a document or append rows. They do not perform upsert or match records by another column. For an import workflow that needs upsert, map the records to the explicit RowID JSON shape first.

## Document time context and date periods

Each configured document persists a Gregorian calendar, a named time zone and its week start. New `create-document` and `transfer --action import_create` operations capture the process's system zone once, before interpreting dates, and default to Monday weeks. Use `--time-zone` to choose a supported IANA name such as `Europe/London`, or `UTC`; results report the saved `timeContext`. If the system zone cannot be represented, supply an explicit zone. Built-in templates are authored in UTC for repeatability, but creating a document from one replaces that authoring context with the captured or explicitly selected creation context. Later saves retain the document's context.

Read the context with `structable describe /absolute/Document.structable --configuration timeContext --json`. For a missing context, use `manage-document --action upgrade_time_context`; for an already configured document, use `set_time_context`. Both require an explicit `--time-zone` and current `--expected-change-token`, and run separately from other edits. Optional `--week-starts-on` selects a weekday; omission means Monday on upgrade or the existing week start on a configured document. See [migration](MIGRATION.md) before choosing a context for an existing shared document.

Canonical relative-date conditions use `datePeriod` in queries, saved views and metric filters. This filter selects the last seven calendar days including today:

```json
{"combination":"and","filters":[{"columnId":"Target Date","condition":{"type":"datePeriod","period":{"kind":"window","unit":"day","count":7,"endOffset":0,"extent":"complete"}}}]}
```

A window contains `count` consecutive units ending at `endOffset` relative to the current unit. Set `endOffset` to `-7` in that example for the seven days preceding the most recent seven. Units are `day`, `week`, `month`, `quarter` and `year`; integer `count` is 1–10,000 and offsets are −10,000–10,000. `extent: "complete"` includes whole units. For this month through today, use `unit: "month"`, `count: 1`, `endOffset: 0` and `extent: "throughCorrespondingDay"`. That extent permits one non-day unit with a nonpositive offset and includes the entire corresponding day, including future timestamps today. It does not mean up to the current instant.

Open-ended conditions use a boundary, for example `{"type":"datePeriod","period":{"kind":"boundary","unit":"day","offset":0,"relation":"onOrAfter"}}` for today and later. `relation: "before"` selects values before the boundary. Boundaries for larger units use the start of the selected week, month, quarter or year.

Periods use the document's zone and week rules, with inclusive lower and exclusive upper bounds, rather than fixed 24-hour durations. Date-only values remain civil dates; timed values use instants. Query results expose `evaluationContext` and bounded `resolvedTemporalFilters`; `resolvedTemporalFilterCount` and `temporalContextTruncated` disclose omitted evidence. Missing-context reads use `legacyHost` and can differ between machines. Legacy filter inputs, including `dateNextDays`, `dateLastDays`, `dateOnOrAfterToday` and `dateBeforeToday`, remain accepted; newly supplied or edited definitions persist canonical periods, while unrelated saves preserve untouched definitions.

Malformed persisted periods, unknown condition types and malformed present contexts refuse loading with a reported path, preserving the original bytes. Unsupported resolution outside Gregorian years 1583–9999 is an explicit error. Preserve a recovery copy and correct the reported definition deliberately. `structable guide` documents the complete filter shapes and constraints.

## Document descriptions

Read the package's README with `structable describe /absolute/Document.structable --configuration description --json`. The JSON result includes the body guard and continuation metadata; follow every returned page before replacing the body. `bodyDigest` covers the whole canonical description, even when only one page is returned; a fresh digest is not proof that you read the whole body.

`manage-document` provides `set_description`, `append_description` and `clear_description`. For example, given an existing UTF-8 Markdown file:

```sh
structable manage-document "$PWD/Bugs.structable" --action set_description --source-path "$PWD/purpose.md"
```

Set accepts one inline value (`--description` or `--content`) or `--source-path`. Set/clear support `--expected-body-digest` as an optional body guard. Append requires **both** `--expected-change-token` and `--expected-body-digest`; it appends the literal fragment to the full server-side body and adds no separator. Include any desired newline yourself, and reread after an uncertain write rather than replaying blindly. The same actions are available through `structable_manage_document`.

Descriptions keep their existing README semantics: CRLF/CR become LF and whitespace-only content is absent. Rowdoc bodies remain exact text. File input and append fragments are limited to 4 MiB; existing larger descriptions remain readable and appendable. UTF-8 BOM characters are preserved. These operations do not introduce a new description storage format.

## Update or uninstall

The tool doesn't update itself. To update, stop running CLI and MCP processes, then repeat the install steps with the new release. The executable and resources folder are replaced separately; finish the installation and check it before restarting your clients.

To uninstall, delete both from the directory you installed into:

```sh
rm -rf "$HOME/.local/bin/structable" \
  "$HOME/.local/bin/StructableToolchain_StructableCore.bundle" \
  "$HOME/.local/bin/StructableToolchain_StructableCore.resources"
```

Then remove the server from any MCP client you registered it with, for example `claude mcp remove --scope user structable`.

## Good to know

- **Write through the tool.** It checks values against column types and keeps row IDs, rowdocs and derived files consistent. Cooperating Structable tools serialize writes to a document. When a write depends on an earlier read, pass that read's `changeToken` as `--expected-change-token` (MCP: `expectedChangeToken`). If the token is stale, read the current document and reconcile the change before retrying. A workflow spanning several calls is not one transaction; direct file edits do not participate in tool locking.
- **Failed writes.** A write that fails validation saves nothing. An `io_failure` can mean changes were saved before the failure. Follow its recovery instructions; if changes may have been saved, read the document and affected rowdocs again before retrying. Keep backups or use version control for documents that matter.
- **File access.** The standalone tool can open whatever the account running it can open; it has no folder grants of its own. `discover` needs an explicit folder to list.
- **Linux.** [The Structable CLI on Linux](LINUX.md) says what the tool needs from the file system and what it does when a change can't finish.

## Terms

The tool is free to use under [these terms](TERMS.md).

## Privacy

The tool runs on your computer. It doesn't send your documents or information about your use to Cynapse. It works offline and has no update checks or analytics. An AI agent that runs it has its own terms and privacy practices.

## Help

- `structable --help`, `structable <command> --help` and `structable guide` document every command and value format.
- Report a problem or ask a question in this repository's [Issues](https://github.com/cynapse/structable-cli/issues). Include `BUILD-INFO.txt` from your archive, your platform, and the command or MCP tool call with its full error. Issues are public, so leave out anything sensitive, such as your documents' contents or private file paths.

Structable is a Cynapse invention
