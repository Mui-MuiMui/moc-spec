# .moc File Format Specification

[日本語](README_JP.md)

- **Version:** 1.0.0
- **Status:** Official Specification

---

## 1. Overview

- The `.moc` file is a definition format for GUI mockups created with the VSCode extension "Mocker."
- Based on TSX (TypeScript JSX), it is an integrated data format designed for seamless collaboration between **Humans**, **GUIs**, and **AI Agents**.

### Tech Stack

- **CSS**: Tailwind CSS v4 (Utility-first CSS)
- **UI Components**: shadcn/ui (Radix UI + Tailwind CSS)
- Import path `@/components/ui/*` refers to shadcn/ui components.
- If the target environment does not support Tailwind CSS or shadcn/ui, reproduce equivalent visuals and layouts using the available tech stack.

### Design Principles

- **Human Readability**: The TSX portion remains directly readable as a standard React component.
- **AI Readability**: Metadata, memos, and editor data are all stored as plain text for easy AI parsing.
- **SSOT (Single Source of Truth)**: If GUI editor state (`craftState`) exists, it takes precedence over the TSX code. The TSX is treated as derivative data generated from the `craftState`.

---

## 2. File Structure

A `.moc` file consists of four main sections:

```
┌─────────────────────────────────┐
│ 1. JSDoc Metadata Block         │  Required
│    (@moc-* tags)                │
├─────────────────────────────────┤
│ 2. Import Block                 │  Optional
│    (ES module import statements)│
├─────────────────────────────────┤
│ 3. TSX Component                │  Required
│    (export default function)    │
├─────────────────────────────────┤
│ 4. Editor Data Block            │  Optional
│    (const __mocEditorData = ``) │
└─────────────────────────────────┘
```

---

## 3. Metadata Block (JSDoc Comment)

Metadata is written using `@moc-*` tags inside a JSDoc comment at the top of the file.

### 3.1 Header Description

```typescript
/**
 * Mocker Document (.moc)
 * A definition file for GUI mockups created with the VSCode extension "Mocker."
 * An integrated data format for collaborative development between Humans, GUIs, and AI Agents.
 *
 * File Structure:
 *   This file is based on TSX (TypeScript JSX).
 *   Metadata and sticky-note memos are stored inside JSDoc comments,
 *   and the internal editor state is stored in a template literal variable.
 *   The TSX portion is directly readable as a React component.
 *
 * Tech Stack:
 *   CSS: Tailwind CSS v4 (Utility-first CSS)
 *   UI Components: shadcn/ui (Radix UI + Tailwind CSS)
 *   The import path "@/components/ui/*" refers to shadcn/ui components.
 *   If the target environment does not support Tailwind CSS or shadcn/ui,
 *   reproduce equivalent visuals and layouts using the available tech stack.
 *
 * SSOT (Single Source of Truth):
 *   If GUI editor state (craftState) exists in the editor data at the end of the file,
 *   craftState takes precedence over the TSX code.
 *   TSX is derivative data auto-generated from craftState, provided as a readable
 *   reference for AI agents. To change the component layout, use the GUI editor.
 *   (When edited via GUI, the craftState content is also overwritten into the TSX.)
 *
 * AI Reading Priority:
 *   1. TSX code (primary source for structure and layout; for human/AI readability)
 *   2. craftState (reference for detailed GUI editor properties)
 *
 * Metadata:
 *   @moc-version  - Document format version (required)
 *   @moc-intent   - Purpose and intent of this page (optional, written by humans)
 *   @moc-theme    - Theme (light | dark) (optional, default: light)
 *   @moc-layout   - Layout mode (flow | absolute) (optional, default: flow)
 *   @moc-viewport - Target viewport (desktop | tablet | mobile | WxH) (optional, default: desktop)
 *
 * AI Instruction Memos:
 *   Sticky-note instructions placed by the user on the canvas for AI agents.
 *   Each memo is written with a @moc-memo tag as a pair of target element ID and instruction text.
 *   The AI should read these memos and apply corrections/suggestions to the corresponding elements.
 *
 * TSX Comment Conventions:
 *   @moc-node <nodeID>  - Associates the element with a Craft.js node
 *   @moc-role <role>    - Describes the role of the element
 *   @moc-memo <memo>    - Summary of a sticky-note memo
 */
```

### 3.2 Metadata Tags

| Tag | Required | Type | Default | Description |
|-----|----------|------|---------|-------------|
| `@moc-version` | **Required** | `string` | - | Document format version (e.g., `1.0.0`) |
| `@moc-intent` | Optional | `string` | `""` | Purpose and intent of the page |
| `@moc-theme` | Optional | `"light" \| "dark"` | `"light"` | Theme |
| `@moc-layout` | Optional | `"flow" \| "absolute"` | `"flow"` | Layout mode |
| `@moc-viewport` | Optional | `string` | `"desktop"` | `desktop`, `tablet`, `mobile`, or `WxH` format |
| `@moc-memo` | Optional | - | - | AI instruction memo (multiple allowed) |

### 3.3 AI Instruction Memos (@moc-memo)

```
@moc-memo #<targetElementId> "instruction text"
```

- `#<targetElementId>`: ID that identifies an element within the TSX (the `id` attribute value)
- `"instruction text"`: Instructions for the AI agent (enclosed in double quotes)
- Multiple memos can be specified

---

## 4. Import Block

Standard ES module import statements.

```typescript
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
```

---

## 5. TSX Component

A React component written in `export default function` format.

```typescript
export default function LoginForm() {
  return (
    <>
      {/* @moc-node abc123 */}
      {/* @moc-role "Login Form" */}
      <div className="flex min-h-screen items-center justify-center">
        {/* @moc-node def456 */}
        {/* @moc-memo "Email validation required" */}
        <Input id="emailInput" type="email" placeholder="Email" />
      </div>
    </>
  );
}
```

### 5.1 TSX Comment Conventions

When the GUI editor generates TSX, it attaches the following comments to each element:

| Comment | Description |
|---------|-------------|
| `{/* @moc-node <nodeID> */}` | Associates with a Craft.js node ID. Attached to all nodes. |
| `{/* @moc-role "<role>" */}` | The element's role property value. Attached when a role is set. |
| `{/* @moc-memo "<memo summary>" */}` | Summary text of a sticky-note memo. Attached when a memo is linked. |

These comments serve as supplementary information for AI agents to understand the elements within the TSX.

---

## 6. Editor Data Block

A block for saving the internal state of the GUI editor.
Stored as a template literal variable at the end of the file.

### 6.1 Format

```typescript
const __mocEditorData = `
{
  "craftState": { ... },
  "memos": [ ... ],
  "viewport": { ... }
}
`;
```

### 6.2 Fields

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `craftState` | **Required** | `Record<string, unknown>` | Craft.js node tree |
| `memos` | **Required** | `MocEditorMemo[]` | Complete data for sticky-note memos |
| `viewport` | Optional | `{ mode, width, height }` | Viewport settings |

### 6.3 Escape Rules

The following characters must be escaped inside the template literal JSON string:

| Character | Escaped Form |
|-----------|-------------|
| `` ` `` (backtick) | `` \` `` |
| `${` (template expression) | `\${` |

The parser restores (unescapes) these before calling `JSON.parse`.

### 6.4 JSON Formatting Rules

- 2-space indentation
- Line endings: LF
- Trailing newline at end of file

---

## 7. Normalization Rules

When saving a `.moc` file, apply the following normalization:

- **Line endings**: LF (`\n`)
- **Trailing newline**: Present (one newline at end of file)
- **Editor data JSON**: 2-space indentation
- **Between sections**: Separated by one blank line

---

## 8. Compatibility Policy

### 8.1 Deprecated Tags

The following tags have been deprecated. Parsers should ignore them when detected:

| Deprecated Tag | Reason |
|----------------|--------|
| `@moc-id` | Difficult to manage ID consistency when copying or templating files |
| `@moc-editor-data` (base64 comment block) | Low human/AI readability; migrated to template literal format |

### 8.2 Unknown Field Handling

- Unknown `@moc-*` tags in the metadata block → **Ignore** (do not error)
- Unknown fields in the editor data JSON → **Ignore** (do not error)
- Since new fields may be added in future versions, do not halt processing when unknown fields are encountered.

### 8.3 Backward Compatibility

This specification (v1.0.0 official release) does **not guarantee** backward compatibility with draft versions.
Legacy-format files may not be correctly parsed by newer-version parsers.

---

## 9. Complete File Example

```typescript
/**
 * Mocker Document (.moc)
 * A definition file for GUI mockups created with the VSCode extension "Mocker."
 * An integrated data format for collaborative development between Humans, GUIs, and AI Agents.
 *
 * ...(header description omitted)...
 *
 * @moc-version 1.0.0
 * @moc-intent Login form mockup
 * @moc-theme light
 * @moc-layout flow
 * @moc-viewport desktop
 *
 */
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

export default function LoginForm() {
  return (
    <>
      {/* @moc-node ROOT */}
      <div className="flex min-h-screen items-center justify-center">
        {/* @moc-node node1 */}
        {/* @moc-memo "Email validation required" */}
        <Input id="emailInput" type="email" placeholder="Email" />
        {/* @moc-node node2 */}
        {/* @moc-memo "Submit button for login action" */}
        <Button id="loginButton">Login</Button>
      </div>
    </>
  );
}

const __mocEditorData = `
{
  "craftState": {
    "ROOT": {
      "type": { "resolvedName": "CraftContainer" },
      "props": { "className": "flex min-h-screen items-center justify-center" },
      "nodes": ["node1", "node2"],
      "linkedNodes": {},
      "parent": null
    }
  },
  "memos": [],
  "viewport": {
    "mode": "desktop",
    "width": 1280,
    "height": 800
  }
}
`;
```

---

## 10. License

MIT License
