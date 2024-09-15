# keyaboard-remap-guide
A user-friendly guide to remap keyboard keys in Linux.

### Knowledge base

Before begin, it's better to explore these resources:
- About **sysfs** in [Wikipedia](https://en.wikipedia.org/wiki/Sysfs) or [Linux kernel docs](https://www.kernel.org/doc/html/v6.11-rc4/filesystems/sysfs.html).
- About **udev** in [Wikipedia](https://en.wikipedia.org/wiki/Udev) or [Arch docs](https://wiki.archlinux.org/title/Udev).
- What is the sysfs [**modalias**](https://wiki.archlinux.org/title/Modalias).
- What is the [**scancodes**](https://en.wikipedia.org/wiki/Scancode).
- How to identify [**scancodes**](https://wiki.archlinux.org/title/Keyboard_input).
- How to map [**scancodes** to **keycodes**](https://wiki.archlinux.org/title/Map_scancodes_to_keycodes).

In short, keyboards send **scancodes** to the computer and the Linux kernel maps the **scancodes** to **keycodes**. The **keycodes** are some interpretation of **scancodes**. They exist because different keyboards can send different **scancodes** to the same button, or the same **scancode** to different buttons. On a ThinkPad keyboard, the screenshot tool button sends a **scancode** of 46, while the `Q` button sends the same **scancode**, for example.

### A step-by-step example

So in this guide we will map **scancodes** to **keycodes** using **udev**. We will take a ThinkPad T14 Gen 3 and map `Fn + PrtScr` to the `Menu` **keycode** so that this combination opens a context menu.

#### 1. Find the right input device

First we should select the input device we want to work with. They are listed in the `/dev/input` directory as `event1`, `event2`, `event3` etc., and the easiest way to find the one we need is to run the `evtest` utility:
```
# evtest
```
The utility will print a list of devices, by selecting one of them, we can get reports by pressing buttons on the selected input device.

So we can find that the device **AT Translated Set 2 keyboard** is listed as `/dev/input/event3` on the ThinkPad T14 Gen 3, but note that most ThinkPad keyboards work as two separate input devices: **AT Translated Set 2 keyboard** and **ThinkPad Extra Buttons**. So, the **AT Translated Set 2 keyboard** works like a typical keyboard, and the **ThinkPad Extra Buttons** represents functions that are available in combination with the `Fn` button, such as sleep mode (`Fn + 4`) and screenshot (`Fn + S`). For a deeper experience, we'll work with the **ThinkPad Extra Buttons** (`/dev/input/event4`).

#### 2. Find the right scancode

After choosing the right input device, we need to find the **scancode** of the button we want to remap. At this stage, by running the `evtest` utility again, selecting the right input device and pressing the button, we will get the report like this:
```
...
Event: time 1726200548.302346, type 4 (EV_MSC), code 4 (MSC_SCAN), value 46
Event: time 1726200548.302346, type 1 (EV_KEY), code 149 (KEY_PROG2), value 1
Event: time 1726200548.302346, -------------- SYN_REPORT ------------
Event: time 1726200548.302363, type 1 (EV_KEY), code 149 (KEY_PROG2), value 0
Event: time 1726200548.302363, -------------- SYN_REPORT ------------
```
Here we need to look at the value of `MSC_SCAN` and as you can see the value of combination `Fn + PrtScr` is `46` and the **scancode** is `KEY_PROG2`. Thus, we found out that the **scancode** `46` of the `PrtScr` button of the **ThinkPad Extra Buttons** is mapped as the `KEY_PROG2` **keycode**.

#### 3. Find the right keycode

So we already know the right **scancode** and the next step is to find the right **keycode**. A better way is to look at the default Linux **keycodes** in the `/usr/include/linux/input-event-codes.h` file. Here we have to look at the `KEY_*` definitions and for our example the variants are: `KEY_MENU`, `KEY_CONTEXT_MENU`, `KEY_ROOT_MENU`, `KEY_MEDIA_TOP_MENU`, `KEY_BRIGHTNESS_MENU`, `KEY_KBD_LCD_MENU1`, `KEY_KBD_LCD_MENU2`, `KEY_KBD_LCD_MENU3`, `KEY_KBD_LCD_MENU4` and `KEY_KBD_LCD_MENU5`. The `KEY_MENU` and `KEY_CONTEXT_MENU` **keycodes** look right ones, but we can be determine which one will work by trying them. In order, not to waste time the right **keycode** for our example is `KEY_MENU`.

#### 4. Get the right device ID

Ok, we know the **scancode** and what the **keycode** to map it to, and the mapping is best done for a specific device, but not for all, so we need to find the ID of the input device. There are two options: the kernel **modalias** and the device name with DMI:
1. We can find **modalias** in the **sysfs** input directory:
```
$ cat /sys/class/input/event4/device/modalias
```
The output for this example will be:
```
input:b0019v17AAp5054e4101-e0,1,4,5,k71,72,73,78,8C,8E,90,93,94,95,98,9C,9E,AB,AD,BE,BF,C2,CA,CB,CD,D4,D8,D9,DA,DF,E0,E1,E3,E4,EC,ED,EE,F0,168,174,176,1D2,1DB,1DC,246,250,27A,ram4,lsfw3,
```
which we can shorten to:
```
input:b0019v17AAp5054e4101*
```
2. For the second option, we can also find in the `/sys/class/input/event4/device/name` and `/sys/devices/virtual/dmi/id/modalias`, but it's better to use `evemu-describe` command from the `evemu` package:
```
# evemu-describe /dev/input/event4
```
From the output, we need to look at the **DMI** and **Input device name** rows and concatenate them into a string:
```
name:ThinkPad Extra Buttons:dmi:bvnLENOVO:bvrN3MET09W(1.06):bd10/11/2022:br1.6:efr1.12:svnLENOVO:pn21AH00B9RA:pvrThinkPadT14Gen3:rvnLENOVO:rn21AH00B9RA:rvrNotDefined:cvnLENOVO:ct10:cvrNone:skuLENOVO_MT_21AH_BU_Think_FM_ThinkPadT14Gen3:
```
which we can shorten to:
```
name:ThinkPad Extra Buttons:dmi:bvn*:bvr*:bd*:svnLENOVO*:pn*:*
```
Before moving to the next step, it's best to check the default **keycode** mapping for availability of the required input device and use that ID, as the option from the default mapping might intefere with your option. The default **keycode** mappings are defined in `/usr/lib/udev/hwdb.d/60-keyboard.hwdb`. And for our example there is a default mapping:
```
...
###########################################################
# Lenovo
###########################################################

# thinkpad_acpi driver
evdev:name:ThinkPad Extra Buttons:dmi:bvn*:bvr*:bd*:svnLENOVO*:pn*:*
 KEYBOARD_KEY_01=screenlock
...
 KEYBOARD_KEY_46=prog2                                  # Fn + PrtSc, on Windows: Snipping tool
...
```
Therefore, it's better to use the `name:ThinkPad Extra Buttons:dmi:bvn*:bvr*:bd*:svnLENOVO*:pn*:*` ID instead of `input:b0019v17AAp5054e4101*`.

#### 5. Create a file for the hardware database

Finally, we can create a file with our mapping. Each map consists of a pair: `KEYBOARD_KEY_` + **scancode** and **keycode** in lowercase:
```
KEYBOARD_KEY_46=menu
```
And with the input ID our file will looks like this:
```
evdev:name:ThinkPad Extra Buttons:dmi:bvn*:bvr*:bd*:svnLENOVO*:pn*:*
 KEYBOARD_KEY_46=menu
```
> Note that before the input ID should be the `evdev:` prefix.
So we can create a file for the hardware database with this command:
```
# echo -e 'evdev:name:ThinkPad Extra Buttons:dmi:bvn*:bvr*:bd*:svnLENOVO*:pn*:*\n KEYBOARD_KEY_46=menu' > /etc/udev/hwdb.d/90-remap.hwdb
```
> Note that the file name doesn't matter, but the number at beggining of the file name must be greater than the number of all `.hwdb` files that have mappings for the input device.
