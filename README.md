# text-box trim

Agent skill for new CSS. `text-box: trim-both cap alphabetic` makes a padding or gap token the distance from cap-height and the baseline to the box edge.

`padding: 12px` on an untrimmed line box is not 12px to the cap. Arial at 16px measures about 14.5px above the cap and 16px below the baseline. Firefox 154 brought `text-box` to Baseline; this skill is the rule an agent follows when it writes a new stylesheet, plus the font-metric step for inputs and selects, which cannot trim.

Measurements and the case: https://skill-text-box-trim.olesgergun.com

## Install

Any agent that reads skills:

```bash
npx skills add gerguno/skill-text-box-trim
```

Claude Code:

```bash
npx skills add gerguno/skill-text-box-trim -g -a claude-code
```

Cursor:

```bash
npx skills add gerguno/skill-text-box-trim -g -a cursor
```

For one project only, drop `-g`.

The repo is also a Claude plugin. The skill lives in [skills/text-box-trim](skills/text-box-trim); to copy it by hand, copy that folder into `~/.claude/skills/` or `~/.cursor/skills/`.

## What it writes

```css
* {
  text-box: trim-both cap alphabetic;
}

:where(input, select) {
  text-box: none;
}
```

Then padding and gap equal the token. Icons sit on the cap-height, not in a flex-centered `1em` box. The `*` rule is for a new stylesheet only; in an existing project the skill trims the component in scope. The rest is in [SKILL.md](skills/text-box-trim/SKILL.md) and [examples.css](skills/text-box-trim/examples.css).

## Font metrics

Inputs and selects stay untrimmed. Textareas trim. The skill checks a font file twice: its cap-height metric against the drawn `H` (where trim cuts), and how it centers in the untrimmed box. It fixes either with [normalize-metrics](https://github.com/gerguno/normalize-metrics) instead of guessing `ascent-override` percentages:

```bash
npx normalize-metrics ./fonts --check
npx normalize-metrics ./fonts --css
```

## What it runs and sends

The skill is instructions and a CSS reference. It runs nothing on its own and sends no data anywhere. When a project has font files, it asks the agent to run `npx normalize-metrics` and a read-only fontTools check on them, locally. normalize-metrics writes a corrected copy and leaves the original file alone unless `--in-place` is passed.

## License

MIT
