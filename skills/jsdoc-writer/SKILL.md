---
name: jsdoc-writer
description: 'Add JSDoc to code entities. TRIGGER when: user asks to add/write/improve JSDoc (e.g., "add jsdoc to Foo", "jsdoc for Bar").'
---

Add precise yet tight JSDoc for the entities specified by the user. Avoid slop. Make it the JSDoc that engineers will love to read, not the JSDoc that PMs will love to show off.

Important:
- Never restate the function name in prose. The signature already says _what_ — the JSDoc should say _why_, or surface constraints and gotchas (nullability, side effects, mutation, valid ranges).
- `@param`/`@returns` descriptions only when the type and name don't tell the full story. A bare `@param {string} name` beats `@param {string} name - The name`.
- One to three lines for most functions. Longer only for genuinely complex contracts.
- Match the voice of the surrounding codebase.

Scope (pass as arguments):
- Name the functions, classes, methods, types, or files to document.
- If a file path is given, document all exported entities in that file.
