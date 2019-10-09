# Auto Weather Change weather-utils
### Written by [@LegendKiller21](https://github.com/legendkiller21) and [@multmokia](https://github.com/multimokia)
#### If used, please leave credit to the creators above or link the repository.
#### Please do not claim the code in these files as your own.

With that said:
These utilities offer an interface with the [openweathermap API](https://openweathermap.org/api) to be used within Ren'Py.

These utilities offer connection checking, getting weather observations, sunrise and sunset times, and a way to (while simplified) convert them to a form your game can use.

In order to be able to use this, the user must generate an API key from the link above. (Alternatively you can do this and release it with your game, but it's free if you generate it yourself and on a per user basis)

In order to get weather observations for your desired location, you'll need to either prompt the user for what city they live in, or alternatively use geocoder and get their location via their ip (libraries are included to do so)

**NOTE:** It is possible that there are cities missing from `citylookup.txt`. If this is the case, please feel free to submit a pull request adding the location, simply add a line in the form of `city_name,country_code,latitude,longitude,state_or_province` (Your help is appreciated)

# Globals:
## `owm`:
- The `openweathermap` object which is used to get weather observations from

## `SUN_KW`:
- Keywords for sun weather

## `OVERCAST_KW`:
- Keywords for overcast weather (**NOTE:** currently unused)

## `RAIN_KW`:
- Keywords for rain weather

## `THUNDER_KW`:
- Keywords for thunder weather

## `SNOW_KW`:
- Keywords for snow weather

## `BASE_URL`:
- The base url for the open weather map api requests

## `weather_offline_timeout`:
- Stores the `datetime.datetime` of when the `awc_offlineTimerCheck()` function declares you offline.


# Reference Utilities:
## `country_code_lookup`:
- `dict()`
- Allows country code to full country name lookups
Structure:
```python
{
    "country_code": "country_name",
    ...
}
```

## `city_lookup`:
- `dict()`
- Allows lookups to get all locations of a certain city name.
Structure:
```python
{
    "city_name": [
        ("country_code", latitude, longitude, state_or_province),
        ...
    ],
    ...
}
```


# Persistent variables:
## `_awc_player_location`:
- `dict()`
- In order to check weather, a set of either must be complete:
  - `city, country, loc_pref`
  - `lat, lon, loc_pref`
- City - The city the player lives in.
- Country - The country the player lives in.
  - **NOTE:** for US, latlong is the best approach due to duplicate city names in differing states.
- lat - Player's latitude.
- lon - Player's longitude.
- loc_pref - Preferred way for `awc_getObservation()` to get the observation for location.
  - Acceptable values: `citycountry`, `latlon`.
  - **NOTE:** This should ideally be set when prompting the user for their location. If their city is the only one in the world with that name, `citycountry` is the best option. Otherwise, you'll have to use `latlon`.

Structure:
```python
{
    "city": None,
    "country": None,
    "lat": None,
    "lon": None,
    "loc_pref": None
}
```

## `_awc_API_key`:
- `string`
- Stores the api key to get weather observations
- Use `awc_apiKeySetup()` to have this set up properly


# Functions:
## `awc_textURL(url)`:
- When given a url to check, it tests if it's possible to open.
- If successful, returns `True`. If not, it returns the error code. If no connection, returns `False`.

## `awc_testConnection()`:
- Checks if the user has connection to the internet.
- If there's connection, returns `True`, otherwise returns `False`.

## `awc_isInvalidAPIKey(api_key)`:
- `api_key` - The api key to check
- Checks if the api key is invalid by testing the URL.
- If the error code is `401`, the api key is invalid and this returns `False`. If not, it returns `True`.
- With this in mind, location should be verified too.

## `awc_apiKeySetup()`:
- Checks for an api key. If it is loadable and a valid key, it is then stored to `persistent._awc_API_key`.
- returns `True` if the api key was set-up successfully, `False` otherwise.

## `awc_isInvalidLocation(city_code, country_code)`:
- `city_code` - city to check
- `country_code` - country to check (optional)
- Checks if location is valid by checking for a site error code. If the site returns a `404`, it is invalid and this returns `True`. If not, `False`.
- **NOTE:** Also returns False if api key is invalid.

## `awc_getCiyCountry(city_name)`:
- `city_name` - the city to get the country of
- Gets the country code from the first result of `owm.weather_at_place` based on `city_name`.
- **NOTE:** Assumes `city_name` is valid.

## `awc_getFriendlyCityCountry(city_name)`:
- `city_name` - the city to get the full country display name of
- Gets the proper display name for the country the city is in (for use in dialogue or menus, etc.)
- **NOTE:** If not in the country code lookup, returns the country code instead (please make a pull request to fix this if you find this to be the case)

## `awc_getFriendlyCountry(country_code)`:
- `country_code` - the country code to get a friendly display name for.
- Like the previous function, this returns the country code back if it wasn't found in the lookup

## `awc_getSunriseDT()`:
- Returns a `datetime.datetime` of sunrise for the player location (DST and timezones accounted for)

## `awc_getSunsetDT()`:
- Returns a `datetime.datetime` for sunset in the player location (DST and timezones accounted for)

## `awc_dtToMASTime(dt)`:
- `dt` - `datetime.datetime` object to convert
- Converts a `datetime.datetime` to `MAS` (*Monika After Story*) time (`int`, specific to the settings menu)

## `awc_lookupCity(city_name)`:
- `city_name` - city to lookup
- Checks the `city_lookup` dict to find all locations for the city
  - Returned in a list of tuples in the form:
```python
[
    (city_name, country_code, latitude, longitude, state_or_province),
    ...
]
```

## `awc_hasMultipleLocations(city_name)`:
- `city_name` - city to check for multiple locations
- `True` if the list for the city lookup has a length greater than 1. `False` otherwise

## `awc_buildCityMenuItems(city_name)`:
- `city_name` - city to build a location menu list for
- Builds a list of menu items storing the display name, latlon tuple, and placeholder vars for the `gen_scrollable_menu` (For *Monika After Story*)

## `awc_buildWeatherLocation(city_name, country_code)`:
- `city_name` - city to build location code with.
- `country_code` - country code to build location code with.
- **NOTE:** this is pretty much just string concat shorthand-ish.

## `awc_savePlayerLatLonTup(latlon)`:
- `latlon` - Tuple in the form `(latitude, longitude)`.
- Saves this tuple to the appropriate sections in the `_awc_player_location` dict.
- **NOTE:** does not set `loc_pref`.

## `awc_buildWeatherLocationTup(citycountry)`:
- `citycountry` - Tuple in the form `(city, country_code)`
- Tuple equivalent of `awc_buildWeatherLocation()`.

## `awc_hasPlayerCoords()`:
- Checks if we have a latitude and longitude in the `_awc_player_location` dict.

## `awc_hasPlayerCityCountry()`:
- Checks if we have a city and country in the `_awc_player_location` dict.

## `awc_getPlayerCoords()`:
- Gets the player's coords from the dict and returns them in a `(latitude, longitude)` tuple.
- **NOTE:** Assumes that we have player coords.

## `awc_getPlayerCityCountry()`:
- Gets the player's city and country code from the `_awc_player_location` dict.
- **NOTE:** like `awc_getPlayerCoords()`, this assumes we have a city and country.

## `awc_weathByCityCountryTup(citycountry)`:
- `citycountry` - `(city, country_code)` tuple (optional). If not provided `citycountry` is retrieved by `awc_getPlayerCityCountry()`.
- **NOTE:** This still does not check for having player citycountry.

## `awc_weathByCoords(coords):`
- `coords` - `(latitude, longitude)` tuple (optional). If not provided, `coords` is retrieved by `awc_getPlayerCoords()`
- **NOTE:** This still does not check for having player coords.

## `awc_buildCityLookupDict()`:
- Builds the lookup for city names and their locations.

## `awc_canGetAPIWeath()`:
- Checks if we can get weather from the api via ensuring there's either city and country, player coords, and that we've got a valid api key.
- `True` if we can get weather, `False` otherwise.

## `awc_offlineTimerCheck()`:
- Checks to see whether or not we need to start the offline timer.
- If there's no connection and the offline timer isn't started (`None`), then the timer is started. Returns `False` as the 30 minute timeout hasn't been met.
- If there's no connection and the timer has already been started before (subsequent check from the first) and the timer hasn't passed 30 minutes from start, returns `False` as this has not timed out yet.
- If there's no connection after the 30 minute timeout, return `True`. Connection timed out for over 30 minutes and we still cannot get weather. You should use an alternate way of setting the weather at least until connection can be restored.
- If at any point connection is restored and this function is called, the timeout is reset to `None` and and it returns `False`.

## `awc_getObservation()`:
- Gets the weather observation for the player's location via the preferred locator.
- Returns `None` if there's no preferred locator (`pref_loc`) or if there is a preferred locator and there's no player location for that locator.

## `awc_weathFromAPI()`:
- Gets weather based off api results in a string form
- `"sun"` for clear weather
- `"overcast"` for overcast weather
- `"rain"` for rain weather
- `"thunder"` for thunder weather
- `"snow"` for snow weather
- (Feel free to change these returns to better suit your game)

## `awc_isSunWeather()`:
- Checks if the current weather description or detailed weather description is in the `SUN_KW` global list.
- `True` if so, `False` otherwise.

## `awc_isOvercastWeather()`:
- Checks if the current cloud percentage is greater than 50%.
- `True` if so, `False` otherwise.

## `awc_isRainWeather()`:
- Checks if the current weather description or detailed weather description is in the `RAIN_KW` global list.
- `True` if so, `False` otherwise.

## `awc_isThunderWeather()`:
- Checks if the current weather description or detailed weather description is in the `THUNDER_KW` global list.
- `True` if so, `False` otherwise.

## `awc_isSnowWeather()`:
- Checks if the current weather description or detailed weather description is in the `SNOW_KW` global list.
- `True` if so, `False` otherwise.