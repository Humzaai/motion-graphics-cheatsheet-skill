Create a ready-to-publish animated motion graphics GIF cheatsheet about: $ARGUMENTS

You are a motion graphics designer and educator. When the user drops a title or topic, produce a complete animated GIF — no clarifying questions unless the topic is genuinely ambiguous. Use your own knowledge to fill in the content. Output is always 1080×1350px (4:5, portrait) — the LinkedIn/Substack optimal format.

---

## BRAND (always use these)
- **Creator:** AI In Public
- **Attribution line:** `AI In Public  ·  aiinpublic.substack.com`
- **Accent color:** `#FF6B35` (coral orange) — used for 1-3 highlight words in the title
- **Logo mark:** 12-spoke Anthropic-style starburst, drawn in accent color
- **Footer:** Dark navy bar (`#1A1A2E`) full width, 60px tall, at bottom of every frame
- **Output folder:** `~/Desktop/Motion Graphics Cheatsheets/`

---

## STEP 1 — Pick the right layout

| Layout | Pick when topic is about… | Animation style |
|--------|--------------------------|-----------------|
| **A. PROGRESSIVE-LIST** | How-to guides, tips, commands, workflows | Right column items reveal one per frame |
| **B. RADIAL-SPOKE** | Stacks, tool ecosystems, collections | Items pop in clockwise around a central logo |
| **C. COLUMN-COMPARE** | Models, platforms, options, A vs B | Columns reveal left to right |
| **D. TIERED-LEVELS** | Beginner / intermediate / advanced, levels | Level bands reveal top to bottom |
| **E. DARK-WORKFLOW** | Agent patterns, system architecture, technical flows | Numbered sections on dark bg, reveal top to bottom |

---

## STEP 2 — Design the content (use your knowledge — never invent fake stats)

Decide:
- `TITLE` — 3-6 words
- `TITLE_ACCENT` — 1-3 words within the title to color orange
- Layout-specific content (items, columns, levels, etc.)

---

## STEP 3 — Write and run the complete Python script

### SHARED BASE CODE (copy this exactly into every script)

```python
from PIL import Image, ImageDraw, ImageFont
import os, math, textwrap

W, H = 1080, 1350
PADDING = 60

# Palette
BG_LIGHT  = "#F7F7F5"
BG_DARK   = "#0E1117"
NAVY      = "#1A1A2E"
ACCENT    = "#FF6B35"
GREEN     = "#2ECC71"
TEXT_DARK = "#1A1A2E"
TEXT_MID  = "#555555"
TEXT_LIGHT= "#999999"
TEXT_WHITE= "#FFFFFF"
DIVIDER   = "#E0E0DC"
TAG_BG    = "#E8F4F0"
TAG_FG    = "#1A6B50"
ATTRIBUTION = "AI In Public  ·  aiinpublic.substack.com"

try:
    F_TITLE = ImageFont.truetype("/System/Library/Fonts/Helvetica.ttc", 54)
    F_HEAD  = ImageFont.truetype("/System/Library/Fonts/Helvetica.ttc", 28)
    F_SUBHD = ImageFont.truetype("/System/Library/Fonts/Helvetica.ttc", 22)
    F_LABEL = ImageFont.truetype("/System/Library/Fonts/Helvetica.ttc", 17)
    F_BODY  = ImageFont.truetype("/Library/Fonts/Arial Unicode.ttf", 17)
    F_ATTR  = ImageFont.truetype("/Library/Fonts/Arial Unicode.ttf", 15)
    F_SMALL = ImageFont.truetype("/Library/Fonts/Arial Unicode.ttf", 13)
    F_NUM   = ImageFont.truetype("/System/Library/Fonts/Helvetica.ttc", 46)
    F_MONO  = ImageFont.truetype("/System/Library/Fonts/Courier.ttc", 14)
except:
    F_TITLE=F_HEAD=F_SUBHD=F_LABEL=F_BODY=F_ATTR=F_SMALL=F_NUM=F_MONO=ImageFont.load_default()

def rr(draw, xy, radius, fill, outline=None, width=1):
    draw.rounded_rectangle(xy, radius=radius, fill=fill, outline=outline, width=width)

def tw(text, font):
    bb = font.getbbox(text); return bb[2] - bb[0]

def th(text, font):
    bb = font.getbbox(text); return bb[3] - bb[1]

def draw_starburst(draw, cx, cy, r_outer=38, r_inner=16, spokes=12, color=ACCENT):
    pts = []
    for i in range(spokes * 2):
        angle = math.radians(i * 180 / spokes - 90)
        r = r_outer if i % 2 == 0 else r_inner
        pts.append((cx + r * math.cos(angle), cy + r * math.sin(angle)))
    draw.polygon(pts, fill=color)

def draw_header(draw, title, accent_words, y=52):
    x = PADDING
    for word in title.split():
        color = ACCENT if word in accent_words.split() else TEXT_DARK
        draw.text((x, y), word + " ", font=F_TITLE, fill=color)
        x += tw(word + " ", F_TITLE)
    ay = y + 66
    draw.text((PADDING, ay), ATTRIBUTION, font=F_ATTR, fill=TEXT_LIGHT)
    div_y = ay + 24
    draw.line([(PADDING, div_y), (W - PADDING, div_y)], fill=DIVIDER, width=1)
    return div_y + 16

def draw_footer(img):
    d = ImageDraw.Draw(img)
    d.rectangle([(0, H - 60), (W, H)], fill=NAVY)
    draw_starburst(d, 52, H - 30, r_outer=12, r_inner=5, color=ACCENT)
    d.text((72, H - 40), ATTRIBUTION.upper(), font=F_ATTR, fill=TEXT_WHITE)

def draw_pill(draw, x, y, label, font=None, bg=TAG_BG, fg=TAG_FG, pad_x=14, h=28, radius=5):
    f = font or F_LABEL
    lw = tw(label, f)
    pw = lw + pad_x * 2
    rr(draw, [x, y, x + pw, y + h], radius, bg)
    draw.text((x + pad_x, y + (h - th("A", f)) // 2), label, font=f, fill=fg)
    return pw

def dotted_line(draw, x0, y0, x1, y1, color="#CCCCCC", dash=6, gap=4, width=1):
    length = math.hypot(x1-x0, y1-y0)
    if length == 0: return
    dx=(x1-x0)/length; dy=(y1-y0)/length
    pos = 0
    while pos < length:
        xe=x0+dx*min(pos+dash,length); ye=y0+dy*min(pos+dash,length)
        draw.line([(x0+dx*pos, y0+dy*pos),(xe,ye)], fill=color, width=width)
        pos += dash + gap

def make_canvas(bg=BG_LIGHT):
    return Image.new("RGB", (W, H), bg)

def save_gif(frames, slug, duration=900):
    out = os.path.expanduser(f"~/Desktop/Motion Graphics Cheatsheets/{slug}.gif")
    os.makedirs(os.path.dirname(out), exist_ok=True)
    all_f = frames + [frames[-1], frames[-1]]
    all_f[0].save(out, save_all=True, append_images=all_f[1:], optimize=False, duration=duration, loop=0)
    size = os.path.getsize(out)
    print(f"Saved: {out}")
    print(f"Frames: {len(all_f)}  |  Size: {size:,} bytes")
    return out
```

---

### LAYOUT A — PROGRESSIVE-LIST (1080×1350)

**Content vars:**
```python
title         = "How to Prompt Claude Code"
title_accent  = "Claude Code"
context_lines = [              # left panel static, 6-9 lines, 34 chars max each
    "Claude Code is an AI coding",
    "agent in your **terminal**.",
    "",
    "It reads your codebase,",
    "runs commands, and ships",
    "features **end-to-end**.",
    "",
    "Start with /plan. Always.",
    "Give it a CLAUDE.md.",
]
# Each item: (label, description, icon_char, icon_color)
# icon_char: single Unicode char rendered in a colored box (↓ ≡ ⊘ / ◉ ★ ⬡ !)
# icon_color: unique hex per item — cycle through palette below
ICON_PALETTE = ["#3498DB","#2ECC71","#E74C3C","#FF6B35","#9B59B6","#F39C12","#1ABC9C","#E67E22"]
items = [
    ("INSTALL",       "npm i -g @anthropic-ai/claude-code",               "↓",  "#3498DB"),
    ("CLAUDE.md",     "Your rulebook. Claude reads it every session.",     "≡",  "#2ECC71"),
    ("MODES",         "Plan → Default → Auto. Trust ladder.",              "⊘",  "#E74C3C"),
    ("SLASH CMDS",    "/plan /review /memory /compact /clear",             "/",  "#FF6B35"),
    ("MEMORY",        "Saves rules and lessons across all sessions.",      "◉",  "#9B59B6"),
    ("SKILLS",        "Custom /commands you build and reuse.",             "★",  "#F39C12"),
    ("MCP SERVERS",   "Connect Gmail, Calendar, Notion and more.",         "⬡",  "#1ABC9C"),
    ("AUTO MODE",     "Ships features end-to-end. Approve nothing.",       "!",  "#E67E22"),
]
slug = "how-to-prompt-claude-code"
```

**Layout constants:**
```python
CONTENT_Y = 162
CTX_X     = PADDING        # left context column
ITEM_H    = 130            # vertical slot per item

SPINE_X   = 492            # x of vertical timeline spine
DOT_R     = 8              # radius of spine dot
ICON_X    = SPINE_X + 28  # icon box left edge
ICON_SIZE = 36             # icon box w & h
PILL_X    = ICON_X + ICON_SIZE + 10   # pill tag left edge
F_ICON    = F_LABEL        # font for icon chars
```

**Key helpers:**
```python
def dot_y(i):
    return CONTENT_Y + 14 + i * ITEM_H + ICON_SIZE // 2

def draw_icon_box(d, x, y, char, color):
    rr(d, [x, y, x+ICON_SIZE, y+ICON_SIZE], 8, color)
    cw = tw(char, F_ICON); ch = th(char, F_ICON)
    d.text((x+(ICON_SIZE-cw)//2, y+(ICON_SIZE-ch)//2 - 1), char, font=F_ICON, fill=TEXT_WHITE)

def draw_pill_tag(d, x, y, label):
    lw = tw(label, F_LABEL); pw = lw + 24
    rr(d, [x, y, x+pw, y+26], 5, TAG_BG)
    d.text((x+12, y+(26-th("A",F_LABEL))//2), label, font=F_LABEL, fill=TAG_FG)
    return pw
```

**Full frame code:**
```python
def make_base():
    img = make_canvas()
    d = ImageDraw.Draw(img)
    draw_header(d, title, title_accent)
    cy = CONTENT_Y + 10
    for line in context_lines:
        parts = line.split("**"); cx = CTX_X
        for i, part in enumerate(parts):
            color = ACCENT if i % 2 == 1 else TEXT_MID
            d.text((cx, cy), part, font=F_BODY, fill=color)
            cx += tw(part, F_BODY)
        cy += 30
    draw_footer(img)
    return img

def make_frame(n):
    img = make_base()
    d = ImageDraw.Draw(img)
    if n == 0:
        return img
    # Growing timeline spine — shadow then spine
    y_top = dot_y(0); y_bot = dot_y(n - 1)
    d.line([(SPINE_X, y_top), (SPINE_X, y_bot)], fill="#D0D0CC", width=4)
    d.line([(SPINE_X, y_top), (SPINE_X, y_bot)], fill="#BEBEBB", width=2)
    # Items
    for i, (label, desc, icon_char, icon_color) in enumerate(items[:n]):
        iy = CONTENT_Y + 14 + i * ITEM_H
        dy = dot_y(i)
        # Dot on spine
        d.ellipse([SPINE_X-DOT_R, dy-DOT_R, SPINE_X+DOT_R, dy+DOT_R],
                  fill=icon_color, outline="#FFFFFF", width=2)
        # Horizontal connector: dot → icon
        d.line([(SPINE_X+DOT_R, dy), (ICON_X, dy)], fill=icon_color, width=2)
        # Icon box
        draw_icon_box(d, ICON_X, iy, icon_char, icon_color)
        # Pill label
        draw_pill_tag(d, PILL_X, iy, label)
        # Description
        wrapped = textwrap.fill(desc, width=34)
        d.multiline_text((PILL_X, iy+32), wrapped, font=F_BODY, fill=TEXT_MID, spacing=4)
    return img

frames = [make_frame(n + 1) for n in range(len(items))]
out = save_gif(frames, slug)
os.system(f'open "{out}"')
```

---

### LAYOUT B — RADIAL-SPOKE (1080×1350)

**Layout constants:**
```python
CENTER_X = 540
CENTER_Y = 720
SPOKE_R  = 340
CARD_W   = 185
CARD_H   = 74
LOGO_R   = 58
```

**Frame code:**
```python
def item_pos(i, n):
    angle = math.radians(i * (360 / n) - 90)
    cx = CENTER_X + SPOKE_R * math.cos(angle)
    cy = CENTER_Y + SPOKE_R * math.sin(angle)
    return cx, cy, angle

def draw_spoke_item(d, i, n, label, icon, desc):
    cx, cy, angle = item_pos(i, n)
    dx = math.cos(angle); dy = math.sin(angle)
    dotted_line(d, CENTER_X + LOGO_R*dx, CENTER_Y + LOGO_R*dy,
                cx - CARD_W//2*0.6*dx, cy - CARD_H//2*0.6*dy)
    xy = [cx-CARD_W//2, cy-CARD_H//2, cx+CARD_W//2, cy+CARD_H//2]
    rr(d, xy, 10, "#FFFFFF", "#E0E0DC", 1)
    d.text((cx - tw(label, F_LABEL)//2, cy - 18), label, font=F_LABEL, fill=TEXT_DARK)
    d.multiline_text((xy[0]+10, cy+6), textwrap.fill(desc, 20), font=F_SMALL, fill=TEXT_LIGHT, align="center", spacing=2)

def make_radial_frame(n):
    img = make_canvas(); d = ImageDraw.Draw(img)
    draw_header(d, title, title_accent)
    draw_starburst(d, CENTER_X, CENTER_Y, LOGO_R, int(LOGO_R/2.4), color=ACCENT)
    for i in range(n):
        draw_spoke_item(d, i, len(items), *items[i])
    draw_footer(img)
    return img

frames = [make_radial_frame(n+1) for n in range(len(items))]
out = save_gif(frames, slug, duration=700)
os.system(f'open "{out}"')
```

---

### LAYOUT C — COLUMN-COMPARE (1080×1350)

**Layout constants (adjust for N columns):**
```python
CONTENT_Y = 162
GAP       = 14
COL_W     = (W - 2*PADDING - GAP*(len(columns)-1)) // len(columns)
COL_X     = [PADDING + i*(COL_W+GAP) for i in range(len(columns))]
HEAD_H    = 60
ROW_H     = (H - 60 - CONTENT_Y - HEAD_H) // max(len(c["rows"]) for c in columns)
```

**Frame code:**
```python
def draw_col_header(d, col, x):
    y = CONTENT_Y
    rr(d, [x, y, x+COL_W, y+HEAD_H], 8, col["color"])
    d.text((x+12, y+10), col["header"], font=F_LABEL, fill=TEXT_WHITE)
    draw_pill(d, x+12, y+34, col["badge"], F_SMALL, "#FFFFFF33", TEXT_WHITE, 8, 18, 4)

def draw_col_rows(d, col, x):
    y = CONTENT_Y + HEAD_H + 6
    for j, (lbl, val) in enumerate(col["rows"]):
        ry = y + j * ROW_H
        bg = "#FFFFFF" if j % 2 == 0 else "#F5F5F3"
        rr(d, [x, ry, x+COL_W, ry+ROW_H-2], 0, bg)
        d.text((x+10, ry+8), lbl, font=F_LABEL, fill=TEXT_DARK)
        d.multiline_text((x+10, ry+28), textwrap.fill(val, 22), font=F_SMALL, fill=TEXT_MID, spacing=2)

# Frame 0 = headers only; Frame N = headers + columns[0..N] rows
def make_col_frame(n_cols_shown):
    img = make_canvas(); d = ImageDraw.Draw(img)
    draw_header(d, title, title_accent)
    for i, col in enumerate(columns):
        draw_col_header(d, col, COL_X[i])
        if i < n_cols_shown:
            draw_col_rows(d, col, COL_X[i])
    draw_footer(img)
    return img

frames = [make_col_frame(0)] + [make_col_frame(n+1) for n in range(len(columns))]
out = save_gif(frames, slug)
os.system(f'open "{out}"')
```

---

### LAYOUT D — TIERED-LEVELS (1080×1350)

**Layout constants:**
```python
CONTENT_Y = 162
LEVEL_H   = 362
LEVEL_GAP = 12
BW        = 7
CX        = PADDING + BW + 20
SNIPPET_X = W - PADDING - 270
SNIPPET_W = 250
```

**Frame code:**
```python
def draw_level(d, lv, idx):
    ly = CONTENT_Y + idx * (LEVEL_H + LEVEL_GAP)
    rr(d, [PADDING, ly, W-PADDING, ly+LEVEL_H], 8, "#FAFAFA", "#EBEBEB", 1)
    d.rectangle([PADDING, ly, PADDING+BW, ly+LEVEL_H], fill=lv["color"])
    d.text((CX, ly+14), lv["number"], font=F_NUM, fill=lv["color"])
    nw = tw(lv["number"], F_NUM)
    d.text((CX+nw+12, ly+20), lv["title"], font=F_HEAD, fill=TEXT_DARK)
    rows = [("HOW TO USE", lv["how"]), ("WHO CONTROLS", lv["controls"]), ("WHAT IT DOES", lv["does"])]
    for j, (lbl, val) in enumerate(rows):
        ry = ly + 86 + j * 84
        d.text((CX, ry), lbl, font=F_SMALL, fill=lv["color"])
        d.multiline_text((CX, ry+18), textwrap.fill(val, 38), font=F_BODY, fill=TEXT_MID, spacing=3)
    rr(d, [SNIPPET_X, ly+14, SNIPPET_X+SNIPPET_W, ly+LEVEL_H-14], 8, NAVY)
    d.multiline_text((SNIPPET_X+16, ly+30), textwrap.fill(lv["how"], 26), font=F_MONO, fill=GREEN, spacing=5)

frames = []
for n in range(1, len(levels)+1):
    img = make_canvas(); d = ImageDraw.Draw(img)
    draw_header(d, title, title_accent)
    for i in range(n): draw_level(d, levels[i], i)
    draw_footer(img)
    frames.append(img)

out = save_gif(frames, slug)
os.system(f'open "{out}"')
```

---

### LAYOUT E — DARK-WORKFLOW (1080×1350)

Identical structure to LAYOUT D with these overrides:
- `make_canvas(bg=BG_DARK)`
- All text: `TEXT_WHITE` or `"#AAAAAA"`
- Level band bg: `"#161B22"`, border color: use level color
- Code box bg: `"#1E2A3A"`, text: `GREEN`
- Footer: `d.rectangle([(0, H-60),(W,H)], fill="#000000")`
- Divider line: `"#333333"`
- Attribution text: `"#666666"`

---

## STEP 4 — Deliver

1. Run the full Python script with Bash
2. `open` the output GIF
3. Print: layout chosen, frame count, file path, file size
