# Setting Your Location — `weather.zsh`

`weather.zsh` needs to know where you are to fetch the right forecast. This is done with two required values and one optional label.

---

## The Three Location Variables

| Variable | Required | What it does |
|----------|----------|--------------|
| `LAT` | ✅ Yes | Latitude — drives the API lookup |
| `LON` | ✅ Yes | Longitude — drives the API lookup |
| `LOCATION_NAME` | ☑️ Optional | A display label shown in output only — never sent to the API |

`LAT` and `LON` are the only values the NOAA API actually uses. The NWS automatically resolves the nearest forecast office and returns the real city/state for your coordinates — you'll see this as the "NWS Location" line in `summary` output. `LOCATION_NAME` is purely cosmetic; if you leave it blank or don't set it, the output still works fine.

---

## How to Find Your Coordinates

You need a **decimal degree** latitude and longitude (e.g. `34.1478`, `-118.1445`). Degrees/minutes/seconds format won't work directly and needs converting first.

A few easy ways to get them:

**Google Maps**
Right-click anywhere on the map → the coordinates appear at the top of the context menu. Click them to copy.

**maps.apple.com**
Drop a pin at your location → the URL updates to include `?ll=LAT,LON`.

**OpenStreetMap**
Navigate to your location → right-click → "Show address" — coordinates are shown in the left panel. Or read them directly from the URL: `map=ZOOM/LAT/LON`.

**Your phone**
On iPhone: Compass app → scroll down — it shows lat/lon in decimal degrees.
On Android: Google Maps → tap and hold to drop a pin → coordinates shown at the bottom of the screen.

**Command line (if you have `curl` and `jq` already)**
```zsh
curl -s "https://nominatim.openstreetmap.org/search?q=Pasadena,CA&format=json&limit=1" \
  -H "User-Agent: weather.zsh-setup" \
  | jq -r '.[0] | "LAT=\(.lat | tonumber | . * 10000 | round / 10000)\nLON=\(.lon | tonumber | . * 10000 | round / 10000)"'
```
Replace `Pasadena,CA` with your city and state or a full address.

> ⚠️ **Nominatim returns high-precision coordinates** (e.g. `34.0802852`) but the NWS API requires a maximum of 4 decimal places — higher precision causes the `/points` lookup to silently fail with no forecast URL. The command above truncates to 4dp automatically. If you obtain coordinates from another source, round them yourself before use:
> ```zsh
> printf "%.4f\n" 34.0802852   # → 34.0803
> ```

> **Note:** The NWS API only covers the continental US, Alaska, Hawaii, Puerto Rico, and other US territories. If your coordinates fall outside NWS coverage, the script will tell you.

---

## Setting Your Location

There are three ways, in order of permanence:

### 1. Config file (permanent — recommended)

Create the config file if it doesn't exist:

```zsh
mkdir -p ~/.config/weather_zsh
touch ~/.config/weather_zsh/config
chmod 600 ~/.config/weather_zsh/config   # required — script refuses to load looser permissions
```

Add your location (and optionally a display name):

```zsh
# ~/.config/weather_zsh/config
# Note: LAT and LON must be truncated to 4 decimal places (NWS API requirement)
LAT=34.0803
LON=-118.3617
LOCATION_NAME=Los Angeles, CA
```

The config file is loaded on every run, so you never need to pass coordinates on the command line again.

### 2. Command-line flags (one-off or scripted use)

```zsh
./weather.zsh --lat 40.7128 --lon -74.0060 --name "New York, NY"
```

Flags override the config file for that run only. Useful for checking another city without changing your defaults.

### 3. Edit the script defaults (simple but less flexible)

At the top of `weather.zsh`, change these three lines:

```zsh
DEFAULT_LAT="34.1478"
DEFAULT_LON="-118.1445"
DEFAULT_LOCATION_NAME="Los Angeles, CA"
```

This permanently changes the fallback used when no config file or flags are present. Fine for personal use, but means your changes will conflict on future script updates — the config file approach is cleaner.

---

## Priority Order

When the script runs, location values are resolved in this order — later values win:

```
Script defaults  →  Config file  →  CLI flags
(lowest priority)                  (highest priority)
```

---

## Future: Location Helper Utility

A future `get_location.zsh` utility is planned to automate coordinate lookup from a place name or the system's current location. For now, the options above cover the most common workflows without introducing dependencies on any particular location service.
