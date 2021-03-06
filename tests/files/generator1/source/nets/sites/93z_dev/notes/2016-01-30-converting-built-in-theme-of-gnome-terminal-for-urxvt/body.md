Recently I’ve been working on a tool that generates simple
reports with ANSI escape codes (mainly with
[color-related SGR parameters](https://en.wikipedia.org/wiki/ANSI_escape_code#Colors)),
and I noticed that current color theme of my terminal emulator is too pale
to work comfortably, so I decided it’s a good time to change theme to more
contrast one—Tango, one of built-in color themes of GNOME Terminal 3.6.2.

Color values for built-in color theme can be obtained from two paths:

1. `~/.gconf/apps/gnome-terminal/profiles/Profile0/%gconf.xml` (config)
2. `gnome-terminal-3.6.2/src/terminal-profile.c` (source code)

First approach (with `%gconf.xml`) involves taking contents of
`stringvalue` element (which is child of `entry` element
with `name="palette"` attribute), splitting it by colon, converting
values like `#EEEEEEEEECEC` to ones urxvt understands. This requires one
to have GNOME Terminal installed somewhere to select color theme and get config file.
As I don’t have that terminal emulator installed (because I don’t use it), I chose second
approach (taking values from source code).

First I downloaded, verified (against SHA-256 checksum) and extracted
`gnome-terminal-3.6.2.tar.xz`: that resulted in `gnome-terminal-3.6.2/`
directory. Quick search revealed that there is `terminal_palettes` array of
arrays of `GdkColor` on line 202 of `gnome-terminal-3.6.2/src/terminal-profile.c`.
Here’s Tango palette:

```c
{ 0, 0x0000, 0x0000, 0x0000 },
{ 0, 0xcccc, 0x0000, 0x0000 },
{ 0, 0x4e4e, 0x9a9a, 0x0606 },
{ 0, 0xc4c4, 0xa0a0, 0x0000 },
{ 0, 0x3434, 0x6565, 0xa4a4 },
{ 0, 0x7575, 0x5050, 0x7b7b },
{ 0, 0x0606, 0x9820, 0x9a9a },
{ 0, 0xd3d3, 0xd7d7, 0xcfcf },
{ 0, 0x5555, 0x5757, 0x5353 },
{ 0, 0xefef, 0x2929, 0x2929 },
{ 0, 0x8a8a, 0xe2e2, 0x3434 },
{ 0, 0xfcfc, 0xe9e9, 0x4f4f },
{ 0, 0x7272, 0x9f9f, 0xcfcf },
{ 0, 0xadad, 0x7f7f, 0xa8a8 },
{ 0, 0x3434, 0xe2e2, 0xe2e2 },
{ 0, 0xeeee, 0xeeee, 0xecec }
```

For purposes of GNOME Terminal these “triples” are converted to `#rrrrggggbbbb`
via following (`pango-1.39.0/pango/pango-color.c`):

```c
g_strdup_printf ("#%04x%04x%04x", color->red, color->green, color->blue);
```

According to urxvt manual (`man urxvt`), it allows user to specify
colors as `rgba:rrrr/gggg/bbbb/aaaa` (this is what lines like
`{ 0, 0x0000, 0x0000, 0x0000 }` must be converted into):

> For complete control, rxvt-unicode also supports "rgba:rrrr/gggg/bbbb/aaaa"
> (exactly four hex digits/component) colour specifications, where the additional
> "aaaa" component specifies opacity (alpha) values. The minimum value of 0000 is
> completely transparent, while "ffff" is completely opaque).

So I wrote simple Python 3 script:

```python3
PALETTE_SOURCE = """
{ 0, 0x0000, 0x0000, 0x0000 },
{ 0, 0xcccc, 0x0000, 0x0000 },
{ 0, 0x4e4e, 0x9a9a, 0x0606 },
{ 0, 0xc4c4, 0xa0a0, 0x0000 },
{ 0, 0x3434, 0x6565, 0xa4a4 },
{ 0, 0x7575, 0x5050, 0x7b7b },
{ 0, 0x0606, 0x9820, 0x9a9a },
{ 0, 0xd3d3, 0xd7d7, 0xcfcf },
{ 0, 0x5555, 0x5757, 0x5353 },
{ 0, 0xefef, 0x2929, 0x2929 },
{ 0, 0x8a8a, 0xe2e2, 0x3434 },
{ 0, 0xfcfc, 0xe9e9, 0x4f4f },
{ 0, 0x7272, 0x9f9f, 0xcfcf },
{ 0, 0xadad, 0x7f7f, 0xa8a8 },
{ 0, 0x3434, 0xe2e2, 0xe2e2 },
{ 0, 0xeeee, 0xeeee, 0xecec }
""".strip()
OUTPUT_TEMPLATE = "urxvt*color{:d}: rgba:{:04x}/{:04x}/{:04x}/ffff"


for n, palette_line in enumerate(PALETTE_SOURCE.split(",\n")):
    line = palette_line.replace("{ 0, ", "").replace(" }", "")
    triple = [int(s, base=16) for s in line.split(", ")]
    print(OUTPUT_TEMPLATE.format(n, *triple))
```

Here’s the output (can be pasted into `.Xdefaults`):

```
urxvt*color0: rgba:0000/0000/0000/ffff
urxvt*color1: rgba:cccc/0000/0000/ffff
urxvt*color2: rgba:4e4e/9a9a/0606/ffff
urxvt*color3: rgba:c4c4/a0a0/0000/ffff
urxvt*color4: rgba:3434/6565/a4a4/ffff
urxvt*color5: rgba:7575/5050/7b7b/ffff
urxvt*color6: rgba:0606/9820/9a9a/ffff
urxvt*color7: rgba:d3d3/d7d7/cfcf/ffff
urxvt*color8: rgba:5555/5757/5353/ffff
urxvt*color9: rgba:efef/2929/2929/ffff
urxvt*color10: rgba:8a8a/e2e2/3434/ffff
urxvt*color11: rgba:fcfc/e9e9/4f4f/ffff
urxvt*color12: rgba:7272/9f9f/cfcf/ffff
urxvt*color13: rgba:adad/7f7f/a8a8/ffff
urxvt*color14: rgba:3434/e2e2/e2e2/ffff
urxvt*color15: rgba:eeee/eeee/ecec/ffff
```
