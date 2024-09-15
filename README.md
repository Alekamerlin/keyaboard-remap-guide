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
