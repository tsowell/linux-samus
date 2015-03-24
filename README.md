This is a quick-and-dirty Arch Linux package for the Linux kernel with
Chromebook Pixel 2015 (Samus) support borrowed from Chromium OS.  For
use with other kernel versions, the patches are in
chromiumos-samus-support.patch.

What works:
  * Touchpad and touchscreen
  * Keyboard backlight
  
What doesn't:
  * Audio
  * Other things
  
What has been tested:
  * Nothing - use at your own risk

To use xf86-input-synaptics with the touchpad, you'll need at least the following in your xorg.conf.d:
```
Section "InputClass"
    Identifier "touchpad"
    MatchIsTouchpad "on"
    MatchDevicePath "/dev/input/event*"
    Driver "synaptics"
EndSection
```
