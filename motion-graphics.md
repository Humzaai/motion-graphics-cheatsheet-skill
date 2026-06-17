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

LOGO_DIR = "/tmp/logos"   # pre-downloaded by download_logos() below

def paste_logo(img, logo_key, x, y, size, radius=10):
    """Paste a real PNG brand logo (from LOGO_DIR) onto img with rounded corners.
    logo_key is the filename without .png (e.g. 'anthropic', 'github', 'notion').
    Returns True if logo was placed, False if file missing (caller draws fallback)."""
    path = f"{LOGO_DIR}/{logo_key}.png"
    if not os.path.exists(path):
        return False
    try:
        import urllib.request
        from io import BytesIO
        logo = Image.open(path).convert("RGBA").resize((size, size), Image.LANCZOS)
        bg = Image.new("RGBA", (size, size), "white")
        bg.paste(logo, (0, 0), logo)
        bg = bg.convert("RGB")
        mask = Image.new("L", (size, size), 0)
        ImageDraw.Draw(mask).rounded_rectangle([0, 0, size, size], radius=radius, fill=255)
        img.paste(bg, (int(x), int(y)), mask)
        return True
    except Exception:
        return False

def download_logos(logo_map):
    """Download brand logos to LOGO_DIR. logo_map = {key: domain}.
    Uses Clearbit first, Google favicon fallback. Call once at script top.
    Example: download_logos({'anthropic':'anthropic.com','github':'github.com'})"""
    import urllib.request
    from io import BytesIO
    os.makedirs(LOGO_DIR, exist_ok=True)
    headers = {"User-Agent": "Mozilla/5.0"}
    for name, domain in logo_map.items():
        path = f"{LOGO_DIR}/{name}.png"
        if os.path.exists(path):
            continue
        for url in [
            f"https://logo.clearbit.com/{domain}?size=128",
            f"https://www.google.com/s2/favicons?domain={domain}&sz=128",
        ]:
            try:
                req = urllib.request.Request(url, headers=headers)
                with urllib.request.urlopen(req, timeout=8) as r:
                    data = r.read()
                img = Image.open(BytesIO(data)).convert("RGBA").resize((128, 128), Image.LANCZOS)
                img.save(path)
                break
            except Exception:
                continue

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

**Animation pattern:** ALL card slot outlines visible from frame 1 (empty placeholders). Each frame fills in one more card. This way the canvas never looks empty.

**Content vars:**
```python
title        = "Claude Code 101"
title_accent = "101"
# icon_kind options: "download" "document" "shield" "slash" "brain" "star" "grid" "bulb"
# "arrow_right" "lock" "check" "plug" "eye" "code" "people" "chart"
# icon_color: unique hex per item — 8-color palette below
ICON_PALETTE = ["#3498DB","#27AE60","#E74C3C","#FF6B35","#8E44AD","#F39C12","#16A085","#D35400"]
items = [
    ("INSTALL",         "npm i -g @anthropic-ai/claude-code",               "download","#3498DB"),
    ("CLAUDE.md",       "Your rulebook. Claude reads it every session.",     "document","#27AE60"),
    ("PERMISSION MODE", "Plan = safe.  Default = balanced.  Auto = trust.", "shield",  "#E74C3C"),
    ("SLASH COMMANDS",  "/plan  /review  /memory  /compact  /clear",        "slash",   "#FF6B35"),
    ("MEMORY",          "Saves rules and lessons across all sessions.",      "brain",   "#8E44AD"),
    ("SKILLS",          "Build custom /commands — your own reusable flows.", "star",    "#F39C12"),
    ("MCP SERVERS",     "Connect Gmail, Calendar, Notion, GitHub + more.",   "grid",    "#16A085"),
    ("PRO TIP",         "Drop files, not folders. Keep CLAUDE.md tight.",    "bulb",    "#D35400"),
]
slug = "claude-code-101"
```

**Layout constants (1080×1350, 8 items):**
```python
CONTENT_Y = 162
N         = len(items)
CARD_W    = W - 2*PADDING           # 960
CARD_GAP  = 10
CARD_H    = (H - 64 - CONTENT_Y - CARD_GAP*(N-1)) // N   # ~131px

BAR_W     = 7                        # left color bar width
ICON_SIZE = 48
ICON_X    = PADDING + BAR_W + 16    # 83
LABEL_X   = ICON_X + ICON_SIZE + 20 # 151
SPINE_X   = PADDING - 18            # 42 (timeline in left margin)
DOT_R     = 8
```

**Icon shapes (drawn — no font glyphs needed):**
```python
def draw_icon(d, x, y, kind, color, size=48):
    cx=x+size//2; cy=y+size//2; w=TEXT_WHITE
    if kind=="download":
        d.line([(cx,y+10),(cx,cy+4)],fill=w,width=4)
        d.polygon([(cx-9,cy+4),(cx+9,cy+4),(cx,y+size-10)],fill=w)
    elif kind=="document":
        for lx in [cy-10,cy,cy+10]: d.line([(x+12,lx),(x+size-12,lx)],fill=w,width=3)
    elif kind=="shield":
        d.ellipse([x+10,y+10,x+size-10,y+size-10],outline=w,width=3)
        d.line([(x+14,y+14),(x+size-14,y+size-14)],fill=w,width=3)
    elif kind=="slash":
        d.line([(x+size-14,y+10),(x+14,y+size-10)],fill=w,width=5)
    elif kind=="brain":
        d.ellipse([x+8,y+8,x+size-8,y+size-8],outline=w,width=3)
        d.ellipse([x+17,y+17,x+size-17,y+size-17],outline=w,width=2)
        d.ellipse([x+size//2-4,y+size//2-4,x+size//2+4,y+size//2+4],fill=w)
    elif kind=="star":
        pts=[]
        for i in range(10):
            a=math.radians(i*36-90); r=18 if i%2==0 else 9
            pts.append((cx+r*math.cos(a),cy+r*math.sin(a)))
        d.polygon(pts,fill=w)
    elif kind=="grid":
        for gx,gy in [(x+10,y+10),(x+27,y+10),(x+10,y+27),(x+27,y+27)]: rr(d,[gx,gy,gx+12,gy+12],3,w)
    elif kind=="bulb":
        d.ellipse([cx-5,y+8,cx+5,y+18],fill=w)
        d.rectangle([cx-3,y+22,cx+3,y+size-10],fill=w)
        d.ellipse([cx-3,y+size-12,cx+3,y+size-6],fill=w)
    elif kind=="arrow_right":
        d.line([(x+10,cy),(x+size-14,cy)],fill=w,width=4)
        d.polygon([(x+size-14,cy-8),(x+size-14,cy+8),(x+size-8,cy)],fill=w)
    elif kind=="check":
        d.line([(x+10,cy),(cx-4,y+size-10),(x+size-10,y+10)],fill=w,width=4)
    elif kind=="lock":
        d.rectangle([x+12,cy-2,x+size-12,y+size-10],fill=w)
        d.arc([x+16,y+10,x+size-16,cy+4],180,0,fill=w,width=3)
    elif kind=="code":
        d.line([(x+14,cy-8),(x+8,cy),(x+14,cy+8)],fill=w,width=3)
        d.line([(x+size-14,cy-8),(x+size-8,cy),(x+size-14,cy+8)],fill=w,width=3)
    elif kind=="people":
        d.ellipse([cx-14,y+8,cx-2,y+20],fill=w)
        d.arc([cx-18,y+20,cx+2,y+size-8],180,0,fill=w,width=3)
        d.ellipse([cx+2,y+10,cx+14,y+22],fill=w)
        d.arc([cx-2,y+22,cx+18,y+size-6],180,0,fill=w,width=3)
    elif kind=="chart":
        for bx,bh in [(x+10,size-16),(x+22,size//2),(x+34,size-24)]:
            d.rectangle([bx,y+size-bh-8,bx+8,y+size-8],fill=w)
```

**Full frame code:**
```python
def card_y(i): return CONTENT_Y + i*(CARD_H+CARD_GAP)
def dot_cy(i): return card_y(i) + CARD_H//2

def hex_rgb(h): h=h.lstrip('#'); return tuple(int(h[i:i+2],16) for i in (0,2,4))
def blend(c,f=0.84,over="#F4F4F1"):
    r,g,b=hex_rgb(c); br,bg_,bb=hex_rgb(over)
    return "#{:02X}{:02X}{:02X}".format(int(r*(1-f)+br*f),int(g*(1-f)+bg_*f),int(b*(1-f)+bb*f))

def make_frame(n_filled):
    img=Image.new("RGB",(W,H),"#F4F4F1")
    d=ImageDraw.Draw(img)
    draw_header(d, title, title_accent)
    draw_footer(img); d=ImageDraw.Draw(img)

    _nw=tw("08",F_NUM)   # width of largest number
    DESC_MAX_W=PADDING+CARD_W-LABEL_X-_nw-28

    for i,(label,desc,icon_kind,color) in enumerate(items):
        cy=card_y(i)
        if i < n_filled:
            # Filled card
            rr(d,[PADDING,cy,PADDING+CARD_W,cy+CARD_H],10,"#FFFFFF","#E0E0DA",1)
            d.rectangle([PADDING,cy,PADDING+BAR_W,cy+CARD_H],fill=color)
            rr(d,[PADDING,cy,PADDING+BAR_W+4,cy+CARD_H],0,color)
            # Faint background number
            ns=f"{i+1:02d}"; nw=tw(ns,F_NUM); nh=th(ns,F_NUM)
            d.text((PADDING+CARD_W-nw-16,cy+(CARD_H-nh)//2),ns,font=F_NUM,fill=blend(color,0.78,"#FFFFFF"))
            # Icon box
            icon_y=cy+(CARD_H-ICON_SIZE)//2
            rr(d,[ICON_X,icon_y,ICON_X+ICON_SIZE,icon_y+ICON_SIZE],10,color)
            draw_icon(d,ICON_X,icon_y,icon_kind,color)
            # Connector line icon → label
            d.line([(ICON_X+ICON_SIZE+2,cy+CARD_H//2),(LABEL_X-4,cy+CARD_H//2)],
                   fill=blend(color,0.55,"#FFFFFF"),width=2)
            # Pill label (upper half)
            pill_y=cy+CARD_H//2-31; lw=tw(label,F_LABEL); pw=lw+26; ph=28
            rr(d,[LABEL_X,pill_y,LABEL_X+pw,pill_y+ph],6,blend(color,0.84,"#FFFFFF"))
            d.text((LABEL_X+13,pill_y+(ph-th("A",F_LABEL))//2),label,font=F_LABEL,fill=color)
            # Description (pixel-accurate wrapping)
            words=desc.split(); line=""; lines_out=[]
            for word in words:
                test=line+" "+word if line else word
                if tw(test,F_BODY)<=DESC_MAX_W: line=test
                else: lines_out.append(line); line=word
            if line: lines_out.append(line)
            for li,ln in enumerate(lines_out[:2]):
                d.text((LABEL_X,cy+CARD_H//2+4+li*22),ln,font=F_BODY,fill=TEXT_MID)
        else:
            # Empty placeholder
            rr(d,[PADDING,cy,PADDING+CARD_W,cy+CARD_H],10,"#FAFAF8","#E4E4E0",1)
            d.rectangle([PADDING,cy,PADDING+BAR_W,cy+CARD_H],fill="#D4D4D0")

    # Growing timeline spine
    if n_filled>0:
        yt=dot_cy(0); yb=dot_cy(n_filled-1)
        d.line([(SPINE_X,yt),(SPINE_X,yb)],fill="#C4C4C0",width=3)
        for i in range(n_filled):
            color=items[i][3]; dcy=dot_cy(i)
            d.ellipse([SPINE_X-DOT_R,dcy-DOT_R,SPINE_X+DOT_R,dcy+DOT_R],fill=color,outline="#FFFFFF",width=2)
            d.line([(SPINE_X+DOT_R,dcy),(PADDING,dcy)],fill=color,width=2)
    return img

frames=[make_frame(n+1) for n in range(N)]
out=save_gif(frames, slug, duration=850)
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
