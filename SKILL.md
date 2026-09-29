---
name: text-box-trim
description: >-
  Use when writing CSS for new UI, setting padding or gap around text,
  setting up global typography, or loading a web font. Also use when a
  button, badge, tab, chip, card, tile, or heading looks optically off
  because font leading adds space the spacing token did not include, or
  when text in an input, textarea, or select sits off-center.
---

# text-box trim

Visual distance from cap-height and the alphabetic baseline to the box edge equals the spacing token. Font leading is not part of that distance.

Scope is new CSS: a new project, a new component, or a global stylesheet written from scratch. Plain CSS. No `@supports` fallbacks.

If the request does not change spacing, type, or the font, and the control already shipped, leave its padding alone. A color or copy change is not a reason to add `text-box`.

## Write this

On a new stylesheet, in this order:

```css
* {
  text-box: trim-both cap alphabetic;
}

:where(input, textarea, select) {
  text-box: none;
}
```

`:where` keeps the exception at zero specificity, so it only wins by coming second. `button` stays trimmed.

Then set `padding` and `gap` to the token. Do not use unequal padding, negative margins, or a `translateY` nudge to cancel leading.

On a single new component, when the global rule is not yours to add, put the same `text-box: trim-both cap alphabetic` on the text elements. Still leave `input`, `textarea`, and `select` at `text-box: none`.

Reference: [examples.css](examples.css). Measurements: https://skill-text-box-trim.olesgergun.com

## Turning trim off

The `:where` rule already turns trim off for `input`, `textarea`, and `select`. Anywhere else that must keep the font's own line box, the declaration is `text-box: none`.

Plain CSS: write that declaration on the element. Do not create a Sass file just to hold it.

```css
text-box: none;
```

The project already uses Sass: add this mixin and `@include` it. Do not invent a second opt-out.

```scss
@mixin trim-disable {
  text-box: none;
}
```

## Icons

An inline icon sits on the baseline. Shift it onto the cap-height:

```css
.icon {
  display: inline-block;
  width: 1em;
  height: 1em;
  vertical-align: baseline;
  translate: 0 calc((100% - 1cap) / 2);
}
```

The parent stays in inline flow (a plain `button` or `span`), so the icon sits on the text's baseline.

`align-items: center` on a flex or grid row with a `1em` icon does not do this. The icon becomes the block size, the cap floats inside it, and a 12px padding token measures about 14px. The same `translate` on that flex icon moves it the wrong way.

If the control is already a flex or grid row, size the icon to the cap instead of translating it:

```css
.icon {
  width: 1cap;
  height: 1cap;
}
```

## Labels with no capital

A label that never has a capital (a lowercase brand style, `text-transform: lowercase`) has nothing at the cap line. Trimmed to the cap, the x-height sits below the token by the difference between cap-height and x-height. Trim that component to the x-height:

```css
.label {
  text-box: trim-both ex alphabetic;
}
```

CSS cannot see whether a string has a capital. Put `ex` on the component whose labels are lowercase by design, never on `*`. Mixed, user-written, or unknown text keeps `cap`. Ascenders (b, d, f, h, k, l, t) paint into the top padding, the same way diacritics do.

## Font metrics

Trim does not run on a real `input`, `textarea`, or `select`. The declaration can show up in computed style while the control keeps its own line box. Those controls stay at `text-box: none` so `g`, `y`, and `p` are not clipped.

Centering text in that untrimmed box is a font-metric problem. Do not guess `ascent-override` percentages.

```
trim is on (buttons, text, tiles, headings), and a font file is in the project
  → compare the font's cap-height metric with the drawn H
       (in the browser: an element with height: 1cap against canvas measureText("H").actualBoundingBoxAscent)
  → they match within 0.5px: leave the font file alone
  → they do not: OS/2 sCapHeight is wrong, and trim cuts at it.
       CSS has no override for cap-height, so --css does not fix it.
       npx normalize-metrics <file-or-folder>
       (writes a copy with sCapHeight from H; ship the copy)

trim is off, and a font file is in the project
  → npx normalize-metrics <file-or-folder> --check
  → already good: stop
  → off, and a stylesheet ships with the font:
       npx normalize-metrics <file-or-folder> --css
  → off, and no stylesheet can travel with the font:
       npx normalize-metrics <file-or-folder>
       (writes a copy; does not overwrite)
```

Until that command has written a CSS file, the `@font-face` you write has `font-family` and `src` only. After it has, copy `ascent-override`, `descent-override`, and `line-gap-override` from that file.

`--css` writes `ascent-override`, `descent-override`, and `line-gap-override` for the original files. Paste those three values only from the command's output or from the CSS file it wrote. If you did not run the command, the file is missing, or the command failed, do not write the descriptors. Say that the command still has to be run. Do not put those descriptors on a font the tool already rewrote. Do not pass `--in-place` unless someone asked to overwrite the file. Rewriting a licensed font and shipping the result can violate the EULA.

The tool grades a face Bad when the cap-center offset is 40‰ or more. Install: `npm i -D normalize-metrics`. Repo: https://github.com/gerguno/normalize-metrics

No font file (a system face only): do not invent overrides and do not trim the control.

## Edges

These were measured. Do not add a special case for them.

- Diacritics paint into the padding. `overflow: hidden` clips them only when that ink is taller than the padding on that side. At 16px Arial, `Ї` sticks about 2.3px above the cap: 4px padding holds, 0px clips. Give the padding room, or drop the clip. Do not turn trim off.
- Ellipsis and `line-clamp` keep the same trim. Descenders paint into the padding and clip only when the padding is shorter than the descent and the box hides overflow.
- `align-items: baseline` across sizes still shares one baseline.
- List padding is measured on the item text, the same way as a button.

## Rationalizations

| Excuse | Reality |
| --- | --- |
| "The designer said padding: 12px, so the declaration is 12px." | 12px of padding on an untrimmed line box is not 12px to the cap. Arial at 16px measures about 14.5px above the cap and 16px below the baseline. |
| "Flex centering puts the icon on the text." | A 1em icon in a centered flex row steals the block size. Padding stops matching the token. |
| "The label is lowercase, so trim to `ex` everywhere." | Only on a component that is lowercase by design. CSS cannot tell whether a string has a capital; unknown text stays on `cap`. |
| "Trim the input so the word sits in the middle." | The control does not take the trim. Descenders need the untrimmed box. Fix the font metrics. |
| "ascent-override: 75% looks about right." | Percentages come from `normalize-metrics --css`, or they do not get written. |
| "The CLI would have printed 98% / 25%, so I'll put that in." | A number you did not see in the command output is an invented number. No file, no run, no descriptors. |
| "Trim is on, so the font file does not matter." | Trim cuts at `OS/2.sCapHeight`. Unica77 LL ships 726 of 2048 for an H that is 1487: trim puts the cap 6px outside a 12px token. Check `1cap` against the drawn H. |
| "I'll rewrite the font with fontTools." | For the web, `--css` on the original file. Rewrite a copy only when CSS cannot ship with the font. |
| "This shipped button's 9px/7px padding looks uneven; I'll clean it up while changing the color." | Shipped padding stays. The request did not change spacing. |
