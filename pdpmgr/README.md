# pdpmgr

pdpmgr and the associated systemd service are an alternative to the [pdpcontrol
script](https://github.com/obsolescence/pidp10/blob/master/bin/pdpcontrol.sh) provided in the standard
[PiDP-10](https://obsolescence.dev/pidp10.html) installation.

The improvements over the standard pdpcontrol script are:
* supports [tmux](https://github.com/tmux/tmux/wiki) and [dtach](https://dtach.sourceforge.net/) in addition to [screen](https://www.gnu.org/software/screen/) as the session manager
* designed to be run as a [systemd](https://systemd.io/) [user
service](https://wiki.archlinux.org/title/Systemd/User)

Status: **EXPERIMENTAL**

## Startup

The standard PiDP-10 installation starts the simh process on user login,
either by adding a line to the user's .profile file or wayfire.ini

Since it's normal that the user might login to the host system multiple times
from different locations, trying to start simh each time is not ideal.

Instead, we'll use systemd to start simh as a service, either controlled
manually or automatically at boot.

## Installation

Install pdpmgr by copying it to /usr/local/bin/

```bash
sudo cp pdpmgr /usr/local/bin/pdpmgr
sudo chown root: /usr/local/bin/pdpmgr
chmod 755 /usr/local/bin/pdpmgr
```

Install the systemd service as a user service:
```bash
systemctl --user link $(pwd)/pidp10.service
```

This command will create a symlink in ~/.config/systemd/user/ pointing to
pidp10.service in the current directory. Alternatively you could copy the file
there, or create it with a command `systemctl edit --user --force --full pidp10.service`.

To start the service:

```bash
systemctl --user start pidp10
```

To stop the service:

```bash
systemctl --user start pidp10
```

To enable the service to start at boot, first enable lingering for the current
user:

```bash
loginctl enable-linger
```

Then enable the pidp10 service:

```bash
systemctl --user enable pidp10
```

## tmux/dtach support

The standard pdpcontrol script uses
[screen](https://www.gnu.org/software/screen/)
to start simh in the background.
Pdpmgr improves on this by adding support for
[tmux](https://github.com/tmux/tmux/wiki) as well as
[dtach](https://dtach.sourceforge.net/).

When using tmux, a new session "pidp10" is created, with the simh process
running in a window called "simh". Refer to the tmux documentation to learn
how to interact with the session and window.

To configure which terminal multiplexer is used, set the
`PIDP10_SESSION_MANAGER` variable to either `screen`, `tmux`, or `dtach` in
`/etc/default/pdpmgr`.

For example:

```bash
PIDP10_SESSION_MANAGER="tmux"
```

## Choosing the emulated operating system

The standard pdpcontrol script supports selecting the emulated operating
system either by reading the positions of console switches,
or by passing a command line argument.

Since the pdpmgr script is intended to be run as a systemd service, the
command line argument is replaced by a file that specifies the default
emulated OS to start up.

To see the currently selected default, run the command:

```bash
pdpmgr setdefault
```

This will show the currently set default, as well as a list of all available
values that may be set as the default.

The list of available defaults are read from `/opt/pidp10/systems/selections`, and the currently selected default is stored in `/opt/pidp10/systems/current`.

A special case is the value `scansw`, which specifies that the operating
system should be booted based on positions of the console switches.

To set the default OS, for example to boot into `its`, run the command:

```bash
pdpmgr setdefault its
```
