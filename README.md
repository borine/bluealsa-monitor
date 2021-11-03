# Automated ALSA Configuration For BlueALSA PCMs

This project is a companion to, and requires installation of, BlueALSA - see
https://github.com/Arkq/bluez-alsa

> __*This is Alpha pre-release code. It requires capabilities of the BlueALSA
> `bluealsa-cli` utility that are available in the upstream
> bluez-alsa project sources, but not yet in a released version.*__

Applications can open PCM streams to bluetooth devices connected through
BlueALSA without any need for specific ALSA configuration entries (see
https://github.com/Arkq/bluez-alsa/blob/master/doc/bluealsa-plugins.7.rst).
However, there is no equivalent way for an ALSA application to obtain
a list of available bluetooth PCMs. For command-line users, the utility
`bluealsa-aplay` will produce such a list, but for GUI applications we
need a way have the connected bluetooth PCMs appear through the ALSA
 _*namehint*_ API in the same way as sound card devices.

This project provides a bash script and associated system files to generate
dynamic ALSA configuration changes as bluetooth audio devices connect or
disconnect such that those devices appear as PCMs in the namehint API. The
current version supports only a single running instance of the `bluealsa`
service.

Optionally, the script can also load BlueALSA PCMs into PulseAudio as devices.
This enables BlueALSA to be used in conjunction with PulseAudio. It is first
necessary to disable PulseAudio's own bluetooth modules for this to work.

The script relies on the `bluealsa-cli` utility which is part of the above
bluez-alsa project. bluez-alsa v3.2.0 or later is required as earlier versions
of `bluealsa-cli` did not have the needed functions for this script. It is
necessary to include the configure option `--enable-cli` when building BlueALSA
to build this utility.

_namehint_ configuration entries are created that cause ALSA client
applications to include connected bluetooth PCMs in their listed PCM devices.
For this to work correctly it requires the application to use the "official"
__namehint__ ALSA API for listing devices. There are some applications which
ignore that API and attempt to parse the ALSA configuration using their own
bespoke code; those applications generally will fail to list BlueALSA devices.

The script is designed to be run in a user session, in the background, and
monitors D-Bus messages from BlueALSA. An ALSA config file is created and
managed by the script, and this config file is included from the user's
`~/.asoundrc` file automatically when the script starts, and is removed when the
script stops. Optionally this file inclusion can be disabled, to support use
of the `ALSA_CONFIG_PATH` environment variable.

The namehint entries include a description string consisting of the device alias
(i.e. its local name), profile, and codec. An example output of `aplay -L`

```shell

$ aplay -L
bluealsa:DEV=C4:67:B5:67:89:AB,PROFILE=sco
    Libratone Zipp Mini, HFP (CVSD)
bluealsa:DEV=C4:67:B5:67:89:AB,PROFILE=a2dp
    Libratone Zipp Mini, A2DP (aptX)

```
To send audio to that device, use the device id as given by `aplay -L`:
```shell

$ aplay -D bluealsa:DEV=01:23:45:67:89:AB,PROFILE=a2dp music.wav

```

> _to make it easier to type device ids on the command line with `aplay` and
> `arecord`, use a bash completion script, such as
>  https://github.com/borine/alsa-utils-completions_


## Installation

A `meson` build specification file is included to simplify the installation. To
use it, do:

```shell
meson setup builddir
sudo meson install -C builddir
```

The default install prefix is `/usr/local`. To install on a production system
this should be overridden in the `setup` command:
```
meson setup --prefix=/usr builddir
sudo meson install -C builddir
```

To disable installation of the udev sudoers file, run setup as: 
```
meson setup -D udev=disabled builddir
sudo meson install -C builddir
```

## Usage

The script should be run as a __user__ service, (__not as root__), and a systemd
user service file is included for that purpose if desired (see below). There are
some optional command line arguments:

__-B, --dbus__

Select the bluealsa d-bus suffix. Not normally required. See the bluez-alsa
documentation for more information.

__-d, --default__

Make BlueALSA PCM the ALSA default (`pcm.!default` in the config file) when
connected. The most recently connected device is used. Takes an optional
value, a comma-separated list consisting of one or more of the tokens _a2dp_, 
_sco_, _capture_, _playback_ to limit the device types that will be accepted as
defaults. Only one of _a2dp_ or _sco_ may be specified. If this argument is used
with no value given, the default is _a2dp,capture,playback_.
For example:
```
bluealsa-monitor --default=a2dp,playback
```
This will set up the ALSA default playback PCM to be the most recently connected
A2DP playback device. The default capture device will remain as the system
default. If no A2DP playback device is connected, then the ALSA default playback
PCM will revert to the system default.

If you do not wish the system default PCM to be used as fallback, you can
choose a different PCM by defining the config keys
`bluealsa.default.capture.fallback` and/or
`bluealsa.default.playback.fallback`
in your `~/.asoundrc` file.
For example:
```
bluealsa.default.playback.fallback "plughw:1,0"
```

__-f, --file__

Absolute path to the dynamic ALSA config file created by this script. Defaults
to `$HOME/.asoundrc-bluealsa-monitor` (except when `--global` option is given).
It is important that this file is __*not `$HOME/.asoundrc`*__.
Not normally required.

__-g,--global__

Do not add an include directive to the user's `~/.asoundrc` file. When this option
is used the user should define the environment variable `ALSA_CONFIG_PATH` to
include the file generated by the bluealsa-monitor service. For example
```
ALSA_CONFIG_PATH=/usr/share/alsa/alsa.conf:/var/local/lib/bluealsa-monitor/asoundrc
```

When this option is given, the default dynamic ALSA config file is:
 `/var/local/lib/bluealsa-monitor/asoundrc`

__-u, --udev__

Emit a synthesized udev event on BlueALSA device connect and disconnect. This 
option may be useful for some applications that do not refresh their audio
device list unless a soundcard change is signalled via udev (e.g. `kodi`).
The script will invoke `sudo` with no password to raise necessary privileges
when needed, so it is important that the user has permission to do this.
This project includes a sudoers config file that grants required permission to
all members of the `audio` group. The permission is specifically limited to
raising a udev `change` event on the default sound card; no other permission
is granted. On systems that do not support `sudo`, alternative means to manage
privileges will be required.

__-p, --pulse__

Add connected devices to PulseAudio. Takes an optional value, a comma-separated
list consisting of one or more of the tokens _a2dp_, _sco_, _capture_, _playback_
to limit the device types that will be used to a particular profile
(either a2dp or sco) and/or a specific direction (playback or capture).
The defaukt is to add all connected devices to PulseAudio.

__-i, --icon__

Select a particular icon to be used by PulseAudio for the BlueALSA devices.
Defaults to _bluetooth_. The available icons will depend on your distribution.

## systemd

The script can be started and stopped manually using:
```shell
systemctl --user start bluealsa-monitor
```
```shell
systemctl --user stop bluealsa-monitor
```

To enable the script to start automatically on user login, the user should enter
the command:

```shell
systemctl --user enable bluealsa-monitor
```

The included systemd service file starts the script with no command-line
options. The recommended way to add options is to use systemctl. A quirk of
systemd is that the existing service command line has to be cancelled in order
to insert a new one.

Suppose we want to enable the udev trigger and restrict the default to A2DP
playback only - so we do the following; 
- start the systemctl editor:
```shell
systemctl --user edit bluealsa-monitor.service
```
-  then insert the following lines:
```
[Service]
ExecStart=
ExecStart=/usr/local/bin/bluealsa-monitor --udev --default=a2dp,playback
```
- save and exit from the editor, then restart the service:
```shell
systemctl --user restart bluealsa-monitor
```

To prevent the script from starting at login, the user should enter the command:
```shell
systemctl --user disable bluealsa-monitor
```

