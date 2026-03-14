# OFL – Open Fixture Language

A custom file format and Python library for describing DMX lighting fixtures.
OFL is designed to be human-readable, easy to write, and fast to parse.

---

## What is OFL?

OFL is a domain-specific language for defining DMX fixtures – moving heads, LED pars, strobes, and more.
Instead of verbose YAML or JSON, OFL uses a clean, indentation-based syntax inspired by Python and KV.

Each fixture lives in its own `.ofl` file. Shared channel definitions can be referenced with `@includes`.

---

## Example

```
fixture Robin600E:
    manufacturer: Robe
    type: moving_head
    weight: 19.5kg
    power: 480W

    @shared/pan_tilt_16bit

    channel dimmer:
        attribute: intensity
        resolution: 16bit
        cap 0..255: open

    channel gobo:
        attribute: gobo
        resolution: 8bit
        cap 0..9:   open
        cap 10..19: gobo(slot=1)
        cap 20..29: gobo(slot=1, spin=cw)

    channel color:
        attribute: color
        resolution: 8bit
        cap 0..9:   open
        cap 10..19: color(hex=#FF0000, name=Red)
        cap 20..29: color(hex=#0000FF, name=Blue)

    mode standard(16ch):
        1,2: pan
        3,4: tilt
        5,6: dimmer
        7:   gobo
        8:   color
```

---

## Syntax

### Fixture Block
Every file starts with a `fixture` block:
```
fixture Name:
    manufacturer: ...
    type: moving_head | led_par | strobe | conventional | led_bar | smoke
    weight: 19.5kg
    power: 480W
```

### Channels
```
channel name:
    attribute: intensity | pan | tilt | color | gobo | strobe | zoom | focus | iris | prism | frost
    resolution: 8bit | 16bit
    cap 0..255: open
```

### Capabilities
```
cap 0..9:   open
cap 10..19: gobo(slot=1)
cap 10..19: color(hex=#FF0000, name=Red)
cap 10..200: strobe(hz_start=1, hz_end=30)
```

### Modes
```
mode standard(16ch):
    1,2: pan
    3,4: tilt
    5:   dimmer
```
Two slots on one line (`1,2: pan`) means a 16bit channel – coarse and fine.

### Includes
```
@pan_tilt_16bit          # searches configured fixture_root
@./my_custom_channel     # relative to current file
@/absolute/path/channel  # absolute path
```
Local channel definitions override included ones.

---

## Project Structure

```
ofl/
├── lexer.py        # Tokenizer – text → token list
├── parser.py       # Parser – token list → AST
├── resolver.py     # Resolver – resolves @ includes, merges channels
├── validator.py    # Validator – semantic checks
├── builder.py      # Builder – AST → Fixture dataclasses
├── writer.py       # Writer – Fixture dataclasses → .ofl text
├── ofl_nodes.py    # AST node dataclasses (internal)
└── ofl_types.py    # Fixture dataclasses (public API)
```

---

## Pipeline

```
.ofl file
    → Lexer    → token list
    → Parser   → AST (FixtureNode)
    → Resolver → resolved AST (@ includes merged)
    → Validator→ semantic validation
    → Builder  → Fixture dataclass
    → (Writer) → .ofl text
```

---

## Usage

```python
from pathlib import Path
from lexer import Lexer
from parser import Parser
from resolver import Resolver
from validator import Validator
from builder import Builder
from writer import Writer

# Load and parse
source = Path("my_fixture.ofl").read_text(encoding="utf-8")
tokens = Lexer(source).tokenize()
ast = Parser(tokens).parse()
resolved = Resolver(Path("fixtures/")).resolve(ast, Path("my_fixture.ofl"))
Validator().validate(resolved)
fixture = Builder.build(resolved)

# Access data
print(fixture.name)
print(fixture.channels)
print(fixture.modes)

# Write back to .ofl
text = Writer.write(fixture)
Path("output.ofl").write_text(text, encoding="utf-8")
```

---

## Valid Attributes

| Attribute   | Description          |
|-------------|----------------------|
| `intensity` | Dimmer / brightness  |
| `pan`       | Horizontal movement  |
| `tilt`      | Vertical movement    |
| `color`     | Color wheel / mixing |
| `gobo`      | Gobo wheel           |
| `strobe`    | Strobe effect        |
| `zoom`      | Beam zoom            |
| `focus`     | Beam focus           |
| `iris`      | Iris / beam size     |
| `prism`     | Prism effect         |
| `frost`     | Frost diffusion      |

---

## Valid Fixture Types

`moving_head`, `led_par`, `strobe`, `conventional`, `led_bar`, `smoke`

---

## Requirements

- Python 3.10+
- No external dependencies

---

## License

MIT