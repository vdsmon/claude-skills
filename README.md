# claude-skills

Personal Claude Code skills by [@vdsmon](https://github.com/vdsmon).

## Install

```
/plugin marketplace add vdsmon/claude-skills
/plugin install vdsmon-skills@vdsmon-skills
```

## Skills

### `skill-polish`

Post-mortem for any skill. Scans the conversation for friction (corrections, skipped steps, rejected tool calls), traces each to the responsible skill file, and applies concrete edits.

Trigger: `skill-polish`, `polish the skill`, `improve the skill`, `that should have been automatic`, `you skipped X`, `close the gaps`.

Works on any skill, not just its own.

## Layout

```
.claude-plugin/
  plugin.json          # Plugin manifest
  marketplace.json     # Self-hosted marketplace entry
skills/
  skill-polish/
    SKILL.md           # Skill definition
```

## License

MIT — see [LICENSE](LICENSE).
