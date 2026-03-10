# zsh-scripts

A growing collection of personal zsh scripts for system administration and daily automation tasks. Each script is self-contained, independently runnable, and includes built-in `--help` documentation.

---

## Scripts

- [`weather.zsh`](#weatherzsh) — NOAA weather CLI with desktop notification and caching support
- [`create_user.zsh`](#create_userzsh) — Interactive Linux user account creation with password policy enforcement

---

## `weather.zsh`

Fetches current conditions and forecasts from the [NOAA/NWS public API](https://api.weather.gov). No API key required. Output formats are designed for terminal use, desktop notifications, status bars, and piping to other tools.

**Default location:** Los Angeles, CA — edit `DEFAULT_LAT`, `DEFAULT_LON`, and `DEFAULT_LOCATION_NAME` at the top of the script to change it.

### Dependencies

| Tool | Purpose | Install |
|------|---------|---------|
| `curl` | HTTP requests to api.weather.gov | `apt install curl` / `pacman -S curl` / `brew install curl` |
| `jq` | JSON parsing | `apt install jq` / `pacman -S jq` / `brew install jq` |
| `bc` | Unit conversions (°C→°F, m/s→mph, Pa→inHg) | `apt install bc` / `pacman -S bc` / `brew install bc` |

### Usage

```zsh
./weather.zsh [OPTIONS]
```

| Option | Description |
|--------|-------------|
| `-l, --lat LATITUDE` | Latitude (decimal degrees) |
| `--lon LONGITUDE` | Longitude (decimal degrees) |
| `-n, --name NAME` | Human-readable location name (display only) |
| `-f, --format FORMAT` | Output format (default: `summary`) |
| `-c, --cache` | Use cached data if fresh (default TTL: 600s) |
| `--no-cache` | Bypass cache, always fetch live data |
| `--cache-ttl SECONDS` | Override cache TTL |
| `--cache-dir PATH` | Override cache directory |
| `--clear-cache` | Invalidate cached data (truncates files in place) |
| `--clothing` | Enable clothing recommendations (default: on) |
| `--no-clothing` | Disable clothing recommendations |
| `-h, --help` | Show full help |

### Output Formats

| Format | Description | Example use case |
|--------|-------------|-----------------|
| `summary` | Current conditions + next 4 forecast periods | Default terminal view |
| `full` | All forecast periods with detailed text | Full 7-day planning |
| `json` | Raw NWS forecast JSON | Piping to `jq` or other tools |
| `notification` | Single-line with key conditions | `notify-send`, dunst, etc. |
| `oneline` | Ultra-compact temp + conditions | Waybar, conky, i3status |

### Examples

```zsh
# Default location, summary view
./weather.zsh

# Full 7-day forecast
./weather.zsh --format full

# Desktop notification (Los Angeles, CA)
notify-send "Weather" "$(./weather.zsh --format notification --cache)"

# Status bar (waybar / conky / i3status)
./weather.zsh --format oneline --cache

# Different city
./weather.zsh --lat 40.7128 --lon -74.0060 --name "New York, NY"

# Pipe raw forecast JSON to jq
./weather.zsh --format json | jq '.properties.periods[0]'

# Force a fresh fetch, skip cache
./weather.zsh --no-cache --format summary

# Suppress clothing recommendation for one run
./weather.zsh --no-clothing
```

### Clothing Recommendations

`summary` and `notification` formats automatically suggest what to wear based on the current temperature:

| Condition | Default threshold |
|-----------|-----------------|
| 👕 Long sleeves | At or below 70°F |
| 🧥 Jacket | At or below 65°F |

Thresholds are user-configurable via the config file (see [Configuration](#configuration) below).

### Caching

Cache files are stored in `~/.cache/weather_zsh/` (one `.json` file per location). Cache is **opt-in** — it is only read when `--cache` is passed.

- `--cache` — read from cache if fresh, fetch and write on miss
- `--no-cache` — always fetch live (still writes to cache)
- `--clear-cache` — truncates cache files in place; the next run fetches fresh data

### Configuration

User preferences can be persisted in:

```
~/.config/weather_zsh/config
```

The file must be `chmod 600` (owner read/write only). Only the following keys are accepted — anything else is ignored with a warning:

```zsh
# Location
LAT=34.1478
LON=-118.1445
LOCATION_NAME="Los Angeles, CA"

# Cache
CACHE_TTL=600
CACHE_DIR=~/.cache/weather_zsh

# Clothing recommendations
LONG_SLEEVE_THRESHOLD=70
JACKET_THRESHOLD=65
CLOTHING_ADVICE=true
```

> 📍 **Finding your coordinates?** See [LOCATION.md](LOCATION.md) for a full guide on how to look up your lat/lon, set your location, and understand which variables are actually required.

---

## `create_user.zsh`

Interactive script for creating Linux user accounts. Prompts for all required information, enforces a 30-day password expiration policy, and requires a password change on first login.

### Requirements

- Must be run as root or with `sudo`
- Target system must have `useradd`, `passwd`, and `chage` available (standard on most Linux distributions)
- Default shell for new accounts is `/bin/zsh` — ensure zsh is installed on the target system

### Usage

```zsh
sudo ./create_user.zsh
```

The script is fully interactive and will prompt for:

1. **Username** — must not already exist on the system
2. **Home directory** — defaults to `/home/<username>` if left blank
3. **Password** — entered twice for confirmation; not echoed to terminal

### What it does

1. Creates the user account with `useradd` (home directory, zsh shell)
2. Sets the password with a 30-day expiration via `passwd`
3. Forces a password change on first login via `chage -d 0`

### Example session

```
=== Create New User ===

Enter username: alice
Enter home directory path (default: /home/alice):

Password requirements:
  - Will be set to expire in 30 days
  - User will be prompted to change on first login

Enter password for alice:
Confirm password:

Creating user with the following settings:
  Username: alice
  Home directory: /home/alice
  Shell: /bin/zsh
  Password expiration: 30 days

✔ User 'alice' created successfully
✔ Password set successfully with 30-day expiration
✔ Password will expire on first login (user must change it)

=== Summary ===
User 'alice' has been created with:
  Home directory: /home/alice
  Default shell: /bin/zsh
  Password expiration: 30 days
  First login: User will be prompted to change password
```

---

## General Notes

- All scripts support `-h` / `--help` for full usage documentation
- Scripts are written for **zsh** but aim to be compatible with standard Linux tooling
- Tested on Arch Linux; should work on any systemd-based Linux distribution and macOS (where applicable)

## License

MIT — see [`LICENSE`](LICENSE)
