# Code Format Guidelines

Formatting conventions for this ZMK user-config repo. Goal: make `config/corne.keymap`
easy to **read visually** — you should be able to look at a layer and immediately see
what every physical key does, without mentally tokenizing devicetree.

These rules are already applied to `config/corne.keymap`.

---

## 1. Keymap bindings — align per layer

A keymap binding row is a sequence of **cells**. A cell is one behavior plus its
arguments, e.g. `&kp Q`, `&none`, `&hm LCTRL A`, `&hltk 4 SPACE`. Corne rows are:

- **Main rows**: 12 cells = 6 left half + 6 right half
- **Thumb row**: 6 cells = 3 left + 3 right

### Rules

- **Align within a layer, not across layers.** Column widths are computed per layer
  (max cell width in that column, within that layer). Cross-layer alignment would pad
  every row to the widest token in the whole file (e.g. `&hm LCTRL A`) and bloat lines
  for no benefit.
- **Preserve the split.** Keep a fixed wider gap (4 spaces) between the left and right
  halves (after cell 6 of main rows, after cell 3 of the thumb row) so the two halves
  read as separate clusters.
- **One blank line** separates the three main rows from the thumb row.
- **Never change token order or content** when formatting — only whitespace. Alignment
  is purely cosmetic; devicetree is whitespace-insensitive between tokens.

### Example

```
&none &kp Q &kp W &kp E &kp R &kp T    &kp Y &kp U &kp I &sk LALT   &kp BSPC  &none
&none &kp A &kp S &kp D &kp F &kp G    &kp H &kp N &kp K &kp O      &kp L     &none
&none &kp Z &kp X &kp C &kp V &kp B    &kp J &kp M &kp P &sk LCTRL  &kp CAPS  &none

&kp LA(SPACE) &mo 3 &hltk 4 SPACE      &kp LSHFT &mo 2 &kp LG(SPACE)
```

---

## 2. Visual layout comment — one per layer

Directly after a layer's `display-name`, place a `/* ... */` block containing a
**unicode box diagram** of the physical key grid. It mirrors the binding layout
(3×12 main grid split in the middle + 1×6 thumb cluster) and shows a short human
label per key.

### Why

- Acts as a **legend** — you read the picture, not the tokens.
- Acts as a **correctness check** — e.g. it makes `&mo 3 → Num` vs `&hltk 4 → Nav`
  obvious at a glance, which is easy to get wrong from the raw bindings.

### Style

- Use unicode box-drawing characters: `┌ ─ ┬ ┐ │ ├ ┼ ┤ └ ┴ ┘`.
- Left and right halves drawn as two separate 6-wide boxes with a 3-space gap.
- Thumb cluster drawn as two 3-wide boxes, centered under the main diagram.
- Inner cell width is per-layer (min 3, max 6), sized to the longest label in that layer.
- Indent the whole comment to align with `display-name` / `bindings` (6 spaces).

### Label derivation

| Binding | Label | Example |
|---|---|---|
| `&none` | *(blank)* | |
| `&kp X` / `&sk X` | glyph/name of `X` | `&kp BSPC` → `Bsp`, `&kp LEFT` → `←` |
| `&hm MOD KEY` | tap key `KEY` | `&hm LCTRL A` → `A` |
| `&mo N` / `&to N` / `&tog N` | layer short name | `&mo 3` → `Num` |
| `&hltk L KEY` | layer short + `␣` | `&hltk 4 SPACE` → `Nav␣` |
| `&bt BT_SEL n` | `BTn` | `&bt BT_SEL 2` → `BT2` |
| `&bt BT_CLR` | `BTclr` | |

**Layer short names:** `0 Base · 1 Cmk · 2 Sym · 3 Num · 4 Nav · 5 Func · 6 BT`

**Keycode glyphs** (subset): `SPACE → ␣`, `BSPC → Bsp`, `RET → Ent`, `TAB → Tab`,
`ESC → Esc`, `LALT/RALT → Alt`, `LCTRL/RCTRL → Ctl`, `LSHFT/RSHFT → Sft`,
`LGUI/RGUI → Cmd`, `LEFT/DOWN/UP/RIGHT → ← ↓ ↑ →`, `N0–N9 → 0–9`, symbols by glyph
(`TILDE → ~`, `AT → @`, …). Composites: `LA(SPACE) → Alt␣`, `LG(SPACE) → Cmd␣`.

### Example

```
/*
 * ┌──────┬──────┬──────┬──────┬──────┬──────┐   ┌──────┬──────┬──────┬──────┬──────┬──────┐
 * │      │  Q   │  W   │  E   │  R   │  T   │   │  Y   │  U   │  I   │ Alt  │ Bsp  │      │
 * ├──────┼──────┼──────┼──────┼──────┼──────┤   ├──────┼──────┼──────┼──────┼──────┼──────┤
 * │      │  A   │  S   │  D   │  F   │  G   │   │  H   │  N   │  K   │  O   │  L   │      │
 * ├──────┼──────┼──────┼──────┼──────┼──────┤   ├──────┼──────┼──────┼──────┼──────┼──────┤
 * │      │  Z   │  X   │  C   │  V   │  B   │   │  J   │  M   │  P   │ Ctl  │ Caps │      │
 * └──────┴──────┴──────┴──────┴──────┴──────┘   └──────┴──────┴──────┴──────┴──────┴──────┘
 *                      ┌──────┬──────┬──────┐   ┌──────┬──────┬──────┐
 *                      │ Alt␣ │ Num  │ Nav␣ │   │ Sft  │ Sym  │ Cmd␣ │
 *                      └──────┴──────┴──────┘   └──────┴──────┴──────┘
 */
```

---

## 3. `behaviors` and `combos` sections

- **Combos** are sorted by first `key-positions` value (ascending) so they scan in
  physical-key order: `tab(2) → esc(14) → lgui(15) → rgui(19) → backspace(20) → enter(32)`.
- Indentation: 2 spaces for the section node, 4 for each named block, 6 for properties.
- Keep one property per line; `timeout-ms`, `key-positions`, `layers`, `bindings` in
  that order within each combo.

---

## 4. `corne.conf`

- Group directives by feature with a `# --- Section ---` heading.
- One `CONFIG_…` per line, sorted alphabetically within a group.
- Commented-out options stay in place as documented toggles (e.g. RGB underglow).

---

## 5. House rules (all files)

- **Indent**: 2 spaces per level for devicetree structure.
- **No trailing whitespace**; files end with a single newline.
- **Comments**: `/* ... */` for diagrams/multi-line, `//` for short inline notes.
- **Keep the MIT/ZMK copyright header** at the top of `corne.keymap`.
- **YAML** (`build.yaml`, `west.yml`, `module.yml`): 2-space indent, one entry per line.

---

## 6. Alignment is generated, not hand-edited

Per-layer alignment + the visual comments are mechanical. Reformatting by hand is
error-prone, so the transformation is scripted (parse cells → compute column widths →
re-emit, with a token-equality check that guarantees only whitespace changed). When
editing a keymap, update the bindings and regenerate the formatting rather than
aligning columns by eye.
