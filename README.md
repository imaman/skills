# Skills

Battle-tested [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) for everyday use.

Every skill in this repo is one I find valuable and use daily on moojo-tech's production monorepo.

They all started the same way — a frustration moment: a review that missed something obvious, a JSDoc block that restated the function name, a regression that slipped through. Then got iterated on until the friction was gone.

Browse the [`skills/`](skills/) directory to see what's available.

## Install

Use either the skills installer or the marketplace configuration.

### npx

```bash
npx skills add https://github.com/imaman/skills --skill regression-dog
```

### Marketplace

Add this to `~/.copilot/settings.json` for GitHub Copilot CLI or `~/.claude/settings.json` for Claude Code:

```json
{
  "extraKnownMarketplaces": {
    "imaman-skills": {
      "source": {
        "source": "github",
        "repo": "imaman/skills"
      }
    }
  },
  "enabledPlugins": {
    "imaman-skills@imaman-skills": true
  }
}
```

### Prerequisites

`pr-title` requires `dcc diff`, `git`, `grep`, and `head` on your PATH.

## License

MIT
