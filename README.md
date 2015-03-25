This is a quick-and-dirty Arch Linux package for the Linux kernel with
Chromebook Pixel 2015 (Samus) support borrowed from Chromium OS.  The
patches are available in linux-samus for use with other distributions,
but these patches are experimental.  Use them at your own risk.

#### Touchpad and touchscreen

Touchpad and touchscreen support is provided by
linux-samus/chromiumos-samus-touchpad-touchscreen.patch.

These kernel config flags should be set:
```
CONFIG_NVRAM
CONFIG_ACPI_CHROMEOS
CONFIG_CHROMEOS
CONFIG_CHROMEOS_LAPTOP
CONFIG_CHROMEOS_PSTORE
```

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

#### Sound

Sound support is provided by
linux-samus/chromiumos-samus-sound.patch.

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

The SoC audio device shows up as the second ALSA device, so you might want
want to add this to /etc/modprobe.d/alsa-base.conf to change the order:
```
options snd_soc_rt5677 index=0
options snd_hda_intel index=1
```

The ALSA UCM configuration from Chromium OS is included in the repository
in ucm/.  To inset the device to a useable initial state, run from the
root of the repository:
```
$ ALSA_CONFIG_UCM=ucm/ alsaucm -c bdw-rt5677 set _verb HiFi
```
The configuration will persist if you save and restore the ALSA state.

##### Issues

The master volume can be set, but not read, through the mixer interface.

#### Keyboard backlight

Keyboard backlight support is provided by
linux-samus/chromiumos-samus-keyboard-backlight.patch.

These kernel config flags should be set:
```
CONFIG_BACKLIGHT_CHROMEOS_KEYBOARD
CONFIG_LEDS_CHROMEOS_KEYBOARD
```

Brightness can be controlled through /sys/class/leds/chromeos::kbd_backlight.

