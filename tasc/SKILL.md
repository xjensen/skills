---
name: tasc
description: TASC methodology (Tags, Attributes, Slots, Classes) for authoring HTML and CSS. Use for any front-end markup or styling work — building a page or component, deciding between a Custom Element vs native element vs class, naming a slot, writing CSS selectors, organizing stylesheets, or reviewing whether existing markup and styles follow TASC conventions.
---

# TASC

TASC stands for **T**ags, **A**ttributes, **S**lots, **C**lasses — an HTML/CSS methodology that uses modern Custom Element syntax to separate concerns:

- **Tags** — documented components offering a full feature set, as Custom Elements.
- **Attributes** — the interface for officially supported component variants.
- **Slots** — named sub-sections of a component.
- **Classes** — escape hatch for consumers to customize beyond the official interface.

Custom Elements share a project-wide prefix. Examples here use `x-` (`<x-card>`, `--x-card-padding`); substitute the project's own prefix if it has one.

## Two roles

TASC separates the **component author**, who defines a component's markup contract and CSS, from the **component consumer**, who places components on a page and customizes them:

- **Authoring a component** — or anything reusable: all rules below apply. Never require classes in default markup.
- **Consuming a component**: use its documented tags, attributes, and slots first; classes are the sanctioned escape hatch when the official interface doesn't cover a customization.

General page work usually mixes both roles — default to author rules for anything that could be reused.

## Rules

1. Prefer native HTML elements; fall back to Custom Elements (`<x-*>`) when the content needs a component-like shape.
2. Customizations on Custom Elements use attributes. Never add custom attributes to native elements.
3. Sub-components use `slot="{component}-{role}"` naming, where `{component}` is the Custom Element name without its prefix — e.g. `card-image` for `<x-card>`, `pop-content` for `<x-pop>`.
4. Reserve classes for consumers. Don't bloat default markup with required classes.
5. All component selectors are wrapped in `:where()` so they sit at specificity 0,0,0. Scope slot selectors to their component — `:where(x-card [slot="card-image"])` — which is still 0,0,0.
6. Each component gets its own CSS file. Within it, base rules come before variant rules — cascade order is the only way variants override base at equal specificity.
7. Where possible, each custom attribute exposes a matching custom property, named `--x-{component}-{property}` (e.g. `--x-card-padding`), so consumers can override defaults even on elements without that attribute.
8. When you encounter a component that doesn't follow TASC, prefer converting it rather than matching the legacy pattern — but flag the conversion to the user before doing it.

## Decision tree

Work through these in order when deciding how to express a piece of UI:

1. **(Tags)** If a native HTML element fits the content, use it.
   - Examples: `<article>`, `<section>`.
   - Keep defaults minimal and easy to override.
2. **(Tags)** If the content needs a component-like shape, use a Custom Element.
   - Examples: `<x-card>`, `<x-layout>`.
   - Use a Custom Element even with no JavaScript — it's more descriptive than `<div>`.
3. **(Attributes)** Use attributes for built-in customizations of a Custom Element.
   - Example: `<x-card format="medium">`.
   - Custom attributes are allowed on Custom Elements only — never on native elements like `<p>`.
   - Boolean state is also an attribute: `<x-pop open>`, `<x-accordion disabled>`. Either JS or the author can set them.
4. **(Slots)** Define sub-components with the `slot` attribute.
   - Example: `<img slot="card-image">`.
   - Slots let you use native elements as sub-components instead of nesting more Custom Elements.
   - Use the most semantically appropriate element for the slot. Reach for `<div>` only when you need a container for styling (e.g. padding around an image).
   - **Naming:** `{component}-{role}`, component name without prefix (rule 3). Examples: `pop-button`, `pop-content`, `card-image`, `card-header`, `accordion-title`.
5. **(Classes)** Offer classes only when none of the above fit.
   - Provide general-purpose utility classes.
   - For high-touch native elements like `<button>` or `<table>`, provide a class collection.

## CSS specificity strategy

The goal is **good-looking defaults, easy to override**. Consumers should never fight specificity to customize a component.

### Two-tier model

**Tier 0 — Component styles (specificity 0,0,0)**

Wrap all component selectors in `:where()`. This applies to base styles, attribute variants, and slot styles alike.

```css
/* base */
:where(x-card) {
  --x-card-padding: 1rem;
  padding: var(--x-card-padding);
}

/* attribute variant — must appear after the base rule */
:where(x-card[format="compact"]) {
  --x-card-padding: 0.5rem;
}

/* slot — scoped to its component, still 0,0,0 */
:where(x-card [slot="card-image"]) {
  border-radius: var(--x-card-image-radius, 4px);
}
```

**Tier 1 — Consumer overrides (any specificity > 0)**

Any normal selector a consumer writes wins automatically — no `!important`, no selector gymnastics.

```css
x-card { --x-card-padding: 2rem; }     /* (0,0,1) beats Tier 0 */
.my-card { --x-card-padding: 2rem; }   /* (0,1,0) beats Tier 0 */
```

Because every attribute variant pairs with a custom property (rule 7), a consumer who wants all cards to use more padding — regardless of `format` — writes a single Tier 1 rule: `x-card { --x-card-padding: 2rem; }`. It overrides both the base default and any attribute-driven variant, since `x-card` (0,0,1) beats `:where(x-card[format="compact"])` (0,0,0).

### Ordering rule

Within a component's CSS file, base rules must come before variant rules. Since both live at specificity 0,0,0, cascade order is the only thing that lets variants override the base. Variants always go at the bottom.

## Example: `<x-pop>`

```html
<x-pop direction="right">
  <button slot="pop-button">Open pop</button>
  <div slot="pop-content" class="x-padding-3">Pop content.</div>
</x-pop>
```

What this snippet communicates:

- `<x-pop>` is a documented component.
- The `direction` attribute is the supported interface for changing pop direction.
- The `slot` attributes show two sub-components: `pop-button` and `pop-content`.
- Slots let us use a native `<button>` directly — no wrapper `<div>` or `<x-pop-button-container>`.
- The consumer-applied `x-padding-3` utility class shows the clean separation between component structure (slot) and consumer-opt-in customization (class).
