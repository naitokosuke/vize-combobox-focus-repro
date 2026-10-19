# vize-combobox-focus-repro

Minimal reproduction for false positives in [vize](https://github.com/ubugeeei-prod/vize)'s `a11y/interactive-supports-focus` on the [APG combobox pattern](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/). In that pattern, DOM focus stays on the input, and `aria-activedescendant` points at the active option, so the listbox and its options are not focusable.

The rule exempts such a listbox, with two gaps:

1. The listbox's `id` is only matched when it is static, and `aria-controls` only when it is static or a bound string literal. An identifier, as in `:id="listId"` / `:aria-controls="listId"` (the usual shape with `useId()`), is not matched.
2. `role="option"` inside the exempted listbox is never exempted.

## Steps

```sh
pnpm install
pnpm lint
```

[`src/BoundIds.vue`](./src/BoundIds.vue):

```vue
<script setup lang="ts">
const listId = "fruits";
</script>

<template>
  <input
    aria-activedescendant="fruit-0"
    :aria-controls="listId"
    aria-expanded="true"
    aria-label="Fruit"
    role="combobox"
  />
  <ul :id="listId" role="listbox">
    <li id="fruit-0" aria-selected="true" role="option">Apple</li>
  </ul>
</template>
```

[`src/StaticIds.vue`](./src/StaticIds.vue) is the same with `id="fruits"` and `aria-controls="fruits"`.

## Expected

No warning in either file.

## Actual

```
Patina lint report: 3 warnings in 2 files

src/BoundIds.vue
  src/BoundIds.vue:13:3 warning a11y/interactive-supports-focus Element with role="listbox" must be focusable (add tabindex="0")
    Reference: docs/content/rules/accessibility.md
  src/BoundIds.vue:14:5 warning a11y/interactive-supports-focus Element with role="option" must be focusable (add tabindex="0")
    Reference: docs/content/rules/accessibility.md

src/StaticIds.vue
  src/StaticIds.vue:10:5 warning a11y/interactive-supports-focus Element with role="option" must be focusable (add tabindex="0")
    Reference: docs/content/rules/accessibility.md
```

## Notes

- [`collect_controlled_listboxes`](https://github.com/ubugeeei-prod/vize/blob/v0.476.0/crates/vize_patina/src/rules/a11y/interactive_supports_focus.rs#L102-L115) reads `aria-controls` as a static value or a bound string literal.
- The listbox's `id` is read as a static value only ([L68-L74](https://github.com/ubugeeei-prod/vize/blob/v0.476.0/crates/vize_patina/src/rules/a11y/interactive_supports_focus.rs#L68-L74)), and only `role="listbox"` is exempted.

With `--preset opinionated`, `vue/use-unique-element-ids` also warns on the static ID references (`aria-controls="fruits"`, `aria-activedescendant="fruit-0"`). So the static ids that get the exemption are what another rule asks you to replace.

## Environment

vize 0.476.0, Node v26.10.0, pnpm 12.9.0, macOS (arm64)
