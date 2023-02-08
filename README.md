Semi-automatic logbook for Signal K
===================================

Status: ready for the first test runs

This application provides both a server-side plugin and the user interface for maintaining semi-automatic logbooks with [Signal K](https://signalk.org). Just like traditional logbooks, you can write an entry at any time. However, there are several things that are done automatically for you:

* Entries written when starting/ending a trip (requires [signalk-autostate](https://github.com/meri-imperiumi/signalk-autostate) plugin)
* When underway, an entry is created every hour recording the current conditions

## User interface

The logbook presents a web-based user interface as part of the [Signal K](https://signalk.org) administration interface. The features should work fine on both desktop and mobile browsers.

Adding a log entry:
![Add entry](https://i.imgur.com/0M7CdOY.png)

Traditional logbook view:
![Logbook as table](https://i.imgur.com/Xa6XNyh.png)
![Editing an entry](https://i.imgur.com/CDD57LQ.png)

Log entries on a map:
![Map view](https://i.imgur.com/G7eqa5Y.png)

## Data storage and format

This logbook app writes the logs to disk using [YAML format](https://en.wikipedia.org/wiki/YAML) which combines machine readability with at least some degree of human readability.

Logs are stored on a file per day basis at `~/.signalk/plugin-config-data/signalk-logbook/YYYY-MM-DD.yml` 
If there are no entries for a given day, no file gets written.

Note: unlike Signal K itself, the log entries are written using "human-friendly" units, so degrees, knots, etc. They look something like:

```yaml
- datetime: 2014-08-15T19:00:19.546Z
  position:
    longitude: 24.7363006
    latitude: 59.7243978
    source: GPS
  heading: 202
  speed:
    stw: 12.5
    sog: 11.8
  log: 9.6
  waypoint: null
  barometer: 1008.71
  wind:
    speed: 13.7
    direction: 283
  text: Set 1st reef on mainsail
  author: bergie
```

It is a good idea to set up automatic backups of these files off the vessel, for example to [GitHub](https://github.com) or some other cloud storage service. How to handle this backup is out of the scope of this plugin.

## API

Other applications can also use the logbook API for retrieving and writing log entries.

### `GET plugins/signalk-logbook/logs`

Returns a list of dates we have logbook entries for.

### `GET plugins/signalk-logbook/logs/YYYY-MM-DD`

Returns all logbook entries for the given date.

### `POST plugins/signalk-logbook/logs`

Add a log entry by sending a JSON object with the followin keys:

* `text`: Log entry text
* `ago`: How many minutes ago the log entry is for, 0-15

All other logbook data for that period is recorded automatically.

TODO: How about recording other non-automated data like crew/skipper changes, sea state, ...?

### `GET plugins/signalk-logbook/logs/YYYY-MM-DD/YYYY-MM-DDThh:mm:ssZ`

Return a single logbook entry, as identified by its datetime value.

### `PUT plugins/signalk-logbook/logs/YYYY-MM-DD/YYYY-MM-DDThh:mm:ssZ`

Update a single logbook entry, as identified by its datetime value.

## Ideas

Some additional ideas for the future:

* Entries written when turning the engine on or off (requires engine _on state_ or RPMs to be available to Signal K)
* Entries written when reefing or changing the sail plan (requires [sailsconfiguration](https://github.com/SignalK/sailsconfiguration) plugin)
* Enable creating additional rules for automated entries when certain things happen (for example, when turning on a watermaker).
* We could ship easy systemd unit files for setting up backups to popular locations, like pushing to a git repo
* One-time script for populating logbook from InfluxDB entries when starting to use the system

## Changes

* 0.1.2 (2023-02-08)
  - Implemented entry deletion
  - Fixed issue with initial load if logging in within this webapp (#5)
* 0.1.0 (2023-02-03)
  - Initial release
