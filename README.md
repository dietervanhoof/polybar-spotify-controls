# polybar-spotify-controls
This set of modules provides controls for spotify that can be added to [polybar](https://github.com/jaagr/polybar).

[![paused](https://i.imgur.com/Wx4vHPr.png)](https://i.imgur.com/Wx4vHPr.png)

[![playing](https://i.imgur.com/wb8ASGo.png)](https://i.imgur.com/wb8ASGo.png)

Specifically it contains:
- The currently playing song
- Next song
- Play/Pause button
- Previous song

## Dependencies
- Python 3
- Python `dbus` module
- Python `GObject3` module
- Polybar with the IPC module
- dbus

## How it works
The Python dbus module is used to listen for `PropertiesChanged` events. The playback status updates are used to update the character printed onto polybar using the IPC module.

Whenever one of these updates arrive in python, a message is sent the correct module. This is done by outputting a message to a symlink file pointing to the correct polybar (this needs to be set up).
The following modules have IPC hooks set up.
- `module/spotify` (to update the song name and artist)
- `module/playpause` (to update the button from play to pause or the other way around)

### playpause
There are 3 hooks defined on `playpause`:
~~~ ini
; Default (no symbol)
hook-0 = echo ""
; Playing (pause symbol)
hook-1 = echo ""
; Paused (play symbol)
hook-2 = echo ""
~~~
The Python script sends messages to hook-1 (2) or hook-2 (3).
### spotify
There are 2 hooks defined on `spotify`:
~~~ ini
hook-0 = echo ""
hook-1 = python3 ~/scripts/spotify/spotify_status.py
~~~
The spotify hook-1 (2) will run another python script which returns the current song and use the return data as text value.

## Installation
### Set up
- Install all dependencies
- Copy all scripts to `~/scripts`
- Have polybar start up on boot- Have the `scripts/spotify/launchlistener.sh` script start up on boot
- Make sure the bars you want to add it to has [IPC enabled](https://github.com/jaagr/polybar/wiki/Module:-ipc)

### Adding all modules
Add all of the desired modules to any bar and modify to your likings. This is my setup:
~~~ ini
[module/previous]
type = custom/script
interval = 86400
format = "%{T3}<label>"
format-padding = 5
; Previous song icon
exec = echo ""
; Check if spotify is running before displaying the icon
exec-if = "pgrep spotify -x"
format-underline = #1db954
line-size = 1
click-left = "dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.Previous"

[module/next]
type = custom/script
interval = 86400
format = "%{T3}<label>"
format-padding = 5
; Next song icon
exec = echo ""
; Check if spotify is running before displaying the icon
exec-if = "pgrep spotify -x"
format-underline = #1db954
line-size = 1
click-left = "dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.Next"

[module/playpause]
type = custom/ipc
; Default
hook-0 = echo ""
; Playing
hook-1 = echo ""
; Paused
hook-2 = echo ""
initial = 1
format-underline = #1db954
line-size = 1
click-left = "dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player.PlayPause"

[module/spotify]
type = custom/ipc
hook-0 = echo ""
hook-1 = python3 ~/scripts/spotify/spotify_status.py
initial = 1
format-padding = 4
format-underline = #1db954
line-size = 1
; [i3wm only] - Uncomment the below line to focus on Spotify when clicking on the song name (credits to https://github.com/Esya)
; click-left = i3-msg '[class="Spotify"] focus'
~~~
Add this to the bar you'd like your controls to be on:
~~~ ini
modules-right = spotify previous playpause next
~~~

# Credits
- [atareao](https://github.com/atareao)
    - [my-weather-indicator](https://github.com/atareao/my-weather-indicator) for the [unwrap function](https://github.com/atareao/my-weather-indicator/blob/da7b4f9827c601e7d7fe449263498451d1465a6f/src/networkmanayer.py#L69).
- [stylesuxx](https://github.com/stylesuxx): 
    - [python-dbus-examples](https://github.com/stylesuxx/python-dbus-examples) for the [python dbus receiver code](https://github.com/stylesuxx/python-dbus-examples/blob/master/receiver.py)
- [Jvanrhijn](https://github.com/Jvanrhijn):
    - [polybar-spotify](https://github.com/Jvanrhijn/polybar-spotify) for the [spotify_status.py script](https://github.com/Jvanrhijn/polybar-spotify/blob/master/spotify_status.py)
