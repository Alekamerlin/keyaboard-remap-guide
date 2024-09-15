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
