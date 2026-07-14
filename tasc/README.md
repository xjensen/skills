# tasc

`tasc` teaches your agent TASC — **T**ags, **A**ttributes, **S**lots, **C**lasses — an HTML/CSS methodology built on modern Custom Element syntax. Once installed, the skill kicks in on any front-end work: the agent follows the TASC decision tree when writing markup, uses the two-tier specificity model when writing CSS, and flags non-TASC components it encounters instead of silently copying their patterns.

## The idea

Most CSS methodologies (BEM and friends) route everything through classes, so markup ends up wearing its entire styling contract on its sleeve. TASC instead gives each concern its own HTML feature:

| Letter | Feature | Role |
|---|---|---|
| **T** | Tags | Documented components with a full feature set, as Custom Elements |
| **A** | Attributes | The official interface for supported component variants |
| **S** | Slots | Named sub-sections of a component |
| **C** | Classes | Escape hatch for consumers to customize beyond the official interface |

A component in the wild looks like this:

```html
<x-pop direction="right">
  <button slot="pop-button">Open pop</button>
  <div slot="pop-content" class="x-padding-3">Pop content.</div>
</x-pop>
```

You can read the whole contract at a glance: `<x-pop>` is a documented component, `direction` is its supported interface, the `slot` attributes mark its two sub-components (letting a native `<button>` serve directly, no wrapper elements), and the lone class is a consumer opt-in — not something the component requires.

No JavaScript is required for any of this. A Custom Element with no registered behavior is still a better container than a `<div>` — more descriptive in the markup and a stable styling hook.

## The specificity trick

TASC components ship every selector wrapped in `:where()`, which pins the entire system — base styles, attribute variants, slot styles — at specificity 0,0,0:

```css
:where(x-card) {
  --x-card-padding: 1rem;
  padding: var(--x-card-padding);
}

:where(x-card[format="compact"]) {
  --x-card-padding: 0.5rem;
}
```

That means *any* selector a consumer writes wins automatically:

```css
x-card { --x-card-padding: 2rem; }  /* (0,0,1) beats everything above */
```

Good-looking defaults, zero-fight overrides. No `!important`, no selector gymnastics, no specificity wars with the system you're building on. And because each attribute variant pairs with a custom property, one flat rule like the above overrides the base *and* every variant at once.

## What the agent does with it

The skill triggers on markup and styling work — building a page or component, choosing between a Custom Element, a native element, or a class, naming a slot, or reviewing existing front-end code. When it's active, the agent:

- works the decision tree in order: native element → Custom Element → attribute → slot → class (last resort);
- distinguishes **authoring** a component (all rules apply, no required classes) from **consuming** one (documented interface first, classes as the sanctioned escape hatch);
- keeps component CSS at specificity 0,0,0 with base rules before variants;
- pairs custom attributes with `--x-{component}-{property}` custom properties;
- prefers converting legacy non-TASC components over imitating them — but asks before doing it.

## Installation

Symlink the skill into your personal skills folder:

```
ln -s /path/to/skills/tasc ~/.claude/skills/tasc
```

Or copy it into a single project's `.claude/skills/` to scope it to that repo.

## The prefix

Examples throughout use a generic `x-` prefix (`<x-card>`, `--x-card-padding`, `x-padding-3`). It's a placeholder — the skill tells the agent to substitute whatever Custom Element prefix your project already uses.
