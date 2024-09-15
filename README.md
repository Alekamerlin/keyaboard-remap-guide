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
