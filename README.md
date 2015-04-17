### Linux kernel for Chromebook Pixel 2015 (Samus)

This repository contains quick-and-dirty patches to mainline Linux with support
for the Chromebook Pixel 2015 (Samus) borrowed from Chromium OS.  Currently the
touchpad, the touchscreen, sound, and the keyboard backlight are working.  This
is useful as a stopgap until mainline has proper support for the hardware.  The
kernel patches are in arch/linux-samus for use with other distributions, and
binary packages are available in the Releases section for Arch Linux and for
Ubuntu.

The Arch kernel package, linux-samus, is also available in the AUR.

#### Installation

##### Ubuntu

###### Install kernel packages

Download the [release tarball for
Ubuntu](https://github.com/tsowell/linux-samus/releases/download/v0.2.0/linux-samus-ubuntu-0.2.0.tar),
untar it, and install the packages:

```
$ curl -LO https://github.com/tsowell/linux-samus/releases/download/v0.2.0/linux-samus-ubuntu-0.2.0.tar
$ tar xvf linux-samus-ubuntu-0.2.0.tar
$ cd linux-samus-ubuntu-0.2.0
$ sudo dpkg -i *.deb
```

###### Edit GRUB configuration

Set the linux-samus kernel as GRUB's default:

```
sudo grub-set-default "Ubuntu, with Linux 3.19.0-11.11+samus-1-generic"
sudo update-grub
```

###### Initialize the audio device state

Reboot and then load the ALSA UCM (included in the release tarball) to
initialize the audio device state:

```
$ cd linux-samus-ubuntu-0.2.0
$ ALSA_CONFIG_UCM=ucm/ alsaucm -c bdw-rt5677 set _verb HiFi 
```

##### Arch

###### Install kernel packages

Download the [release tarball for
Arch](https://github.com/tsowell/linux-samus/releases/download/v0.2.0/linux-samus-arch-0.2.0.tar),
untar it, and install the packages:

```
$ curl -LO https://github.com/tsowell/linux-samus/releases/download/v0.2.0/linux-samus-arch-0.2.0.tar
$ tar xvf linux-samus-arch-0.2.0.tar
$ cd linux-samus-arch-0.2.0
$ sudo pacman -U *.pkg.tar.xz
```

###### Edit GRUB configuration

Generate a new grub.cfg with grub-mkconfig:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

###### Configure Synaptics

To use xf86-input-synaptics with the touchpad, you'll need at least the
following in your xorg.conf.d:

```
Section "InputClass"
    Identifier "touchpad"
    MatchIsTouchpad "on"
    MatchDevicePath "/dev/input/event*"
    Driver "synaptics"
EndSection
```

###### Change ALSA device order

The SoC audio device shows up as the second ALSA device, so you might want to
add this to /etc/modprobe.d/alsa-base.conf to change the order:

```
options snd_soc_rt5677 index=0
options snd_hda_intel index=1
```

###### Initialize the audio device state

Reboot and then load the ALSA UCM (included in the release tarball) to
initialize the audio device state:

```
$ cd linux-samus-arch-0.2.0
$ ALSA_CONFIG_UCM=ucm/ alsaucm -c bdw-rt5677 set _verb HiFi 
```

NOTE: If you're running from an SD card (slow), your Pixel might appear to
hang for a few minutes while it syncs to disk before rebooting.  I recommend
running "sudo sync" manually before rebooting after a lot of disk activity
(like building and installing a kernel package).

###### Issues

The master volume can be set, but not read, through the mixer interface.

#### Sound

##### Persisting state

In Arch, alsa-utils provides systemd configuration files to save and restore
the audio device state once the UCM has been applied to initialize it.  You can
manually save the state with `alsactl store`.

##### Headphone jack

Audio must be rerouted in software from the internal speakers to the headphone
jack when something is plugged in.  If you first install the UCM files by
copying the directory `common/ucm/bdw-rt5677` to `/usr/share/alsa/ucm`, you can
use, for example, `acpid` to do the routing automatically by adding this case
to `/etc/acpi/handler.sh`:

```
    jack/headphone)
        case "$3" in
            plug)
                logger "headphone plugged"
                alsaucm -c bdw-rt5677 set _verb HiFi set _enadev Headphone
                ;;
            unplug)
                logger "headphone unplugged"
                alsaucm -c bdw-rt5677 set _verb HiFi set _disdev Headphone
                ;;
            *)
                logger "ACPI action undefined: $3"
                ;;
        esac
        ;;
```

#### Keyboard backlight

Brightness can be controlled through /sys/class/leds/chromeos::kbd_backlight.

#### Patches

The patches in arch/linux-samus apply to Linux 3.19.2.

##### Touchpad and touchscreen

Touchpad and touchscreen support is provided by
arch/linux-samus/chromiumos-samus-touchpad-touchscreen.patch.

These kernel config flags should be set:
```
CONFIG_NVRAM (must be 'y', chromeos_acpi will not work with nvram as module)
CONFIG_ACPI_CHROMEOS
CONFIG_CHROMEOS
CONFIG_CHROMEOS_LAPTOP
CONFIG_CHROMEOS_PSTORE
```

##### Sound

Sound support is provided by
arch/linux-samus/chromiumos-samus-sound.patch.

These kernel config flags should be set:
```
CONFIG_SND_SOC
CONFIG_SND_SOC_I2C_AND_SPI
CONFIG_SND_SOC_INTEL_BDW_RT5677_MACH
CONFIG_SND_SOC_INTEL_BROADWELL_MACH
CONFIG_SND_SOC_INTEL_HASWELL
CONFIG_SND_SOC_INTEL_SST
CONFIG_SND_SOC_INTEL_SST_ACPI
CONFIG_SND_SOC_RL6231
CONFIG_SND_SOC_RT286
CONFIG_SND_SOC_RT5677
CONFIG_SND_SOC_RT5677_SPI
```

##### Keyboard backlight

Keyboard backlight support is provided by
arch/linux-samus/chromiumos-samus-keyboard-backlight.patch.

These kernel config flags should be set:
```
CONFIG_BACKLIGHT_CHROMEOS_KEYBOARD
CONFIG_LEDS_CHROMEOS_KEYBOARD
```

