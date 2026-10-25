# Xterm.js Readline
[![node ci](https://github.com/strtok/xterm-readline/actions/workflows/node.js.yml/badge.svg)](https://github.com/strtok/xterm-readline/actions/workflows/node.js.yml)

This is an xterm-js addon that provides a readline like capability to xterm-js. It allows the creation of cli or repl like interfaces on top of xterm-js.

Large parts of this source are inspired / derived / ported from the MIT licensed [rustyline](https://github.com/kkawakam/rustyline) project.

# Features

* Multi-line prompt and input
* Extendable syntax highlighting
* Unicode / multi column-width character support

# Example

This is a multi-line example that treats any line ending with '&&' as a continuation.

```typescript
import "xterm/css/xterm.css";
import { Terminal } from 'xterm';
import { Readline } from "xterm-readline";

const term = new Terminal({
  theme: {
        background: "#191A19",
        foreground: "#F5F2E7",
  },
  cursorBlink: true,
  cursorStyle: "block"
});

const rl = new Readline();

term.loadAddon(rl);
term.open(document.getElementById('terminal'));
term.focus();

rl.setCheckHandler((text) => {
  let trimmedText = text.trimEnd();
  if (trimmedText.endsWith("&&")) {
    return false;
  }
  return true;
});

function readLine() {
  rl.read(">")
    .then(processLine);
}

function processLine(text) {
  rl.println("you entered: " + text);
  setTimeout(readLine);
}

readLine();
```

# Cancelling a read

Pass an optional `AbortSignal` to cancel a pending read. The caller owns the
controller; the Ctrl-C handler can use it to abort the read:

```ts
const controller = new AbortController();
rl.setCtrlCHandler(() => controller.abort());

try {
  const input = await rl.read("> ", { signal: controller.signal });
  // Use input.
} catch (error) {
  if (!controller.signal.aborted || error !== controller.signal.reason) {
    throw error;
  }
  // Cancelled. The caller decides whether to start another read.
}
```

Aborting rejects with `signal.reason`. Calling `abort()` without a reason uses
a `DOMException` named `AbortError`; custom reasons are preserved. An already
aborted signal rejects without printing a prompt. Use a fresh controller for
each cancellation scope. Only one read can be active at a time; overlapping
calls reject instead of replacing the pending read.

The Ctrl-C handler now runs both during and between reads, including while a
prompt is being printed. If the handler aborts the read, input is discarded and
no replacement prompt is printed. Output already written or queued to xterm is
left intact; cancellation does not add a newline or `^C`. If the handler does
not abort, Ctrl-C retains its existing behavior: print `^C`, clear input, and
reprint the prompt while leaving the same read promise pending.

A signal can also be aborted by a Cancel button or another application action.
Cancelling a read does not automatically cancel unrelated work; other operations
must observe a signal themselves. Registering an aborting Ctrl-C handler affects
its controller between reads too, so applications should manage that handler's
lifetime along with the operation it cancels.

# Keyboard Shortcuts

Key             | Action
---------       | ------
Home            | Move cursor to the beginning of line
End             | Move cursor to end of line
Left            | Move cursor one character left
Right           | Move cursor one character right
Up              | Move cursor up (multi-line edit)
Down            | Move cursor down (multi-line edit)
Ctrl-C          | Call the registered handler; unless it aborts the read, clear input and reprint the prompt
Ctrl-D, Del     | Delete the character under the cursor
Enter           | Apply the line, or begin a new line in multiline mode
Alt/Shift Enter | Force add \n to the input
Ctrl-U          | Clear line
Ctrl-K          | Delete text from cursor to end of line

# Demo

A small paren-balanced REPL under `demo/` for manual testing. Hot-reloads
on `src/` edits.

```
npm install
npm run demo
```

# Real World Uses

* [Marwood](https://github.com/strtok/marwood)

# License Agreement

If you contribute code to this project, you implicitly allow your code to be distributed under the MIT license.
