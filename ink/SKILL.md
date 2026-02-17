---
name: ink
description: >
  Use this skill whenever the user wants to build a CLI (command-line interface) tool or terminal UI
  using React/JSX. Triggers include: "build a CLI", "terminal UI", "interactive command-line app",
  "CLI tool with components", or any mention of the Ink library. Also trigger when the user wants
  to render dynamic styled output in the terminal, handle keyboard input in a CLI, or build a
  multi-component terminal layout. Use this skill even if the user just mentions "React for the
  terminal" or wants to add interactivity (spinners, prompts, menus, progress bars) to a Node.js CLI.
---

# Ink — React for CLIs

## Overview

Ink lets you build and test CLI output using React components. It uses Yoga (Flexbox) to layout
terminal UIs with familiar CSS-like props. If you know React, you already know Ink.

---

## Setup

```sh
npm install ink react
```

Quick scaffold:
```sh
npx create-ink-app my-ink-cli
# TypeScript:
npx create-ink-app --typescript my-ink-cli
```

**Babel (JS only):**
```sh
npm install --save-dev @babel/preset-react
```
`babel.config.json`:
```json
{ "presets": ["@babel/preset-react"] }
```

---

## Workflow Decision Tree

```
User wants to...
├── Display static styled text → <Text> with color/bold/italic props
├── Layout multiple elements → <Box> with flexbox props
├── Handle keyboard input → useInput hook
├── Exit the app programmatically → useApp().exit()
├── Show a log of completed items → <Static> component
├── Fill remaining space → <Spacer>
├── Render permanent newlines → <Newline>
└── Transform output strings → <Transform>
```

---

## Core Components

### `<Text>` — All terminal text must be wrapped here

```jsx
import { render, Text } from 'ink';

render(
  <>
    <Text color="green">Green</Text>
    <Text color="#ff6b6b">Hex color</Text>
    <Text bold italic underline>Styled</Text>
    <Text strikethrough>Removed</Text>
    <Text inverse>Flipped colors</Text>
    <Text color="red" dimColor>Dimmed</Text>
    <Text backgroundColor="blue" color="white">BG color</Text>
  </>
);
```

**Key props:** `color`, `backgroundColor`, `dimColor`, `bold`, `italic`, `underline`,
`strikethrough`, `inverse`, `wrap` (`"wrap"` | `"truncate"` | `"truncate-start"` |
`"truncate-middle"` | `"truncate-end"`)

> ⚠️ `<Box>` cannot be nested inside `<Text>`. Only text nodes and other `<Text>` components.

---

### `<Box>` — Flexbox layout container (like `<div style="display:flex">`)

```jsx
import { render, Box, Text } from 'ink';

render(
  <Box flexDirection="column" padding={1} gap={1}>
    <Box justifyContent="space-between">
      <Text>Left</Text>
      <Text>Right</Text>
    </Box>
    <Box borderStyle="round" borderColor="cyan" padding={1}>
      <Text>Bordered box</Text>
    </Box>
  </Box>
);
```

**Dimension:** `width`, `height`, `minWidth`, `minHeight` (number or `"50%"`)

**Spacing:** `padding`, `paddingTop/Bottom/Left/Right`, `paddingX`, `paddingY`,
`margin`, `marginTop/Bottom/Left/Right`, `marginX`, `marginY`

**Gap:** `gap`, `columnGap`, `rowGap`

**Flex:** `flexDirection` (`row`|`column`|`row-reverse`|`column-reverse`), `flexGrow`,
`flexShrink`, `flexBasis`, `flexWrap`, `alignItems`, `alignSelf`, `justifyContent`

**Border:** `borderStyle` (`single`|`double`|`round`|`bold`|`classic`|`singleDouble`|`doubleSingle`),
`borderColor`, `borderTop/Bottom/Left/Right` (boolean)

**Overflow:** `overflow` (`visible`|`hidden`)

---

### `<Newline>`, `<Spacer>`, `<Static>`, `<Transform>`

```jsx
// Newline — adds blank lines
<Text>Line 1<Newline />Line 2</Text>
<Newline count={2} />

// Spacer — fills remaining flex space
<Box>
  <Text>Left</Text>
  <Spacer />
  <Text>Right</Text>
</Box>

// Static — renders items permanently (great for logs/completed tasks)
<Static items={completedTasks}>
  {task => <Text key={task.id} color="green">✔ {task.title}</Text>}
</Static>

// Transform — transforms string output of children
<Transform transform={output => output.toUpperCase()}>
  <Text>hello</Text>
</Transform>
// => HELLO
```

---

## Core Hooks

### `useInput` — Keyboard input handling

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

**`key` properties:** `upArrow`, `downArrow`, `leftArrow`, `rightArrow`, `return`, `escape`,
`ctrl`, `shift`, `tab`, `backspace`, `delete`, `meta`, `pageUp`, `pageDown`, `fn`

**Options:** `{ isActive: boolean }` — set `false` to pause listening

---

### `useApp` — Programmatic exit

```jsx
const { exit } = useApp();
// exit() with no args = success; exit(new Error('reason')) = failure
```

### `useFocus` — Component focus state

```jsx
const { isFocused } = useFocus({ autoFocus: true, id: 'my-input' });
```

### `useFocusManager` — Control focus programmatically

```jsx
const { focusNext, focusPrevious, focus, disableFocus, enableFocus } = useFocusManager();
```

### `useStdout` / `useStderr` — Direct stream writes (bypass Ink rendering)

```jsx
const { write } = useStdout();
write('Direct output bypassing Ink\n');
```

---

## App Lifecycle & `render()`

```jsx
import { render } from 'ink';

const { rerender, unmount, waitUntilExit, clear } = render(<App />, {
  stdout: process.stdout,   // default
  stdin: process.stdin,     // default
  exitOnCtrlC: true,        // default
  patchConsole: true,       // default — patch console.log through Ink
  debug: false,             // if true, don't clear frames
});

await waitUntilExit();    // wait for app to finish
rerender(<App newProp />); // update with new props
unmount();                 // force exit
clear();                   // clear terminal output
```

App lifecycle note: the process stays alive as long as there is active work (timers, promises,
`useInput` on stdin). A component tree with no async work renders once then exits immediately.

---

## Accessibility (Screen Readers)

```jsx
render(<App />, { isScreenReaderEnabled: true });
// or set env: INK_SCREEN_READER=true

<Box aria-role="checkbox" aria-state={{ checked: true }}>
  <Text>Accept terms</Text>
</Box>
// Screen reader output: "(checked) checkbox: Accept terms"
```

**ARIA props** on `<Box>` / `<Text>`:
- `aria-role` — `button`|`checkbox`|`radio`|`list`|`listitem`|`progressbar`|`table`|etc.
- `aria-state` — `{ checked, disabled, expanded, selected }` (booleans)
- `aria-label` — string (custom label for screen readers)
- `aria-hidden` — boolean (default: false)

---

## Testing

```sh
npm install --save-dev ink-testing-library
```

```jsx
import { render } from 'ink-testing-library';

test('renders correctly', () => {
  const { lastFrame } = render(<MyComponent />);
  expect(lastFrame()).toContain('expected text');
});
```

---

## CI Mode

In CI environments (`CI=true`), Ink renders only the last frame on exit. To opt out: `CI=false node my-cli.js`

---

## Useful Third-Party Packages

| Package | Description |
|---|---|
| `ink-text-input` | Text input field |
| `ink-spinner` | Loading spinner |
| `ink-select-input` | Dropdown/select menu |
| `ink-progress-bar` | Progress bar |
| `ink-table` | Table renderer |
| `ink-task-list` | Task list with status icons |
| `ink-confirm-input` | Yes/No prompt |
| `ink-form` | Full form management |
| `ink-markdown` | Syntax-highlighted Markdown |
| `ink-scroll-list` | Scrollable list |
| `ink-link` | Clickable terminal hyperlink |
| `ink-use-stdout-dimensions` | Hook for terminal dimensions |

---

## Complete Example: Interactive Menu

```jsx
import React, { useState } from 'react';
import { render, Box, Text, Newline, useInput, useApp } from 'ink';

const ITEMS = ['Option A', 'Option B', 'Option C'];

const Menu = () => {
  const [selected, setSelected] = useState(0);
  const [confirmed, setConfirmed] = useState(null);
  const { exit } = useApp();

  useInput((input, key) => {
    if (key.upArrow) setSelected(i => Math.max(0, i - 1));
    if (key.downArrow) setSelected(i => Math.min(ITEMS.length - 1, i + 1));
    if (key.return) setConfirmed(ITEMS[selected]);
    if (input === 'q') exit();
  });

  if (confirmed) {
    return <Text color="green">✔ Selected: {confirmed}</Text>;
  }

  return (
    <Box flexDirection="column" padding={1} borderStyle="round" borderColor="cyan">
      <Text bold>Choose an option</Text>
      <Text dimColor>↑↓ navigate · Enter select · q quit</Text>
      <Newline />
      {ITEMS.map((item, i) => (
        <Text key={item} color={i === selected ? 'cyan' : undefined}>
          {i === selected ? '▶ ' : '  '}{item}
        </Text>
      ))}
    </Box>
  );
};

render(<Menu />);
```

---

## Best Practices

- **Every text string must live in `<Text>`** — don't put raw strings directly in `<Box>`.
- **Layout = `<Box>`** — use `flexDirection`, `justifyContent`, `alignItems` freely; everything is Flexbox.
- **Static content** — use `<Static>` for logs or completed task lines that don't need re-rendering.
- **Exit explicitly** — use `useApp().exit()` or `unmount()` so the process doesn't hang.
- **Add ARIA props** — use `aria-role` and `aria-label` on interactive elements for accessibility.
- **CI-safe** — always test with `CI=true` if shipping to pipelines; rely on `waitUntilExit()` for scripting.
