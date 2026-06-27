# DeezSkills

A [Claude Code](https://code.claude.com/docs/en/plugins) plugin marketplace.

## Plugins

### `weather-forecast`

Get the current weather, hourly and multi-day forecasts, and active US weather
alerts for any location — no API key required. Powered by [wttr.in](https://wttr.in)
(forecasts) and the [US National Weather Service API](https://api.weather.gov)
(alerts, US locations only).

Ask things like *"what's the weather in Tokyo?"*, *"hourly forecast for tomorrow"*,
or *"any weather alerts near me?"* and Claude will use the skill automatically.

## Installation

In Claude Code, add this marketplace and install the plugin:

```
/plugin marketplace add <your-github-username>/DeezSkills
/plugin install weather-forecast@deez-skills
```

> Replace `<your-github-username>` with your GitHub account once this repo is pushed.

To update later, after you push new commits:

```
/plugin marketplace update deez-skills
```

## Local development / testing

Test a plugin without installing it from the marketplace:

```bash
claude --plugin-dir ./plugins/weather-forecast
```

Validate the marketplace and plugin manifests:

```bash
claude plugin validate .
```

## Repository layout

```
.claude-plugin/marketplace.json          # marketplace catalog
plugins/weather-forecast/
  .claude-plugin/plugin.json              # plugin manifest
  skills/weather-forecast/SKILL.md        # the skill
```

## License

MIT
