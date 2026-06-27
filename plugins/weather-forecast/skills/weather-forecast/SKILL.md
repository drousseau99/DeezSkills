---
name: weather-forecast
description: Get current weather, hourly and multi-day forecasts, plus active weather alerts for any location. Use when the user asks about the weather, temperature, rain, hourly conditions, forecast, or storm/weather warnings for a city, place, or their current location.
---

# Weather Forecast

Fetch weather conditions and forecasts from [wttr.in](https://wttr.in) (free, no API key) and active weather alerts from the US National Weather Service [api.weather.gov](https://api.weather.gov) (free, no API key, US locations only).

## How to use

1. **Determine the location.** Use the location the user gave (city name, "London", "Paris,FR", an airport code like "SFO", or a zip code). If the user says "here", "my location", or gives no location, omit the location entirely — wttr.in geolocates by IP.

2. **Fetch the forecast.** Run the request through `curl` with a short timeout. The `format=j1` JSON output is the most reliable to parse:

   ```bash
   curl -fsS --max-time 15 "https://wttr.in/<LOCATION>?format=j1"
   ```

   Replace `<LOCATION>` with a URL-encoded location (spaces become `%20` or `+`). For the IP-based location, use `https://wttr.in/?format=j1`.

3. **Summarize for the user.** Parse the JSON and present, in this order:
   - **Active alerts (if any):** show these first and prominently — see "Weather alerts" below.
   - **Current conditions:** temperature (°C and °F), what it feels like, a short description (e.g. "Partly cloudy"), humidity, and wind.
   - **Hourly detail:** see "Hourly forecast" below.
   - **Daily forecast:** the next 3 days with high/low temps, chance of rain, and a one-line description each.

   Keep it concise and scannable. Lead with any alerts, then current conditions, then hourly, then the daily outlook.

## Hourly forecast

The `format=j1` response already contains hourly data — no extra request needed. Each entry in `weather[N].hourly` covers a 3-hour step (8 entries per day), with a `time` field in HHMM form (`"0"` = 00:00, `"300"` = 03:00, `"1200"` = 12:00, etc.).

Useful per-hour fields: `tempC`/`tempF`, `FeelsLikeC`/`FeelsLikeF`, `weatherDesc`, `chanceofrain`, `chanceofsnow`, `chanceofthunder`, `humidity`, `windspeedMiles`/`windspeedKmph`, `winddir16Point`, `uvIndex`.

By default show today's remaining hourly steps (skip entries whose time is already past the current `observation_time`). If the user asks for "tomorrow" or "the next 24/48 hours," pull from `weather[1]` / `weather[2]` as well. Present as a compact table or list: time → temp (feels like), description, rain %.

## Weather alerts

After fetching `format=j1`, read the location's latitude/longitude from `nearest_area[0].latitude` / `longitude`, then query the NWS for active alerts. **NWS requires a `User-Agent` header** and only covers US locations:

```bash
curl -fsS --max-time 15 -H "User-Agent: claude-weather-skill (contact@example.com)" \
  "https://api.weather.gov/alerts/active?point=<LAT>,<LON>"
```

The response is a GeoJSON `FeatureCollection`. Each alert in `features[].properties` has: `event` (e.g. "Heat Advisory"), `severity`, `urgency`, `headline`, `areaDesc`, `description`, and `instruction`. Surface the `event`, `severity`, and `headline` for each, and offer the full `description`/`instruction` if the user wants details.

- If `features` is empty, say there are no active alerts for that location.
- If the location is **outside the US**, skip the NWS call and note that alert data isn't available for that region.

## Useful endpoints

- One-line current weather: `curl -fsS --max-time 15 "https://wttr.in/<LOCATION>?format=3"`
- Full JSON (current + hourly + 3-day forecast): `curl -fsS --max-time 15 "https://wttr.in/<LOCATION>?format=j1"`
- Active US alerts by coordinates: `curl -fsS --max-time 15 -H "User-Agent: <app> (<email>)" "https://api.weather.gov/alerts/active?point=<LAT>,<LON>"`
- Force wttr.in units in rendered output: append `&u` (USCS/Fahrenheit) or `&m` (metric/Celsius). With `format=j1` you get both unit systems, so convert as needed for the user.

## Notes

- If `curl` fails or a service is unreachable, tell the user and offer to retry — do not invent weather data. If only the alerts call fails, still show the forecast and note that alerts couldn't be checked.
- For ambiguous city names (e.g. "Springfield"), ask the user which one, or include the country/state code in the location.
- Respect the user's preferred units if they state one; otherwise show both °C and °F.
