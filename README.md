# vize-nested-v-if-type-binding-repro

Minimal reproduction for a false positive in [vize](https://github.com/ubugeeei-prod/vize)'s `type/no-unsafe-template-binding`.

A call with an object-literal argument, used as a condition, is reported as resolving to `any` / `unknown` in two places:
- in a `v-if` nested under an element with its own `v-if`;
- in a `v-else-if`.

The same call in a top-level `v-if` is not reported.

## Steps

```sh
pnpm install
pnpm lint
```

The rule is not in the default preset, so the `lint` script passes `--preset opinionated --type-aware`.

[`src/NestedVIf.vue`](./src/NestedVIf.vue):

```vue
<script setup lang="ts">
const isA = (o: { kind: string }) => o.kind === "a";
const show = true;
</script>

<template>
  <p v-if="isA({ kind: 'a' })">not reported</p>
  <div v-if="show">
    <p v-if="isA({ kind: 'a' })">reported</p>
  </div>
</template>
```

[`src/ElseIf.vue`](./src/ElseIf.vue):

```vue
<template>
  <p v-if="show">shown</p>
  <p v-else-if="isA({ kind: 'a' })">reported</p>
</template>
```

## Expected

No warning. `isA` returns `boolean`.

## Actual

```
Patina lint report: 2 warnings in 2 files

src/ElseIf.vue
  src/ElseIf.vue:8:17 warning type/no-unsafe-template-binding Template directive expression resolves to an unsafe `any` or `unknown` type
    Reference: docs/content/rules/type-and-script.md
    Help:
      Give the value a concrete type in <script setup> or narrow it before using it in the template.

src/NestedVIf.vue
  src/NestedVIf.vue:9:14 warning type/no-unsafe-template-binding Template directive expression resolves to an unsafe `any` or `unknown` type
    Reference: docs/content/rules/type-and-script.md
    Help:
      Give the value a concrete type in <script setup> or narrow it before using it in the template.
```

## Notes

Inside the nested `v-if`:
- reported: `isA({ kind: 'a' })`, `!isA({ kind: 'a' })` and `isA({ kind: 'a' } as const)`;
- not reported: `isA(kA)` (a named const argument), `isAKind('a')` (a string argument) and `isA({ kind: 'a' }) === true`.

Both positions are emitted inside a combined guard. The nested `v-if` becomes `if ((show) && (isA({ kind: 'a' }))) {`, as `vize check --show-virtual-ts` shows.

[`bind_checked_expressions`](https://github.com/ubugeeei-prod/vize/blob/v0.476.0/crates/vize_patina/src/linter/native_type_aware/expression_bindings/mapped.rs#L78-L94) only gives a condition its own typed probe when it is the whole guard; its comment reads "An else-if or synthetic enclosing guard needs a different owner". The type is then queried some other way, which answers `any` / `unknown` for these expressions.

## Environment

vize 0.476.0, Node v26.10.0, pnpm 12.9.0, macOS (arm64)
