# Setup and Tweaks Arch Linux x86

In this document, all the setup for my Arch system is explained. The objective
for this document is having a guide for all the necessary tweaks and package
installations to have my functional system this includes:

- Having an Optimus setup for Nvidia and Intel cards.
- Controlling brightness controls for Nvidia cards.
- Modifying Logitech hardware color configuration and button setup.
- And more! (see the table of contents to see the full changes.)

# Optimus Setup
The Acer Nitro 5 uses the Nvidia Optimus technology. This allows to have an integrated
intel graphic cards for low performance and improved battery with an nvidia graphics
card for higher performance. Although multiple solutions are given from the Arch Wiki
for managing this, the best solution I could found out of the box is using
[optimus manager](https://github.com/Askannz/optimus-manager).

Using the `lightdm` display manager, this configuration pretty much works out of the
box. Some additional configuration is needed in order to obtain a real battery life
saving by configuring the power management options, in which the PCI power control one
was the most successful one. My optimus configuration file looks something like:

```
[intel]
DRI=3
accel=
driver=modesetting
modeset=yes
tearfree=yes

[nvidia]
DPI=96
PAT=yes
allow_external_gpus=no
dynamic_power_management=no
ignore_abi=no
modeset=yes
options=overclocking

[optimus]
auto_logout=no
pci_power_control=yes
pci_remove=no
pci_reset=no
startup_auto_battery_mode=integrated
startup_auto_extpower_mode=nvidia
startup_mode=nvidia
switching=nouveau
```

With this setup, additional displays can be used without the need to switching back to
the nvidia performance card. Remember to install `mesa-utils` for `glxinfo` so it can
detect the state correctly when it is integrated.

# Time Syncing
For network time syncing, NTP is not suggested as laptops do not have a permanent
network connection and is really slow for time syncing. Instead use `chrony` that is
specifically designed to combat this issues. The Colombian NTP servers can be seen
[this page](https://www.ntppool.org/zone/co). Currently, in my =/etc/chrony.conf= I
have the following servers:

```
server 1.co.pool.ntp.org offline
server 0.south-america.pool.ntp.org offline
server 3.south-america.pool.ntp.org offline
```

To activate the servers when I'm connected to a network, I use the following
[aur package](https://aur.archlinux.org/packages/networkmanager-dispatcher-chrony/).
Remember to install the `chrony` package and enable `chronyd.service`;
additionally, disable and stop `systemd-timesyncd` for enabling chrony properly.

# Logitech Mouse Customization
I have a [Logitech G3000s](https://www.logitechg.com/en-eu/products/
gaming-mice/g300s-gaming-mouse.910-004345.html) mouse; this mouse has some colors to
switch through by default and it can be customized to have different colors.
Additionally, it has some custom buttons that allow to be customized.

Using the `ratslap` command, the color and buttons can be set. To changing the
color to red and having the two middle buttons to change between tabs use:

```bash
sudo ratslap \
      --modify F3 \
      --colour red \
      --G8 LeftCtrl+PageUp \
      --G9 LeftCtrl+PageDown \
      --print F3 \
      --select F3
```

# Touchpad Configuration
For configuring the touchpad for a more natural behavior, the `libinput` package was
used as is the currently use package for managing input devices; for making it work in
Xorg run the following command:

``` bash
sudo pacman -S xf86-input-libinput
```

And restart the graphical environment so the devices are now managed by libinput. Then,
to configure the touchpad for the following features:

- Tapping for left click.
- Tapping with two fingers for right click.
- Tapping with three fingers for middle click (paste).
- Natural scrolling, similar to windows.

Create the file `/etc/X11/xorg.conf.d/30-touchpad.conf` and write:

```
Section "InputClass"
    Identifier "libinput touchpad catchall"
    MatchIsTouchpad "on"
    MatchDevicePath "/dev/input/event*"
    Driver "libinput"
    Option "Tapping" "on"
    Option "ClickMethod" "clickfinger"
    Option "NaturalScrolling" "true"
EndSection
```

And then restart your computer to reload xorg.

# Hard Drive Configuration
To improving quality of life for hard drives, it is important to prevent spinning down
issues. The default values normally set can be to aggressive and deteriorate the
lifespan of a hard drive. For improving this manually, install the `hdparm` package and
do:

``` bash
hdparm -B 127 /dev/XXX
```

This setting will keep spin down without being two aggressive. To make this setting
permanent at reboot, create a udev rule. In the file `/etc/udev/rules.d/69-hdparm.rules`
write the following to automatically the detect the disks to apply the rule:

```
ACTION=="add|change", KERNEL=="sd[a-z]", ATTRS{queue/rotational}=="1", RUN+="/usr/bin/hdparm -B 127 /dev/%k"
```

Related page:
[Arch Linux wiki.]\
(https://wiki.archlinux.org/title/Hdparm#Power_management_configuration)

# Keyboard Layout Configuration
For my personal laptop, I use the following configuration for my keyboard layout:
- The US keyboard layout as I find the best one for programming.
- The `altgr-intl` variant in order to write in spanish easily and quickly.
- The Caps Lock and Escape keys are swapped in order for improved VIM-like usage.

For setting this in an Xorg server, use the following command:

```
localectl --no-convert set-x11-keymap us evdev altgr-intl caps:swapescape
```

# XDG Base Directory Specification
The XDG Base Directory Specification is a directory specification which hopes to protect
the user home directory from being spammed with multiple unnecessary directories that
are used to store data and configuration from multiple apps.
[Read the specification here.]\
(https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)

For complying with the standard, set the following variables in the file
`/etc/profile.d/xdg_compliance.sh`:

```
export XDG_CONFIG_HOME="$HOME"/.config
export XDG_CACHE_HOME="$HOME"/.cache
export XDG_DATA_HOME="$HOME"/.local/share
```

Other global variables I set for multiple other apps for complying with the
specification are:

```
export XINITRC="$XDG_CONFIG_HOME"/X11/xinitrc

export GTK2_RC_FILES="$XDG_CONFIG_HOME"/gtk-2.0/gtkrc

export IPYTHONDIR="$XDG_CONFIG_HOME"/jupyter
export JUPYTER_CONFIG_DIR="$XDG_CONFIG_HOME"/jupyter

export CARGO_HOME="$XDG_DATA_HOME"/cargo
export RUSTUP_HOME="$XDG_DATA_HOME"/rustup

export ASPELL_CONF="per-conf $XDG_CONFIG_HOME/aspell/aspell.conf;"
export ASPELL_CONF="${ASPELL_CONF} personal $XDG_CONFIG_HOME/aspell/en.pws;"
export ASPELL_CONF="${ASPELL_CONF} repl $XDG_CONFIG_HOME/aspell/en.prepl;"

export NPM_CONFIG_USERCONFIG="$XDG_CONFIG_HOME"/npm/npmrc

export LESSHISTFILE="$XDG_CACHE_HOME"/less/history

export PASSWORD_STORE_DIR="$XDG_DATA_HOME"/pass

export TEXMFVAR="$XDG_CACHE_HOME"/texlive/texmf-var

export PYLINTHOME="$XDG_CACHE_HOME"/pylint

export DOCKER_CONFIG="$XDG_CONFIG_HOME"/docker

export AWS_SHARED_CREDENTIALS_FILE="$XDG_CONFIG_HOME"/aws/credentials
export AWS_CONFIG_FILE="$XDG_CONFIG_HOME"/aws/config
```

# QT5 Platform
For configuring the QT5 applications, I use `qt5ct` which allows for configuration to
the platform similar as `lxapperance` to the X11 server. In this manner, to configure
the QT apps `qt5ct` write the following file `/etc/profile.d/qt5_vars.sh`:

```
export QT_QPA_PLATFORMTHEME=qt5ct
```

# Network Performance Tweaks
```
net.ipv4.tcp_fastopen=3
net.core.netdev_max_backlog=1638
```
