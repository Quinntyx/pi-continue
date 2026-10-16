# pi-continue

Adds `/continue` to Pi. It requests another assistant response using the current
session, without adding a user turn or sending a synthetic continuation prompt.

## Install

```sh
pi install git:git.quinntyx.dev/quinntyx/pi-continue@main
```

Run `/reload` in existing Pi sessions to load it. Pi clones the Git package into
its managed package cache; source development belongs in this repository, not
in the profile's `extensions/` directory.

## Usage

```text
/continue
```

Each continuation displays a dim **Continued from existing session** status line
in the transcript. The status is UI-only and never sent to the model.

No arguments are accepted. The session must contain a conversation, have a
selected model, and be idle with no queued messages. Busy sessions are not
interrupted or silently queued.

## How it works

Pi's public extension API does not expose a bare continuation method. This
extension uses `sendMessage(..., { triggerTurn: true })` to start Pi's standard
run pipeline with a custom control marker rendered as a compact status line.
Its `context` handler removes all such markers before conversion to provider
messages, including markers restored from earlier session history.

The marker is stored as a visible custom session entry, **not a user turn**.
It is never sent to the model. Existing messages remain unchanged, and Pi owns
streaming, tools, cancellation, and run settlement. Provider-specific handling
of consecutive assistant responses remains the provider adapter's responsibility.

## Tests

From this directory with Node.js 22.19 or newer:

```sh
npm ci --ignore-scripts
npm test
```

The development dependency pins Pi's SDK to the tested version. To test against
another installed Pi, set `PI_CONTINUE_HOST=/path/to/pi-coding-agent/dist/index.js`.
The extension uses Pi's host-provided packages at runtime; it does not bundle them.

The integration test loads the extension through Pi's real resource loader and
calls `/continue` twice through `session.prompt()`. A loopback HTTP provider
captures the actual serialized requests, checking unchanged transcript prefixes,
no new user turns, visible saved status entries, and no leaked control markers or
status text. Unit tests also exercise the real TUI text renderer at narrow and
wide terminal widths. Tests make no external requests.
