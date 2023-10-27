So I tested the following terminals:

- xterm
- Linux console
- Konsole (KDE)
- gnome-terminal (VTE)
- urxvt
- OS/X's standard terminal

And all of them interpret Ctrl+Space as NUL character.

After long digging I _finally_ found where I was first reading about it.

```
he NULL character (code 0) is represented by Ctrl-@, "@" being the code immediately
before "A" in the ASCII character set.
For convenience, a lot of terminals accept Ctrl-Space as an alias for Ctrl-@.
```

(Reference: [Wikipedia](https://en.wikipedia.org/wiki/Control_character#How_control_characters_map_to_keyboards))

In the [VT520 docs](https://vt100.net/docs/vt510-rm/chapter8.html) I sadly didn't find it with crossreading, but in [VT220 docs](http://manx-docs.org/mirror/vt100.net/docs/vt220-rm/table3-5.html) (link found via Chapter 3.2.5) it looks like it is what I was looking for - even though it's not talking about Ctrl+@.

But in case of doubt, I'd go with what most (see list above) terminal emulators do. Mapping Ctrl+Space to emit a NUL byte. :)

EDIT: added OS/X terminal to the list.