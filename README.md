# Structable CLI

**Better CSV files.**\
Typed columns. Stable row IDs. Markdown rowdocs for every row. Saved views and metrics in the file.\
Still plain text. Still in your repo. Still diffs clean.\
People get a native Mac app. Agents get MCP. The file needs neither, so the work carries on.

The free `structable` command and MCP server for Structable Documents, on macOS and Linux.

A Structable Document is a folder of plain files: `data.csv` for the rows, `schema.json` for the column types, Markdown rowdocs under `Rowdocs/`, `views.json` for saved views and `Metrics/` for metrics. [Structable](https://cynapse.com/structable) is the native Mac app. This tool lets scripts and AI agents read and write the same documents without the app, through typed values, stable row IDs and change tokens instead of raw file edits.

This repository holds the release downloads and these instructions. The tool's source code is not published here.

## What you get

One executable, `structable`, with three ways in:

- **Commands** for scripts and terminals: `describe`, `query`, `add-rows`, `update-rows`, `rowdocs`, `transfer` (CSV and TSV import and export) and the rest, 17 in all. Add `--json` for machine-readable output.
- **`structable mcp`**, a local stdio MCP server with the same 17 capabilities as tools, for AI agents.
- **`structable shell`**, an interactive session for exploring a document.

Reads and writes are free. They don't need the Structable app or a Structable Pro subscription; Pro is about creating and editing in the Mac app.

## Requirements

| Download | Runs on |
| --- | --- |
| `structable-macos-arm64.tar.gz` | macOS 27 or later on Apple silicon |
| `structable-linux-x86_64.tar.gz` | 64-bit Intel or AMD Linux with glibc 2.34 or later and libcurl, for example Ubuntu 22.04 or later, Debian 12 or later, Fedora, Red Hat Enterprise Linux 9 and compatible distributions, and Amazon Linux 2023 |

On Linux, `libcurl` comes with `curl`, so the install steps below already need it. A minimal container may lack it: install `libcurl4` (Debian and Ubuntu) or `libcurl` (Fedora and Red Hat). Linux on ARM and Alpine Linux aren't supported.

Each archive holds the `structable` executable, the resources folder it needs beside it, `BUILD-INFO.txt` (the release, source commit and compiler), `THIRD-PARTY-NOTICES.txt` and `TERMS.md`.

## Install

Both platforms follow the same steps: download the archive and the checksum list, check the archive against it, then copy the executable **and its resources folder** into the same directory. The executable can't run without that folder beside it. If any step fails, the rest don't run, so a failed download or check installs nothing.

These steps install into `~/.local/bin`. The commands download the latest release; to pin one, replace `latest/download` with `download/<tag>`, using a tag from the [Releases](https://github.com/cynapse/structable-cli/releases) page.

### macOS

```sh
(
set -e
cd "$(mktemp -d)"
base=https://github.com/cynapse/structable-cli/releases/latest/download
curl -fsSLO "$base/structable-macos-arm64.tar.gz"
curl -fsSLO "$base/SHA256SUMS"
grep ' structable-macos-arm64.tar.gz$' SHA256SUMS | shasum -a 256 -c -
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
cd "$(mktemp -d)"
base=https://github.com/cynapse/structable-cli/releases/latest/download
curl -fsSLO "$base/structable-linux-x86_64.tar.gz"
curl -fsSLO "$base/SHA256SUMS"
grep ' structable-linux-x86_64.tar.gz$' SHA256SUMS | sha256sum -c -
tar -xzf structable-linux-x86_64.tar.gz

mkdir -p "$HOME/.local/bin"
install -m 0755 structable-linux-x86_64/structable "$HOME/.local/bin/structable"
rm -rf "$HOME/.local/bin/StructableToolchain_StructableCore.resources"
cp -R structable-linux-x86_64/StructableToolchain_StructableCore.resources "$HOME/.local/bin/"
)
```

For every user of a machine, or in a container or cloud agent environment's setup script, run the same steps as root with `/usr/local/bin` in place of `$HOME/.local/bin`.

If you share the tool with other people, for example in a container image, keep `TERMS.md` and `THIRD-PARTY-NOTICES.txt` from the archive with it. The [terms](TERMS.md) require both.

### Check the install

```sh
structable --version
structable guide | head
```

If the shell can't find `structable`, add `~/.local/bin` to your `PATH` or use the full path. `structable --version` prints the tool's interface version (for example `0.1.0`), which doesn't change with every release; `BUILD-INFO.txt` in the archive names the exact release.

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

**Claude desktop and other MCP clients** that launch local stdio servers and read structured results or their JSON text mirror can use a configuration like this one. Replace the path with yours, since most clients don't expand `~`:

```json
{
  "mcpServers": {
    "structable": {
      "command": "/home/you/.local/bin/structable",
      "args": ["mcp"]
    }
  }
}
```

Every MCP tool result comes back in `structuredContent` and as full JSON text in `content`, so clients that forward only text can read it too. After you configure a client, restart it, start a new session and ask the agent to call `structable_guide` and summarize it. A summary of the guide confirms the agent can read results; seeing only a title means the setup needs checking.

The agent works where it runs. An agent on your computer can open the documents your account can open. A cloud agent needs the tool installed in its own environment, and it works on the files there, not on your computer.

## First commands

```sh
structable create-document --output-path "$PWD/Bugs.structable" --template builtin.bug-issue-tracker
structable add-rows "$PWD/Bugs.structable" --rows '[{"Title":"Export drops the last row","Status":"Inbox","Severity":"High"}]'
structable query "$PWD/Bugs.structable" --search export --select '["Title","Status","Severity"]'
structable describe "$PWD/Bugs.structable"
```

`structable guide` lists the templates, value encodings and filter shapes; `structable --help` and `structable <command> --help` explain each command. Paths must be absolute (a leading `~` is expanded).

## Update or uninstall

The tool doesn't update itself. To update, repeat the install steps with the new release; they replace the executable and its resources folder together.

To uninstall, delete both from the directory you installed into:

```sh
rm -rf "$HOME/.local/bin/structable" \
  "$HOME/.local/bin/StructableToolchain_StructableCore.bundle" \
  "$HOME/.local/bin/StructableToolchain_StructableCore.resources"
```

Then remove the server from any MCP client you registered it with, for example `claude mcp remove --scope user structable`.

## Good to know

- **Write through the tool.** It checks every value against the column's type and keeps row IDs, rowdocs and derived files consistent. Pass the `changeToken` from your last read as `--expected-change-token` (MCP: `expectedChangeToken`) so a write fails instead of overwriting someone else's newer change.
- **Failed writes.** A write that fails validation saves nothing. A write that fails partway through (an `io_failure`) can leave some changes saved, so read the document again before retrying. Keep backups or use version control for documents that matter.
- **File access.** The standalone tool can open whatever the account running it can open; it has no folder grants of its own. `discover` needs an explicit folder to list.
- **Linux.** [The Structable CLI on Linux](LINUX.md) says what the tool needs from the file system and what it does when a change can't finish.

## Terms

The tool is free to use under [these terms](TERMS.md).

## Privacy

The tool runs on your computer. It doesn't send your documents or information about your use to Cynapse. It works offline and has no update checks or analytics. An AI agent that runs it has its own terms and privacy practices.

## Help

- `structable --help`, `structable <command> --help` and `structable guide` document every command and value format.
- Report a problem or ask a question in this repository's [Issues](https://github.com/cynapse/structable-cli/issues). Include `BUILD-INFO.txt` from your archive, your platform, and the command or MCP tool call with its full error. Issues are public, so leave out anything sensitive, such as your documents' contents or private file paths.

Structable is a trademark of Cynapse.
