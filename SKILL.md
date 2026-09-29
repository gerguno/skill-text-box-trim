---
name: text-box-trim
description: >-
  Use when writing CSS for new UI, setting padding or gap around text,
  setting up global typography, or loading a web font. Also use when a
  button, badge, tab, chip, card, tile, or heading looks optically off
  because font leading adds space the spacing token did not include, or
  when text in an input or select sits off-center.
---

# text-box trim

Visual distance from cap-height and the alphabetic baseline to the box edge equals the spacing token. Font leading is not part of that distance.

Scope is new CSS: a new project, a new component, or a global stylesheet written from scratch. Plain CSS. `text-box` is Baseline (Firefox 154), so no `@supports` fallbacks. A browser without it shows the untrimmed box, a few pixels more padding, and nothing breaks.

Never add the `*` rule to a stylesheet that already ships. It moves every existing component at once. In an existing project, trim only the component you are writing or the one whose spacing you were asked to fix.

If the request does not change spacing, type, or the font, and the control already shipped, leave its padding alone. A color or copy change is not a reason to add `text-box`.

## Write this

On a new stylesheet, in this order:

```css
* {
  text-box: trim-both cap alphabetic;
}

:where(input, select) {
  text-box: none;
}
```

`:where` keeps the exception at zero specificity, so it only wins by coming second. `button` stays trimmed.

Tailwind: put both rules in `@layer base`, in the same order. Do not add `[text-box:…]` utilities to each element.

Then set `padding` and `gap` to the token. Do not use unequal padding, negative margins, or a `translateY` nudge to cancel leading.

On a single component, when the global rule is not yours to add, put the same `text-box: trim-both cap alphabetic` on the text elements. Still leave `input` and `select` at `text-box: none`.

Reference: [examples.css](examples.css). Measurements: https://skill-text-box-trim.olesgergun.com

## Turning trim off

The `:where` rule already turns trim off for `input` and `select`. Anywhere else that must keep the font's own line box, the declaration is `text-box: none`.

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

If the control is already a flex or grid row, keep the icon at `1em` and shrink only its margin box to the cap:

```css
.icon {
  width: 1em;
  height: 1em;
  margin-block: calc((1cap - 1em) / 2);
}
```

The row's block size stays one cap, the icon centers on the cap, and it paints about 2px into the padding on each side, the same as the inline icon. Measured at 16px Arial: 12px padding measures 12px. This negative margin sizes the icon; it does not cancel leading, so the rule above does not forbid it.

When a cap-sized icon is what the design wants, `width: 1cap; height: 1cap` also measures right. It is about 28% smaller than `1em` in Arial, so do not pick it just to make the numbers work.

## Labels with no capital

A label that never has a capital (a lowercase brand style, `text-transform: lowercase`) has nothing at the cap line. Trimmed to the cap, the x-height sits below the token by the difference between cap-height and x-height. Trim that component to the x-height:

```css
.label {
  text-box: trim-both ex alphabetic;
}
```

CSS cannot see whether a string has a capital. Put `ex` on the component whose labels are lowercase by design, never on `*`. Mixed, user-written, or unknown text keeps `cap`. Ascenders (b, d, f, h, k, l, t) paint into the top padding, the same way diacritics do.

## Font metrics

Trim does not run on a real `input` or `select`. The declaration can show up in computed style while the control keeps its own line box, and Chrome clips an input's text to the trimmed line. Those controls stay at `text-box: none` so `g`, `y`, and `p` are not clipped.

A `textarea` takes the trim: its text lays out like a block, the first cap lands on the padding token, and the last line's descenders stay inside the bottom padding when it scrolls. Leave it trimmed.

Centering text in that untrimmed box is a font-metric problem. Do not guess `ascent-override` percentages.

Every project on this skill has both boxes: trimmed text, buttons, and headings, and untrimmed `input` and `select`. One font file serves both. A font file in the project gets two checks, in this order:

1. **Where trim cuts.** Trim cuts at `OS/2.sCapHeight`. Compare it with the top of the drawn `H`. `--check` does not test this.
   - In a browser, with the font loaded: an element with `height: 1cap` against canvas `measureText("H").actualBoundingBoxAscent`. They match within 0.5px.
   - Without a browser, read the file (needs `fonttools`, plus `brotli` for WOFF2). It prints unitsPerEm, sCapHeight, and the top of H. They match within unitsPerEm / 32.
     ```
     python3 -c "import sys;from fontTools.ttLib import TTFont;from fontTools.pens.boundsPen import BoundsPen;f=TTFont(sys.argv[1]);g=f.getGlyphSet();p=BoundsPen(g);g[f.getBestCmap()[72]].draw(p);print(f['head'].unitsPerEm,f['OS/2'].sCapHeight,p.bounds[3])" <font-file>
     ```
2. **How the untrimmed box centers.** `npx normalize-metrics <file-or-folder> --check`.

| Cap-height | `--check` | Do |
| --- | --- | --- |
| matches | good | Nothing. Leave the font file alone. |
| matches | off | `npx normalize-metrics <file-or-folder> --css` when a stylesheet ships with the font. `npx normalize-metrics <file-or-folder>` (writes a copy) only when none can. |
| wrong | off | `npx normalize-metrics <file-or-folder>`. The copy takes `sCapHeight` from H and centers the untrimmed box, so it fixes both. Ship the copy with no `--css` descriptors. |
| wrong | good | The tool skips a face whose line metrics are already good and writes no copy. Do not patch `sCapHeight` yourself. Report both numbers and say the font needs a corrected `sCapHeight`. |

CSS has no override for cap-height. The `--css` descriptors center an untrimmed box. They do not change where trim cuts.

Until that command has written a CSS file, the `@font-face` you write has `font-family` and `src` only. After it has, copy `ascent-override`, `descent-override`, and `line-gap-override` from that file.

`--css` writes `ascent-override`, `descent-override`, and `line-gap-override` for the original files. Paste those three values only from the command's output or from the CSS file it wrote. If you did not run the command, the file is missing, or the command failed, do not write the descriptors. Say that the command still has to be run. Do not put those descriptors on a font the tool already rewrote. Do not pass `--in-place` unless someone asked to overwrite the file. Rewriting a licensed font and shipping the result can violate the EULA.

The tool grades a face Bad when the cap-center offset is 40‰ or more. Install: `npm i -D normalize-metrics`. Repo: https://github.com/gerguno/normalize-metrics

No font file (a system face only): do not invent overrides. `input` and `select` stay at `text-box: none`. Buttons, text, and headings still take the trim.

## Edges

These were measured. Do not add a special case for them.

- Diacritics paint into the padding. `overflow: hidden` clips them only when that ink is taller than the padding on that side. At 16px Arial, `Ї` sticks about 2.3px above the cap: 4px padding holds, 0px clips. Give the padding room, or drop the clip. Do not turn trim off.
- Ellipsis and `line-clamp` keep the same trim. Descenders paint into the padding and clip only when the padding is shorter than the descent and the box hides overflow.
- `align-items: baseline` across sizes still shares one baseline.
- List padding is measured on the item text, the same way as a button.
- Inline `code`, `mark`, and `kbd` with a background and padding keep their untrimmed box under the `*` rule (Chrome 152: 22px tall with trim and without). Leave them.

Not measured: scripts other than Latin and Cyrillic. Devanagari hangs its headline and vowel signs above the cap, and CJK has no cap or alphabetic baseline to trim to. Keep `cap alphabetic` for Latin and Cyrillic. For other scripts, say that the trim has not been measured instead of picking edges.

## Rationalizations

| Excuse | Reality |
| --- | --- |
| "The designer said padding: 12px, so the declaration is 12px." | 12px of padding on an untrimmed line box is not 12px to the cap. Arial at 16px measures about 14.5px above the cap and 16px below the baseline. |
| "Flex centering puts the icon on the text." | A 1em icon in a centered flex row steals the block size. Padding stops matching the token. Shrink its margin box to `1cap`. |
| "Just make the icon `1cap`, the numbers work." | They do, and the icon is about 28% smaller. Keep the design's icon size and use `margin-block`. |
| "It's an existing project, but `*` is the rule, so I'll add it." | The `*` rule is for a stylesheet written from scratch. On a shipped one it moves every component. Trim the component in scope. |
| "The label is lowercase, so trim to `ex` everywhere." | Only on a component that is lowercase by design. CSS cannot tell whether a string has a capital; unknown text stays on `cap`. |
| "Trim the input so the word sits in the middle." | The control does not take the trim. Descenders need the untrimmed box. Fix the font metrics. |
| "ascent-override: 75% looks about right." | Percentages come from `normalize-metrics --css`, or they do not get written. |
| "The CLI would have printed 98% / 25%, so I'll put that in." | A number you did not see in the command output is an invented number. No file, no run, no descriptors. |
| "Trim is on, so the font file does not matter." | Trim cuts at `OS/2.sCapHeight`. Unica77 LL ships 726 of 2048 for an H that is 1487: trim puts the cap 6px outside a 12px token. Check `1cap` against the drawn H. |
| "I'll rewrite the font with fontTools." | fontTools reads the file. It never writes it. The table under Font metrics says whether `--css` or a copy from `npx normalize-metrics` fixes it. |
| "`--check` passed, so the font is fine." | `--check` grades the untrimmed box only. A wrong `sCapHeight` passes it. Compare `1cap` with the drawn H as well. |
| "Older browsers ignore `text-box`, so I'll wrap it in `@supports`." | `text-box` is Baseline (Firefox 154). Write the declaration plain. |
| "This shipped button's 9px/7px padding looks uneven; I'll clean it up while changing the color." | Shipped padding stays. The request did not change spacing. |
