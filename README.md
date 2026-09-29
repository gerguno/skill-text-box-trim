# text-box trim

Agent skill for new CSS. `text-box: trim-both cap alphabetic` makes a padding or gap token the distance from cap-height and the baseline to the box edge.

`padding: 12px` on an untrimmed line box is not 12px to the cap. Arial at 16px measures about 14.5px above the cap and 16px below the baseline. Firefox 154 brought `text-box` to Baseline; this skill is the rule an agent follows when it writes a new stylesheet, plus the font-metric step for inputs, textareas, and selects, which cannot trim.

Measurements and the case: https://skill-text-box-trim.olesgergun.com

## Install

Any agent that reads skills:

```bash
npx skills add gerguno/skill-text-box-trim
```

Claude Code:

```bash
git clone https://github.com/gerguno/skill-text-box-trim ~/.claude/skills/text-box-trim
```

Cursor:

```bash
git clone https://github.com/gerguno/skill-text-box-trim ~/.cursor/skills/text-box-trim
```

For one project only, clone into `.claude/skills/text-box-trim` or `.cursor/skills/text-box-trim` inside the repo.

## What it writes

```css
* {
  text-box: trim-both cap alphabetic;
}

:where(input, textarea, select) {
  text-box: none;
}
```

Then padding and gap equal the token. Icons sit on the cap-height, not in a flex-centered `1em` box. The rest is in [SKILL.md](SKILL.md) and [examples.css](examples.css).

## Font metrics

Inputs, textareas, and selects stay untrimmed. When a face sits off-center in that box, or its cap-height metric does not match the drawn `H`, the skill runs [normalize-metrics](https://github.com/gerguno/normalize-metrics) instead of guessing `ascent-override` percentages:

```bash
npx normalize-metrics ./fonts --check
npx normalize-metrics ./fonts --css
```

## License

MIT
