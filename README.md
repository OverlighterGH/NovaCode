[make_svg.py](https://github.com/user-attachments/files/34702736/make_svg.py)
"""Генерує анімований nova-code.svg для README.md на GitHub.

Запуск:  python make_svg.py
Результат: nova-code.svg (прозорий фон, літери з'являються по одній, потім заливка #5F2FD4)
"""

NOVA_ART = [
    "███╗   ██╗ ██████╗ ██╗   ██╗ █████╗ ",
    "████╗  ██║██╔═══██╗██║   ██║██╔══██╗",
    "██╔██╗ ██║██║   ██║██║   ██║███████║",
    "██║╚██╗██║██║   ██║╚██╗ ██╔╝██╔══██║",
    "██║ ╚████║╚██████╔╝ ╚████╔╝ ██║  ██║",
    "╚═╝  ╚═══╝ ╚═════╝   ╚═══╝  ╚═╝  ╚═╝",
]
CODE_ART = [
    " ██████╗ ██████╗ ██████╗ ███████╗",
    "██╔════╝██╔═══██╗██╔══██╗██╔════╝",
    "██║     ██║   ██║██║  ██║█████╗  ",
    "██║     ██║   ██║██║  ██║██╔══╝  ",
    "╚██████╗╚██████╔╝██████╔╝███████╗",
    " ╚═════╝ ╚═════╝ ╚═════╝ ╚══════╝",
]

MAIN = "#5F2FD4"        # основний колір
LIGHT = "#7D48ED"       # трохи світліший верх заливки
SHADOW = "#4A26A8"      # колір "тіні" ╔═╗║╚╝ після заливки
SURFACE = "#C3AAFF"     # поверхня "рідини"
DIM_BOX = "#301969"     # контур до заливки
DIM_TXT = "#422194"     # букви/цифри до заливки
DIM_HI = "#7D55EB"      # яскравіші букви/цифри, що мигають

CW, CH = 10, 20         # розмір символу в пікселях (1:2, як у терміналі)
PAD = 12
ALPHA = "NOVACODE0123456789ABCDEFNOVA"

# Часова лінія одного циклу, секунди
CYCLE = 11.5
LETTER_START, LETTER_STEP = 0.3, 0.35
FILL_START, FILL_END = 3.2, 6.4
FADE_START, FADE_END = 10.4, 11.1

W = max(len(l) for l in NOVA_ART + CODE_ART)


def center(lines):
    bw = max(len(l) for l in lines)
    left = (W - bw) // 2
    return [(" " * left + l).ljust(W) for l in lines]


ART = center(NOVA_ART) + [" " * W] + center(CODE_ART)
HC = len(ART)
PX, PY = W * CW, HC * CH

# номер літери для кожної клітинки
LET = [[-1] * W for _ in range(HC)]


def map_letters(row0, left, widths, base):
    for r in range(row0, row0 + 6):
        x = left
        for i, w in enumerate(widths):
            for cx in range(x, x + w):
                LET[r][cx] = base + i
            x += w


map_letters(0, 0, [10, 9, 9, 8], 0)
map_letters(7, (W - 33) // 2, [8, 9, 8, 8], 4)


def n(x):
    s = f"{x:.2f}".rstrip("0").rstrip(".")
    return s if s else "0"


def rect(x, y, w, h):
    return f'<rect x="{n(x)}" y="{n(y)}" width="{n(w)}" height="{n(h)}"/>'


# ---- символи рамки ╔ ╗ ╚ ╝ ═ ║ малюємо самі, щоб лінії ідеально стикувались ----
G, T = CW * 0.25, 1.7


def box_rects(c, x, y):
    cx, cy = CW / 2, CH / 2
    r = []

    def hl(x0, x1, yy):
        r.append(rect(x + x0, y + yy - T / 2, x1 - x0, T))

    def vl(xx, y0, y1):
        r.append(rect(x + xx - T / 2, y + y0, T, y1 - y0))

    if c == "═":
        hl(0, CW, cy - G); hl(0, CW, cy + G)
    elif c == "║":
        vl(cx - G, 0, CH); vl(cx + G, 0, CH)
    elif c == "╔":
        hl(cx - G, CW, cy - G); hl(cx + G, CW, cy + G)
        vl(cx - G, cy - G, CH); vl(cx + G, cy + G, CH)
    elif c == "╗":
        hl(0, cx + G, cy - G); hl(0, cx - G, cy + G)
        vl(cx + G, cy - G, CH); vl(cx - G, cy + G, CH)
    elif c == "╚":
        hl(cx - G, CW, cy + G); hl(cx + G, CW, cy - G)
        vl(cx - G, 0, cy + G); vl(cx + G, 0, cy - G)
    elif c == "╝":
        hl(0, cx + G, cy + G); hl(0, cx - G, cy - G)
        vl(cx + G, 0, cy + G); vl(cx - G, 0, cy - G)
    return r


# ---- суцільні блоки █ (зливаємо сусідні в один прямокутник) та рамка ----
block_rects, box_all = [], []
for cy in range(HC):
    cx = 0
    while cx < W:
        if ART[cy][cx] == "█":
            start = cx
            while cx < W and ART[cy][cx] == "█":
                cx += 1
            block_rects.append(rect(start * CW, cy * CH, (cx - start) * CW + 0.3, CH + 0.3))
        else:
            if ART[cy][cx] != " ":
                box_all += box_rects(ART[cy][cx], cx * CW, cy * CH)
            cx += 1


def anim(attr, values, key_secs, begin=None, repeat=True):
    keys = ";".join(n(k / CYCLE) if k < CYCLE else "1" for k in key_secs)
    vals = ";".join(str(v) for v in values)
    extra = f' begin="{n(begin)}s"' if begin else ""
    rep = ' repeatCount="indefinite"' if repeat else ""
    return (f'<animate attributeName="{attr}" values="{vals}" keyTimes="{keys}" '
            f'dur="{n(CYCLE)}s"{rep}{extra}/>')


# ---- шар 1: порожня форма з букв і цифр, кожна літера з'являється окремо ----
letters_svg = []
for li in range(8):
    ta = LETTER_START + LETTER_STEP * li
    parts = []
    for cy in range(HC):
        for cx in range(W):
            if LET[cy][cx] != li:
                continue
            c = ART[cy][cx]
            if c == " ":
                continue
            x, y = cx * CW, cy * CH
            if c == "█":
                ch = ALPHA[(cx * 31 + cy * 17) % len(ALPHA)]
                hi = (cx * 7 + cy * 3) % 5 == 0
                fill = DIM_HI if hi else DIM_TXT
                blink = ""
                if hi:
                    blink = (f'<animate attributeName="opacity" values="1;.35;1" '
                             f'dur="{n(1.2 + (cx % 4) * 0.25)}s" begin="{n((cx * 0.13 + cy * 0.07) % 1.5)}s" '
                             f'repeatCount="indefinite"/>')
                parts.append(f'<text x="{n(x + CW / 2)}" y="{n(y + CH / 2)}" fill="{fill}">{ch}{blink}</text>')
            else:
                parts.append(f'<g fill="{DIM_BOX}">{"".join(box_rects(c, x, y))}</g>')
    letters_svg.append(
        f'<g opacity="0">{"".join(parts)}'
        + anim("opacity", [0, 0, 1, 1, 0, 0],
               [0, ta, ta + 0.15, FADE_START, FADE_END, CYCLE])
        + "</g>")

# ---- шар 2: заливка, що піднімається знизу вгору ----
fill_y = anim("y", [PY, PY, 0, 0, PY, PY], [0, FILL_START, FILL_END, FADE_END, FADE_END + 0.05, CYCLE])
band_y = anim("y", [PY, PY, -CH, -CH, PY, PY], [0, FILL_START, FILL_END, FADE_END, FADE_END + 0.05, CYCLE])
band_op = anim("opacity", [0, 0, 1, 1, 0, 0],
               [0, FILL_START, FILL_START + 0.05, FILL_END - 0.1, FILL_END + 0.2, CYCLE])
fill_op = anim("opacity", [1, 1, 0, 0], [0, FADE_START, FADE_END, CYCLE])
shine_op = anim("opacity", [0, 0, 1, 1, 0, 0],
                [0, FILL_END, FILL_END + 0.3, FADE_START - 0.3, FADE_START, CYCLE])

BLOCKS = "".join(block_rects)
BOXES = "".join(box_all)

svg = f'''<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="{-PAD} {-PAD} {PX + 2 * PAD} {PY + 2 * PAD}" width="{PX + 2 * PAD}" height="{PY + 2 * PAD}" role="img" aria-label="Nova Code">
<title>Nova Code</title>
<defs>
  <linearGradient id="body" gradientUnits="userSpaceOnUse" x1="0" y1="0" x2="0" y2="{PY}">
    <stop offset="0" stop-color="{LIGHT}"/><stop offset="1" stop-color="{MAIN}"/>
  </linearGradient>
  <linearGradient id="shine" gradientUnits="userSpaceOnUse" x1="0" y1="0" x2="{PX // 3}" y2="0">
    <stop offset="0" stop-color="#fff" stop-opacity="0"/>
    <stop offset=".5" stop-color="#fff" stop-opacity=".45"/>
    <stop offset="1" stop-color="#fff" stop-opacity="0"/>
    <animateTransform attributeName="gradientTransform" type="translate" from="{-PX // 3} 0" to="{PX} 0" dur="2.6s" repeatCount="indefinite"/>
  </linearGradient>
  <clipPath id="rise"><rect x="0" y="{PY}" width="{PX}" height="{PY + CH}">{fill_y}</rect></clipPath>
  <clipPath id="band"><rect x="0" y="{PY}" width="{PX}" height="{CH}">{band_y}</rect></clipPath>
  <g id="blocks" shape-rendering="crispEdges">{BLOCKS}</g>
  <g id="boxes">{BOXES}</g>
</defs>
<g font-family="Consolas, 'Courier New', monospace" font-size="15" font-weight="bold" text-anchor="middle" dominant-baseline="central">
{chr(10).join(letters_svg)}
</g>
<g opacity="1">{fill_op}
  <g clip-path="url(#rise)">
    <use href="#blocks" xlink:href="#blocks" fill="url(#body)"/>
    <use href="#boxes" xlink:href="#boxes" fill="{SHADOW}"/>
  </g>
  <g clip-path="url(#band)" opacity="0">{band_op}
    <use href="#blocks" xlink:href="#blocks" fill="{SURFACE}"/>
    <use href="#boxes" xlink:href="#boxes" fill="{SURFACE}"/>
  </g>
  <g opacity="0">{shine_op}
    <use href="#blocks" xlink:href="#blocks" fill="url(#shine)"/>
  </g>
</g>
</svg>
'''

with open("nova-code.svg", "w", encoding="utf-8") as f:
    f.write(svg)
print(f"nova-code.svg готово ({len(svg) // 1024} КБ)")
