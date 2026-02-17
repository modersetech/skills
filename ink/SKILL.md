# Ink CLI Skill

## Overview

Ink is a React renderer for building CLI (command-line interface) applications. It uses Yoga (Flexbox layout engine) to build terminal UIs with familiar React component patterns. If you know React, you already know Ink.

## When to Use This Skill

Use this skill when the user asks to:
- Build a CLI tool or terminal UI with React/JSX
- Create interactive command-line apps with components
- Render dynamic, styled terminal output
- Build CLIs with user input handling, focus management, and state

---

## Setup

```sh
npm install ink react
```

For TypeScript projects:
```sh
npx create-ink-app --typescript my-ink-cli
```

For quick scaffolding:
```sh
npx create-ink-app my-ink-cli
```

### Babel Setup (JavaScript)

```sh
npm install --save-dev @babel/preset-react
```

`babel.config.json`:
```json
{
  "presets": ["@babel/preset-react"]
}
```

---

## Basic Usage

```jsx
import React, { useState, useEffect } from 'react';
import { render, Text } from 'ink';

const Counter = () => {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCounter(prev => prev + 1);
    }, 100);
    return () => clearInterval(timer);
  }, []);

  return <Text color="green">{counter} tests passed</Text>;
};

render(<Counter />);
```

---

## App Lifecycle

- An Ink app stays alive as long as there is active work in the event loop.
- Exit via **Ctrl+C** (default), `exit()` from `useApp`, or `unmount()` from `render()`.

```jsx
const { waitUntilExit } = render(<MyApp />);
await waitUntilExit();
console.log('App exited');
```

---

## Core Components

### `<Text>`

Renders styled terminal text. All text **must** be wrapped in `<Text>`.

```jsx
import { render, Text } from 'ink';

render(
  <>
    <Text color="green">Green text</Text>
    <Text color="#ffffff">White hex</Text>
    <Text bold>Bold</Text>
    <Text italic>Italic</Text>
    <Text underline>Underline</Text>
    <Text strikethrough>Strikethrough</Text>
    <Text inverse>Inversed colors</Text>
    <Text color="red" dimColor>Dimmed red</Text>
    <Text backgroundColor="blue" color="white">Blue background</Text>
  </>
);
```

**Props:**
- `color` — string (named color, hex, or rgb)
- `backgroundColor` — string
- `dimColor` — boolean (default: false)
- `bold` — boolean
- `italic` — boolean
- `underline` — boolean
- `strikethrough` — boolean
- `inverse` — boolean
- `wrap` — `"wrap"` | `"truncate"` | `"truncate-start"` | `"truncate-middle"` | `"truncate-end"` (default: `"wrap"`)

```jsx
<Box width={7}>
  <Text wrap="truncate">Hello World</Text>
</Box>
//=> 'Hello…'
```

---

### `<Box>`

The core layout component — equivalent to `<div style="display: flex">`. Every element in Ink is a Flexbox container.

```jsx
import { render, Box, Text } from 'ink';

render(
  <Box margin={2} padding={1} flexDirection="column">
    <Text>Item 1</Text>
    <Text>Item 2</Text>
  </Box>
);
```

**Dimension Props:**
- `width`, `height` — number or `"50%"`
- `minWidth`, `minHeight` — number

**Padding Props:** `padding`, `paddingTop`, `paddingBottom`, `paddingLeft`, `paddingRight`, `paddingX`, `paddingY`

**Margin Props:** `margin`, `marginTop`, `marginBottom`, `marginLeft`, `marginRight`, `marginX`, `marginY`

**Gap Props:** `gap`, `columnGap`, `rowGap`

**Flex Props:**
- `flexDirection` — `"row"` | `"row-reverse"` | `"column"` | `"column-reverse"`
- `flexGrow` — number (default: 0)
- `flexShrink` — number (default: 1)
- `flexBasis` — number or string
- `flexWrap` — `"nowrap"` | `"wrap"` | `"wrap-reverse"`
- `alignItems` — `"flex-start"` | `"center"` | `"flex-end"`
- `alignSelf` — `"flex-start"` | `"center"` | `"flex-end"`
- `justifyContent` — `"flex-start"` | `"center"` | `"flex-end"` | `"space-between"` | `"space-around"`

**Border Props:**
- `borderStyle` — `"single"` | `"double"` | `"round"` | `"bold"` | `"singleDouble"` | `"doubleSingle"` | `"classic"`
- `borderColor` — string
- `borderTop`, `borderBottom`, `borderLeft`, `borderRight` — boolean

```jsx
<Box borderStyle="round" borderColor="green" padding={1}>
  <Text>Rounded green border</Text>
</Box>
```

**Overflow:** `overflow` — `"visible"` | `"hidden"`

---

### `<Newline>`

Adds a blank line (like `\n`).

```jsx
<Text>
  Line 1<Newline />
  Line 2
</Text>
```

**Props:** `count` — number of newlines (default: 1)

---

### `<Spacer>`

Fills all remaining space in a flex container (works like `flexGrow={1}`).

```jsx
<Box>
  <Text>Left</Text>
  <Spacer />
  <Text>Right</Text>
</Box>
```

---

### `<Static>`

Renders items permanently (they don't get re-rendered). Useful for logs or completed tasks.

```jsx
import { render, Static, Text, Box } from 'ink';

render(
  <Static items={completedTasks}>
    {task => (
      <Box key={task.id}>
        <Text color="green">✔ {task.title}</Text>
      </Box>
    )}
  </Static>
);
```

**Props:** `items` — array of items to render

---

### `<Transform>`

Transforms the string output of child components.

```jsx
<Transform transform={output => output.toUpperCase()}>
  <Text>hello world</Text>
</Transform>
// Output: HELLO WORLD
```

---

## Core Hooks

### `useInput(inputHandler, options?)`

Listens for keyboard input.

```jsx
import { useInput } from 'ink';

useInput((input, key) => {
  if (input === 'q') exit();
  if (key.upArrow) moveUp();
  if (key.downArrow) moveDown();
  if (key.return) select();
  if (key.escape) cancel();
  if (key.ctrl && input === 'c') exit();
});
```

**Key object properties:** `upArrow`, `downArrow`, `leftArrow`, `rightArrow`, `return`, `escape`, `ctrl`, `shift`, `tab`, `backspace`, `delete`, `meta`, `pageUp`, `pageDown`, `fn`

**Options:**
- `isActive` — boolean (default: true) — whether to listen for input

---

### `useApp()`

Returns `{ exit }` to programmatically exit the app.

```jsx
import { useApp } from 'ink';

const MyApp = () => {
  const { exit } = useApp();

  useEffect(() => {
    setTimeout(exit, 3000); // exit after 3 seconds
  }, []);

  return <Text>Running...</Text>;
};
```

---

### `useStdin()`

Access stdin stream and check if stdin is available.

```jsx
const { stdin, isRawModeSupported, setRawMode } = useStdin();
```

---

### `useStdout()`

Write directly to stdout, bypassing Ink's rendering.

```jsx
const { stdout, write } = useStdout();
write('Direct stdout output\n');
```

---

### `useStderr()`

Write directly to stderr.

```jsx
const { stderr, write } = useStderr();
write('Error message\n');
```

---

### `useFocus(options?)`

Manage focus for a component. Returns `{ isFocused }`.

```jsx
import { useFocus, Text } from 'ink';

const Item = ({ label }) => {
  const { isFocused } = useFocus();
  return <Text color={isFocused ? 'blue' : 'white'}>{label}</Text>;
};
```

**Options:**
- `isActive` — boolean (default: true)
- `autoFocus` — boolean (default: false)
- `id` — string

---

### `useFocusManager()`

Control focus programmatically.

```jsx
const { focusNext, focusPrevious, focus, disableFocus, enableFocus } = useFocusManager();
```

---

### `useCursor()`

Track and set cursor position.

```jsx
const { position, setPosition } = useCursor({ items, initialIndex: 0 });
```

---

## API: `render(tree, options?)`

Renders a React component tree to the terminal.

```jsx
import { render } from 'ink';

const { rerender, unmount, waitUntilExit, clear, cleanup } = render(<App />, {
  stdout: process.stdout,        // output stream (default: process.stdout)
  stdin: process.stdin,          // input stream (default: process.stdin)
  stderr: process.stderr,
  exitOnCtrlC: true,             // exit on Ctrl+C (default: true)
  patchConsole: true,            // patch console.log to render via Ink (default: true)
  debug: false,                  // don't clear frames (default: false)
});

// Rerender with new props or component
rerender(<App newProp="value" />);

// Unmount and exit
unmount();

// Wait until app is done
await waitUntilExit();

// Clear terminal output
clear();
```

---

## Screen Reader Support

```jsx
render(<MyApp />, { isScreenReaderEnabled: true });
// or: INK_SCREEN_READER=true node my-cli.js
```

**ARIA Props on `<Box>` and `<Text>`:**
- `aria-role` — `"button"` | `"checkbox"` | `"radio"` | `"list"` | `"listitem"` | `"progressbar"` | `"table"` | etc.
- `aria-state` — `{ checked, disabled, expanded, selected }` (booleans)
- `aria-label` — string
- `aria-hidden` — boolean (default: false)

```jsx
<Box aria-role="checkbox" aria-state={{ checked: true }}>
  <Text>Accept terms and conditions</Text>
</Box>
// Screen reader output: "(checked) checkbox: Accept terms and conditions"
```

---

## Testing

```jsx
import { render } from 'ink-testing-library';

test('counter', () => {
  const { lastFrame } = render(<Counter />);
  expect(lastFrame()).toContain('0 tests passed');
});
```

Install: `npm install --save-dev ink-testing-library`

---

## Continuous Integration

When `CI=true`, Ink only renders the last frame on exit. To opt out:

```sh
CI=false node my-cli.js
```

---

## Useful Third-Party Components

| Package | Description |
|---|---|
| `ink-text-input` | Text input field |
| `ink-spinner` | Loading spinner |
| `ink-select-input` | Dropdown/select |
| `ink-link` | Clickable terminal link |
| `ink-progress-bar` | Progress bar |
| `ink-table` | Table renderer |
| `ink-markdown` | Syntax-highlighted Markdown |
| `ink-task-list` | Task list with status |
| `ink-confirm-input` | Yes/No prompt |
| `ink-form` | Full form management |
| `ink-scroll-list` | Scrollable list |
| `ink-color-picker` | Color picker |
| `ink-use-stdout-dimensions` | React hook for terminal size |

---

## Best Practices

1. **Wrap all text in `<Text>`** — Never put raw strings inside `<Box>` or other non-Text components.
2. **Use `<Box>` for layout** — Every layout element is Flexbox; use `flexDirection`, `justifyContent`, and `alignItems` freely.
3. **Use `<Static>` for logs** — Items that won't change (like completed tasks or log lines) should use `<Static>` to prevent unnecessary re-renders.
4. **Use `useInput` for keyboard handling** — Prefer this hook over raw stdin for clean input management.
5. **Exit explicitly** — Use `useApp().exit()` or `unmount()` from `render()` to control when your CLI ends.
6. **Provide ARIA props** — For accessible CLIs, add `aria-role` and `aria-label` to interactive elements.
7. **CI mode** — Ink auto-detects CI environments; rely on `waitUntilExit()` for scripting use cases.

---

## Quick Example: Interactive Menu

```jsx
import React, { useState } from 'react';
import { render, Box, Text, useInput, useApp } from 'ink';

const ITEMS = ['Option A', 'Option B', 'Option C'];

const Menu = () => {
  const [selected, setSelected] = useState(0);
  const [confirmed, setConfirmed] = useState(null);
  const { exit } = useApp();

  useInput((input, key) => {
    if (key.upArrow) setSelected(prev => Math.max(0, prev - 1));
    if (key.downArrow) setSelected(prev => Math.min(ITEMS.length - 1, prev + 1));
    if (key.return) setConfirmed(ITEMS[selected]);
    if (input === 'q') exit();
  });

  if (confirmed) {
    return <Text color="green">You selected: {confirmed}</Text>;
  }

  return (
    <Box flexDirection="column">
      <Text bold>Choose an option (↑↓ to navigate, Enter to select, q to quit):</Text>
      {ITEMS.map((item, i) => (
        <Text key={item} color={i === selected ? 'cyan' : 'white'}>
          {i === selected ? '▶ ' : '  '}{item}
        </Text>
      ))}
    </Box>
  );
};

render(<Menu />);
```
