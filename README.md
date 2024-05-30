# Home Assistant Appdeamon script to control your lights
This script allows for extensive control of your lighting based upon defined parameters which you pass in.  This script can be instantiated multiple times across many rooms / lights / time of day permutations.  You have the ability to transistion lights (change from one color / brightness to another) over a period of time, or you may choose an "instantaneous" change.

## Features
* Supports sun events using the suncalc module
* Supports offset (begin times only) for sun events
* Supports a start time setting
* Configurable time to transistion from one state to the next (immediate, or over a period of time)
* Supports multiple different motion sensors (in theory you could use any binary_sensor you wanted, but I haven't tested it with door sensors)
* Can change the brightness and the xy_color of a light
* Configurable time to turn off lights (if set)
* Configured to be a "run once" routine, so you will need a 'tracking toggle' in HA (you will also need some way to turn off the toggle . . . this is up to you to figure out)
* As a "run once" routine, you only have to set "start times" for your configurations, and not have to worry about making sure that the times to change are to the "minute".  Just set a start time for a routine, and it will run at that time and only that time (unless you have the light configured to shut off automatically, at which point the script will keep running).
* Supports the ability to "shut off" other input booleans so that the scripts can be retriggered

## Configuration Values

> [!WARNING]
> Some values are mutually exclusive.  These will be called out in the below documentation.

> [!TIP]
> The power in this script comes from the ability to leverage multiple motion sensors, combined with "time" functions.  This allows you to run multiple instances of this script (it is all self contained).

These are the variables that will go in app.yaml.

| YAML Variable | Type | Required | Default Value | Description|
|---------------|------|----------|---------------|------------|
|light| Array|*| **["light.group_all"]** | The light you want to perform an action on.  It is recommended that at this point in time you use HA groups to group specific lights.  Multiple lights are not supported at thi time. |
|motion_sensor| arra of strings|*|**none**| An array list of the motion sensors you want to use to trigger the settings.|
|trigger_time_start| String / 24 hour format of hour and minute to trigger| | **00:00** | 24 hour representation in the format of 'hh:mm' in string format.  Will be added to the current day to create the trigger time.  This value is ignored if you use the "sun event" setting. |
| trigger_start_offset | int | |**0**| Setting in minutes to adjust the trigger time of a sun event may be a positive or negative number. See the table below for SunCalc events.  EX: if you set this value to '-90'and use the sun event of 'sunset', then the trigger time will be 90 minutes BEFORE sunset. Conversley, a positive number will set the trigger time to AFTER the event.|
|sun_event_start| string| | **'default'** | Used with the property column in the SunCalc Values, will set the trigger time to that event in your location.  Please see trigger_start_offset for more information|
|lat| decimal| *| **0** | Your latitude, needed to get accurate sun location |
|long| decimal| *| **0** | Your longitude, needed to get accurate sun location |
|x_color| decimal| *| **0**| Target value for colors in the x space.  Chosen as this eliminates the need for color temperature.  If you want to keep the lights the same color.  Just pass in the current xy_color x value and the lights won't change |
|y_color| decimal|* |**0**| same as x_color, just for the y color space.  This is the second entry when viewing the state of a light, and is located under xy_color in the light's attributes|
|brightness| int |*|**128**| Target brightness to set.  Any whole number between 0 and 255.|
|time_to_off|int| |**0**| Delay (in seconds) until the light is automatically turned off.  Any non-zero number will cause the light to shut off automatically|
|transistion|bool| |**false**| Set to true to cause the lights to "shift" to the new settings.  This will cause the xy and brightness values to move towards what you have set.  Adjustments are made every second |
|transisition_time_in_seconds| int| required if trasisition is set | **0** | Amount of time in seconds, to transistion to your configured settings|
|state_holder| string | * |  | This is the programatic name of the input boolean (toggle) that you created in HA.|
|input_bools_to_turn_off| array of strings| | | Use this to turn off other state_holder toggles as needed.  This happens upon script start up (initiation)|
|write_to_logs| boolean| | false | Set to true to see debugging information from the script.  This will output varible values to the appdaemon log file |

## SunCalc Values
| Property | Description |
|----------|-------------|
|sunrise	|sunrise (top edge of the sun appears on the horizon)|
|sunriseEnd	|sunrise ends (bottom edge of the sun touches the horizon)|
|goldenHourEnd	|morning golden hour (soft light, best time for photography) ends|
|solarNoon	|solar noon (sun is in the highest position)|
|goldenHour	|evening golden hour starts|
|sunsetStart	|sunset starts (bottom edge of the sun touches the horizon)|
|sunset	|sunset (sun disappears below the horizon, evening civil twilight starts)|
|dusk	|dusk (evening nautical twilight starts)|
|nauticalDusk	|nautical dusk (evening astronomical twilight starts|
|night	|night starts (dark enough for astronomical observations)|
|nadir	|nadir (darkest moment of the night, sun is in the lowest position)|
|nightEnd	|night ends (morning astronomical twilight starts)|
|nauticalDawn	|nautical dawn (morning nautical twilight starts)|
|dawn	|dawn (morning nautical twilight ends, morning civil twilight starts) |


## Example Configuration Triggered by time and shutting off in 5 minutes with the lights taking 20 seconds to adjust
```
lights:
  module: lighting
  class: BedroomLights
  light: ["light.bedroom_lamps"]
  motion_sensor: ["binary_sensor.bedroom_motion_sensor"]
  sun_event_start: none
  trigger_time_start: "10:00"
  lat: 40.1759
  long: -86.0217
  x_color: 0.47
  y_color: 0.337
  brightness: 128
  time_to_off: 300
  transistion: true
  transisition_time_in_seconds: 20
  state_holder: input_boolean.bedroom_sunset
  write_to_logs: true
```

## Example Configuration Triggered 90 minutes before SunSet.  Shuts off in 30 minutes and takes one minute to transistion.
```
lights:
  module: lighting
  class: BedroomLights
  light: ["light.bedroom_lamps"]
  motion_sensor: ["binary_sensor.bedroom_motion_sensor"]
  sun_event_start: "sunset"
  trigger_start_offset: -90
  lat: 40.1759
  long: -86.0217
  x_color: 0.47
  y_color: 0.337
  brightness: 128
  time_to_off: 1800
  transistion: true
  transisition_time_in_seconds: 60
  state_holder: input_boolean.bedroom_sunset
  write_to_logs: true
```

## Example Configuration: Immediately based upon sun event with no shut off. No output to logs
```
lights:
  module: lighting
  class: BedroomLights
  light: ["light.bedroom_lamps"]
  motion_sensor: ["binary_sensor.bedroom_motion_sensor"]
  sun_event_start: "sunset"
  trigger_start_offset: 0
  lat: 40.1759
  long: -86.0217
  x_color: 0.47
  y_color: 0.337
  brightness: 128
  time_to_off: 0
  transistion: false
  state_holder: input_boolean.bedroom_sunset
  write_to_logs: false
```
