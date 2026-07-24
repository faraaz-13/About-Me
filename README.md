import random
import requests
import io
import math
from PIL import Image

USERNAME = "faraaz-13"
DISPLAY_NAME = "Shaik Zaiba Faraaz"
TITLE = "ECE Engineer | AI & VLSI | ISRO Dreamer"
EMAIL = "shaikzaiba1306@gmail.com"

# ─── FILE 1: CONTRIBUTION GRAPH SVG ─────────────────────────────────────────

def make_contribution_svg():
    COLS, ROWS = 53, 7
    CELL = 11
    GAP  = 3
    PAD  = 40
    W    = PAD * 2 + COLS * (CELL + GAP)
    H    = PAD * 2 + ROWS * (CELL + GAP) + 30

    # Contribution colors (GitHub style)
    colors = ["#161b22", "#0e4429", "#006d32", "#26a641", "#39d353"]

    # Generate random-ish contribution data
    random.seed(42)
    grid = []
    for c in range(COLS):
        col = []
        for r in range(ROWS):
            # More activity in recent weeks
            weight = c / COLS
            val = random.choices([0,1,2,3,4],
                weights=[0.4-weight*0.2, 0.25, 0.15, 0.12, 0.08+weight*0.1])[0]
            col.append(val)
        grid.append(col)

    svg = f'''<svg xmlns="http://www.w3.org/2000/svg" width="{W}" height="{H}"
     viewBox="0 0 {W} {H}">
  <defs>
    <filter id="glow" x="-40%" y="-40%" width="180%" height="180%">
      <feGaussianBlur in="SourceGraphic" stdDeviation="2.5" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <filter id="glow2" x="-60%" y="-60%" width="220%" height="220%">
      <feGaussianBlur in="SourceGraphic" stdDeviation="4" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
  </defs>

  <!-- Background -->
  <rect width="{W}" height="{H}" rx="12" fill="#0d1117"/>
  <rect x="1" y="1" width="{W-2}" height="{H-2}" rx="11"
        fill="none" stroke="#30363d" stroke-width="1"/>

  <!-- Title -->
  <text x="{W//2}" y="22" text-anchor="middle"
        font-family="monospace" font-size="11" fill="#8b949e"
        letter-spacing="2">CONTRIBUTION ACTIVITY</text>

  <!-- Month labels -->
'''
    months = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"]
    for i, m in enumerate(months):
        x = PAD + (i * COLS // 12) * (CELL + GAP)
        svg += f'  <text x="{x}" y="{PAD - 8}" font-family="monospace" font-size="9" fill="#8b949e">{m}</text>\n'

    # Day labels
    days = ["","Mon","","Wed","","Fri",""]
    for i, d in enumerate(days):
        if d:
            y = PAD + i * (CELL + GAP) + CELL
            svg += f'  <text x="{PAD - 6}" y="{y}" font-family="monospace" font-size="9" fill="#8b949e" text-anchor="end">{d}</text>\n'

    # Contribution squares with diagonal reveal animation
    total_cells = COLS * ROWS
    for c in range(COLS):
        for r in range(ROWS):
            val = grid[c][r]
            color = colors[val]
            x = PAD + c * (CELL + GAP)
            y = PAD + r * (CELL + GAP)

            # Diagonal delay: bottom-left = 0, top-right = max
            diag = (c + (ROWS - 1 - r)) / (COLS + ROWS - 2)
            delay = diag * 2.5

            use_glow = "glow2" if val >= 4 else ("glow" if val >= 3 else "none")
            filter_attr = f'filter="url(#{use_glow})"' if use_glow != "none" else ""

            cell_id = f"c{c}_{r}"
            svg += f'''
  <g id="{cell_id}">
    <rect x="{x}" y="{y}" width="{CELL}" height="{CELL}" rx="2"
          fill="{color}" {filter_attr}>
      <animate attributeName="opacity"
               values="0;1" dur="0.3s"
               begin="{delay:.2f}s" fill="freeze"/>
      <animate attributeName="transform"
               type="scale" from="0.3" to="1"
               dur="0.3s" begin="{delay:.2f}s" fill="freeze"
               additive="sum"
               transform="translate({x + CELL/2},{y + CELL/2})"/>
    </rect>'''

            # Glint flash for any filled cell
            if val > 0:
                svg += f'''
    <rect x="{x}" y="{y}" width="{CELL}" height="{CELL}" rx="2"
          fill="white" opacity="0">
      <animate attributeName="opacity"
               values="0;0;0.9;0" dur="0.6s"
               begin="{delay + 0.05:.2f}s" fill="freeze"/>
    </rect>'''

            svg += '\n  </g>'

    # Legend
    lx = W - PAD - 5 * (CELL + GAP)
    ly = H - 18
    svg += f'\n  <text x="{lx - 8}" y="{ly + 9}" font-family="monospace" font-size="9" fill="#8b949e">Less</text>'
    for i, col in enumerate(colors):
        gf = 'filter="url(#glow)"' if i >= 3 else ""
        svg += f'\n  <rect x="{lx + i*(CELL+GAP)}" y="{ly}" width="{CELL}" height="{CELL}" rx="2" fill="{col}" {gf}/>'
    svg += f'\n  <text x="{lx + 5*(CELL+GAP) + 4}" y="{ly + 9}" font-family="monospace" font-size="9" fill="#8b949e">More</text>'

    svg += '\n</svg>'
    return svg


# ─── FILE 2: TERMINAL CARD SVG ───────────────────────────────────────────────

def fetch_avatar_ascii(username, width=38, height=18):
    try:
        resp = requests.get(f"https://github.com/{username}.png?size=200", timeout=6)
        img = Image.open(io.BytesIO(resp.content)).convert("L").resize((width, height))
        chars = " .·:;!><~-=+*#%@"
        ascii_rows = []
        for y in range(height):
            row = ""
            for x in range(width):
                px = img.getpixel((x, y))
                row += chars[int(px / 256 * len(chars))]
            ascii_rows.append(row)
        return ascii_rows
    except:
        # Fallback art
        return [
            "  .·:!><~-=+*#%@@@#*+=-~><! ",
            " :!><~-=+*#%@@@@@@@@#*+=-~>< ",
            "·!><~-=+*##%@@@@@@@@@@#*+=-~>",
            "!><~-=+*#%@@@@@@@@@@@@@#*+=- ",
            "><~-=+*#%@@@@@@@@@@@@@@@@#*+ ",
            "~-=+*#%@@@@@  ZAIBA  @@@@@#* ",
            "-=+*#%@@@@@@         @@@@@@# ",
            "=+*#%@@@@@@@  ECE    @@@@@@ ",
            "+*#%@@@@@@@@         @@@@@@  ",
            "*#%@@@@@@@@@  VLSI   @@@@@   ",
            "#%@@@@@@@@@@         @@@@    ",
            "%@@@@@@@@@@@  ISRO   @@@     ",
            "@@@@@@@@@@@@@       @@       ",
            "@@@@@@@@@@@@@@@@@@@@         ",
            " @@@@@@@@@@@@@@@@@           ",
            "  @@@@@@@@@@@@@@             ",
            "   @@@@@@@@@@                ",
            "    @@@@@@@                  ",
        ]

def make_terminal_svg():
    ascii_rows = fetch_avatar_ascii(USERNAME)
    ROWS = len(ascii_rows)
    MAX_W = max(len(r) for r in ascii_rows)
    ascii_rows = [r.ljust(MAX_W) for r in ascii_rows]

    CHAR_W = 7.2
    CHAR_H = 14
    PAD_X  = 16
    PAD_Y  = 44
    BOT    = 44

    W = int(PAD_X * 2 + MAX_W * CHAR_W) + 10
    H = int(PAD_Y + ROWS * CHAR_H + BOT) + 10

    # Time per row reveal
    ROW_DUR   = 0.18
    TOTAL_DUR = ROWS * ROW_DUR + 1.0

    svg = f'''<svg xmlns="http://www.w3.org/2000/svg" width="{W}" height="{H}"
     viewBox="0 0 {W} {H}">
  <defs>
    <filter id="termglow">
      <feGaussianBlur in="SourceGraphic" stdDeviation="1.5" result="b"/>
      <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <filter id="scanglow">
      <feGaussianBlur in="SourceGraphic" stdDeviation="3" result="b"/>
      <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
  </defs>

  <!-- Window background -->
  <rect width="{W}" height="{H}" rx="10" fill="#0d1117"/>
  <rect x="1" y="1" width="{W-2}" height="{H-2}" rx="9"
        fill="none" stroke="#30363d" stroke-width="1"/>

  <!-- Title bar -->
  <rect width="{W}" height="32" rx="10" fill="#161b22"/>
  <rect y="22" width="{W}" height="10" fill="#161b22"/>
  <rect x="1" y="31" width="{W-2}" height="1" fill="#30363d"/>

  <!-- Traffic lights -->
  <circle cx="16" cy="16" r="5" fill="#ff5f57"/>
  <circle cx="32" cy="16" r="5" fill="#ffbd2e"/>
  <circle cx="48" cy="16" r="5" fill="#28c840"/>

  <!-- Title -->
  <text x="{W//2}" y="21" text-anchor="middle"
        font-family="monospace" font-size="11" fill="#8b949e">
    zaiba@github: ~
  </text>

  <!-- Scan line effect -->
  <rect x="{PAD_X}" y="{PAD_Y}" width="{W - PAD_X*2}" height="2"
        fill="#39d353" opacity="0.15" filter="url(#scanglow)">
    <animateTransform attributeName="transform" type="translate"
                      from="0,0" to="0,{ROWS * CHAR_H}"
                      dur="{TOTAL_DUR - 0.5:.1f}s" begin="0.3s"
                      fill="freeze" repeatCount="1"/>
  </rect>

  <!-- ASCII rows revealed row by row -->
'''
    for i, row in enumerate(ascii_rows):
        delay = 0.3 + i * ROW_DUR
        ry = PAD_Y + i * CHAR_H + CHAR_H - 3

        # Color based on brightness
        bright = sum(1 for ch in row if ch not in " .·")
        if bright > MAX_W * 0.6:
            color = "#39d353"
        elif bright > MAX_W * 0.3:
            color = "#26a641"
        else:
            color = "#0e4429"

        safe_row = row.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;")
        svg += f'''
  <text x="{PAD_X}" y="{ry}" font-family="monospace" font-size="{CHAR_H - 2}"
        fill="{color}" filter="url(#termglow)" opacity="0">
    {safe_row}<animate attributeName="opacity" values="0;1" dur="0.05s"
    begin="{delay:.2f}s" fill="freeze"/>
  </text>'''

        # Cursor sweeping across
        cy = PAD_Y + i * CHAR_H
        svg += f'''
  <rect x="{PAD_X}" y="{cy}" width="6" height="{CHAR_H}" fill="#39d353" opacity="0">
    <animate attributeName="opacity" values="0;1;1;0" dur="{ROW_DUR:.2f}s"
             begin="{delay:.2f}s" fill="freeze"/>
    <animate attributeName="x"
             from="{PAD_X}" to="{PAD_X + MAX_W * CHAR_W}"
             dur="{ROW_DUR:.2f}s" begin="{delay:.2f}s" fill="freeze"/>
  </rect>'''

    # Typewriter whoami at bottom
    whoami_y = PAD_Y + ROWS * CHAR_H + 14
    whoami_delay = 0.3 + ROWS * ROW_DUR
    prompt = f"$ whoami → {DISPLAY_NAME}"

    svg += f'''
  <!-- Prompt line -->
  <text x="{PAD_X}" y="{whoami_y}" font-family="monospace" font-size="11"
        fill="#8b949e" opacity="0">
    <animate attributeName="opacity" values="0;1" dur="0.1s"
             begin="{whoami_delay:.2f}s" fill="freeze"/>
  </text>'''

    # Typewriter char by char
    for i, ch in enumerate(prompt):
        cx2 = PAD_X + i * 7
        d = whoami_delay + i * 0.045
        color2 = "#58a6ff" if ch == "$" else ("#f0883e" if "→" in prompt[:i+1] and i >= prompt.index("→") else "#c9d1d9")
        safe_ch = ch.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;")
        svg += f'''
  <text x="{cx2}" y="{whoami_y}" font-family="monospace" font-size="11"
        fill="{color2}" opacity="0">
    {safe_ch}<animate attributeName="opacity" values="0;1" dur="0.01s"
    begin="{d:.2f}s" fill="freeze"/>
  </text>'''

    # Blinking cursor after prompt
    cursor_x = PAD_X + len(prompt) * 7 + 2
    cursor_delay = whoami_delay + len(prompt) * 0.045
    svg += f'''
  <rect x="{cursor_x}" y="{whoami_y - 10}" width="6" height="12"
        fill="#39d353" opacity="0">
    <animate attributeName="opacity" values="0;1" dur="0.1s"
             begin="{cursor_delay:.2f}s" fill="freeze"/>
    <animate attributeName="opacity" values="1;0;1" dur="1s"
             begin="{cursor_delay + 0.1:.2f}s" repeatCount="indefinite"/>
  </rect>

</svg>'''
    return svg


# ─── FILE 3: NEOFETCH INFO CARD SVG ──────────────────────────────────────────

def make_neofetch_svg():
    W, H = 340, 310

    lines = [
        ("cyan",   "         zaiba@github"),
        ("gray",   "         ─────────────────────"),
        ("orange", "  OS    ▸ B.Tech ECE · 2nd Year · 4th Sem"),
        ("blue",   "  HOST  ▸ Vemu Institute of Technology"),
        ("green",  "  CGPA  ▸ 9.8 / 10 🔥"),
        ("cyan",   "  RANK  ▸ ECET 324 (State Level)"),
        ("white",  ""),
        ("orange", "  STACK ▸ Python · C · C++ · HTML/CSS"),
        ("blue",   "         Arduino · IoT · Embedded C"),
        ("green",  "         scikit-learn · OpenCV · NumPy"),
        ("cyan",   "         VLSI Design (Learning 🔬)"),
        ("white",  ""),
        ("orange", "  WIN   ▸ 🏆 Hackathon Winner 2026"),
        ("blue",   "  BOT   ▸ 🤖 e-Yantra IIT Bombay"),
        ("green",  "  CERT  ▸ 🎓 Forage GenAI Analytics"),
        ("cyan",   "  WORK  ▸ 🏭 Hitachi Intern Bangalore"),
        ("white",  ""),
        ("orange", "  GOAL  ▸ 🚀 ISRO · DRDO · Google · AI"),
        ("gray",   ""),
        ("white",  "  ██ ██ ██ ██ ██ ██ ██ ██"),
    ]

    color_map = {
        "cyan":   "#56d4e0",
        "orange": "#f0883e",
        "blue":   "#58a6ff",
        "green":  "#39d353",
        "white":  "#c9d1d9",
        "gray":   "#484f58",
    }

    CHAR_H = 13
    PAD_X  = 14
    PAD_Y  = 36

    svg = f'''<svg xmlns="http://www.w3.org/2000/svg" width="{W}" height="{H}"
     viewBox="0 0 {W} {H}">
  <defs>
    <filter id="nglow">
      <feGaussianBlur in="SourceGraphic" stdDeviation="1.2" result="b"/>
      <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
  </defs>

  <!-- Background -->
  <rect width="{W}" height="{H}" rx="10" fill="#0d1117"/>
  <rect x="1" y="1" width="{W-2}" height="{H-2}" rx="9"
        fill="none" stroke="#30363d" stroke-width="1"/>

  <!-- Title bar -->
  <rect width="{W}" height="28" rx="10" fill="#161b22"/>
  <rect y="18" width="{W}" height="10" fill="#161b22"/>
  <rect x="1" y="27" width="{W-2}" height="1" fill="#30363d"/>
  <circle cx="14" cy="14" r="4" fill="#ff5f57"/>
  <circle cx="28" cy="14" r="4" fill="#ffbd2e"/>
  <circle cx="42" cy="14" r="4" fill="#28c840"/>
  <text x="{W//2}" y="19" text-anchor="middle"
        font-family="monospace" font-size="10" fill="#8b949e">neofetch</text>

  <!-- Zaiba ASCII small logo -->
  <text x="{PAD_X}" y="{PAD_Y}" font-family="monospace" font-size="11"
        fill="#39d353" filter="url(#nglow)" opacity="0">
    ███████╗ █╗
    <animate attributeName="opacity" values="0;1" dur="0.1s"
             begin="0.1s" fill="freeze"/>
  </text>
  <text x="{PAD_X}" y="{PAD_Y + 13}" font-family="monospace" font-size="11"
        fill="#26a641" filter="url(#nglow)" opacity="0">
    ╚══███╔╝ █║
    <animate attributeName="opacity" values="0;1" dur="0.1s"
             begin="0.15s" fill="freeze"/>
  </text>
  <text x="{PAD_X}" y="{PAD_Y + 26}" font-family="monospace" font-size="11"
        fill="#0e4429" filter="url(#nglow)" opacity="0">
    ███╔╝   ╚╝
    <animate attributeName="opacity" values="0;1" dur="0.1s"
             begin="0.2s" fill="freeze"/>
  </text>

'''
    for i, (color_key, text) in enumerate(lines):
        color = color_map[color_key]
        delay = 0.3 + i * 0.06
        ty = PAD_Y + 40 + i * CHAR_H
        safe = text.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;")
        svg += f'''  <text x="{PAD_X}" y="{ty}" font-family="monospace" font-size="10"
        fill="{color}" opacity="0" filter="url(#nglow)">
    {safe}
    <animateTransform attributeName="transform" type="translate"
                      from="0,6" to="0,0" dur="0.25s"
                      begin="{delay:.2f}s" fill="freeze"/>
    <animate attributeName="opacity" values="0;1" dur="0.25s"
             begin="{delay:.2f}s" fill="freeze"/>
  </text>\n'''

    # Color swatches at the bottom
    swatch_colors = ["#39d353","#58a6ff","#f0883e","#56d4e0","#bc8cff","#ff7b72","#f0883e","#8b949e"]
    sw_y = H - 22
    for i, sc in enumerate(swatch_colors):
        sx = PAD_X + i * 20
        delay = 0.3 + len(lines) * 0.06 + i * 0.04
        svg += f'''  <rect x="{sx}" y="{sw_y}" width="16" height="10" rx="2"
        fill="{sc}" opacity="0">
    <animate attributeName="opacity" values="0;1" dur="0.2s"
             begin="{delay:.2f}s" fill="freeze"/>
  </rect>\n'''

    svg += '</svg>'
    return svg


# ─── FILE 4: BANNER SVG ──────────────────────────────────────────────────────

def make_banner_svg():
    W, H = 860, 160

    svg = f'''<svg xmlns="http://www.w3.org/2000/svg" width="{W}" height="{H}"
     viewBox="0 0 {W} {H}">
  <defs>
    <filter id="bglow">
      <feGaussianBlur in="SourceGraphic" stdDeviation="8" result="b"/>
      <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <filter id="tglow">
      <feGaussianBlur in="SourceGraphic" stdDeviation="3" result="b"/>
      <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <linearGradient id="titleGrad" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%"   stop-color="#56d4e0"/>
      <stop offset="40%"  stop-color="#bc8cff"/>
      <stop offset="70%"  stop-color="#f0883e"/>
      <stop offset="100%" stop-color="#39d353"/>
    </linearGradient>
  </defs>

  <!-- Background -->
  <rect width="{W}" height="{H}" rx="14" fill="#0d1117"/>
  <rect x="1" y="1" width="{W-2}" height="{H-2}" rx="13"
        fill="none" stroke="#30363d" stroke-width="1"/>

  <!-- Ambient orbs -->
  <ellipse cx="120" cy="80" rx="80" ry="60"
           fill="#56d4e0" opacity="0.06" filter="url(#bglow)">
    <animate attributeName="opacity" values="0.04;0.1;0.04"
             dur="4s" repeatCount="indefinite"/>
  </ellipse>
  <ellipse cx="{W-120}" cy="80" rx="80" ry="60"
           fill="#bc8cff" opacity="0.06" filter="url(#bglow)">
    <animate attributeName="opacity" values="0.04;0.1;0.04"
             dur="5s" begin="1s" repeatCount="indefinite"/>
  </ellipse>
  <ellipse cx="{W//2}" cy="80" rx="100" ry="50"
           fill="#f0883e" opacity="0.04" filter="url(#bglow)">
    <animate attributeName="opacity" values="0.02;0.07;0.02"
             dur="6s" begin="2s" repeatCount="indefinite"/>
  </ellipse>

  <!-- Grid lines -->
  {''.join(f'<line x1="{x}" y1="0" x2="{x}" y2="{H}" stroke="#ffffff" stroke-width="0.3" opacity="0.03"/>' for x in range(0, W, 40))}
  {''.join(f'<line x1="0" y1="{y}" x2="{W}" y2="{y}" stroke="#ffffff" stroke-width="0.3" opacity="0.03"/>' for y in range(0, H, 40))}

  <!-- Name -->
  <text x="{W//2}" y="72" text-anchor="middle"
        font-family="'Courier New', monospace" font-size="42" font-weight="bold"
        fill="url(#titleGrad)" filter="url(#tglow)" opacity="0">
    {DISPLAY_NAME}
    <animate attributeName="opacity" values="0;1" dur="1.2s"
             begin="0.2s" fill="freeze"/>
    <animateTransform attributeName="transform" type="translate"
                      from="0,-20" to="0,0" dur="1.2s"
                      begin="0.2s" fill="freeze"/>
  </text>

  <!-- Subtitle -->
  <text x="{W//2}" y="100" text-anchor="middle"
        font-family="monospace" font-size="15"
        fill="#8b949e" opacity="0">
    {TITLE}
    <animate attributeName="opacity" values="0;1" dur="1s"
             begin="1s" fill="freeze"/>
  </text>

  <!-- Divider line animated -->
  <line x1="{W//2 - 200}" y1="114" x2="{W//2 + 200}" y2="114"
        stroke="url(#titleGrad)" stroke-width="1" opacity="0">
    <animate attributeName="opacity" values="0;0.6" dur="0.8s"
             begin="1.5s" fill="freeze"/>
  </line>

  <!-- Bottom tags -->
  <text x="{W//2}" y="138" text-anchor="middle"
        font-family="monospace" font-size="12" fill="#484f58" opacity="0">
    ⚡ Python  ·  C/C++  ·  VLSI  ·  IoT  ·  AI/ML  ·  Embedded Systems  ·  ISRO
    <animate attributeName="opacity" values="0;1" dur="0.8s"
             begin="1.8s" fill="freeze"/>
  </text>

  <!-- Corner decorations -->
  <text x="16" y="20" font-family="monospace" font-size="11" fill="#39d353" opacity="0.5">
    ┌──
  </text>
  <text x="{W-40}" y="20" font-family="monospace" font-size="11" fill="#39d353" opacity="0.5" text-anchor="end">
    ──┐
  </text>
  <text x="16" y="{H-8}" font-family="monospace" font-size="11" fill="#39d353" opacity="0.5">
    └──
  </text>
  <text x="{W-40}" y="{H-8}" font-family="monospace" font-size="11" fill="#39d353" opacity="0.5" text-anchor="end">
    ──┘
  </text>

</svg>'''
    return svg


# ─── GENERATE ALL FILES ──────────────────────────────────────────────────────

print("Generating banner SVG...")
with open("/mnt/user-data/outputs/zaiba-banner.svg", "w") as f:
    f.write(make_banner_svg())
print("✅ zaiba-banner.svg")

print("Generating contribution graph SVG...")
with open("/mnt/user-data/outputs/zaiba-contributions.svg", "w") as f:
    f.write(make_contribution_svg())
print("✅ zaiba-contributions.svg")

print("Generating terminal card SVG...")
with open("/mnt/user-data/outputs/zaiba-terminal.svg", "w") as f:
    f.write(make_terminal_svg())
print("✅ zaiba-terminal.svg")

print("Generating neofetch info card SVG...")
with open("/mnt/user-data/outputs/zaiba-neofetch.svg", "w") as f:
    f.write(make_neofetch_svg())
print("✅ zaiba-neofetch.svg")


# ─── GENERATE README ─────────────────────────────────────────────────────────

readme = '''<!-- ANIMATED BANNER -->
<div align="center">
<img src="./zaiba-banner.svg" alt="Shaik Zaiba Faraaz Banner" width="100%"/>
</div>

<br/>

<!-- SOCIAL BADGES -->
<div align="center">

<a href="mailto:shaikzaiba1306@gmail.com">
<img src="https://img.shields.io/badge/Gmail-shaikzaiba1306-EA4335?style=for-the-badge&logo=gmail&logoColor=white"/>
</a>
<a href="https://linkedin.com/in/shaikzaiba">
<img src="https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white"/>
</a>
<a href="https://smartstadium-ai.streamlit.app/">
<img src="https://img.shields.io/badge/🏟️ SmartStadium AI-Live-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white"/>
</a>
<img src="https://komarev.com/ghpvc/?username=faraaz-13&label=Profile+Views&color=56d4e0&style=for-the-badge"/>

</div>

<br/>

<!-- TERMINAL + NEOFETCH SIDE BY SIDE -->
<table align="center" border="0" cellspacing="0" cellpadding="8">
<tr>
<td align="center" valign="top" width="52%">
<img src="./zaiba-terminal.svg" alt="ASCII Terminal" width="100%"/>
</td>
<td align="center" valign="top" width="48%">
<img src="./zaiba-neofetch.svg" alt="Neofetch Info Card" width="100%"/>
</td>
</tr>
</table>

<br/>

<!-- CONTRIBUTION GRAPH -->
<div align="center">
<img src="./zaiba-contributions.svg" alt="Contribution Graph" width="95%"/>
</div>

<br/>

<!-- GITHUB STATS -->
<div align="center">

<img src="https://github-readme-stats.vercel.app/api?username=faraaz-13&show_icons=true&theme=radical&hide_border=true&border_radius=15&bg_color=0d1117&title_color=56d4e0&icon_color=f0883e&text_color=c9d1d9" height="180"/>
<img src="https://github-readme-stats.vercel.app/api/top-langs/?username=faraaz-13&layout=compact&theme=radical&hide_border=true&border_radius=15&bg_color=0d1117&title_color=56d4e0" height="180"/>

<br/><br/>

<img src="https://github-readme-streak-stats.herokuapp.com/?user=faraaz-13&theme=radical&hide_border=true&border_radius=15&background=0d1117&ring=56d4e0&fire=f0883e&currStreakLabel=bc8cff" width="700"/>

<br/><br/>

<img src="https://github-readme-activity-graph.vercel.app/graph?username=faraaz-13&theme=react-dark&hide_border=true&border_radius=15&bg_color=0d1117&color=56d4e0&line=bc8cff&point=f0883e&area=true" width="95%"/>

</div>

<br/>

<!-- PROJECTS TABLE -->
<div align="center">

### 🚀 Featured Projects

| Project | Tech | Highlight |
|:---|:---:|:---:|
| [🏟️ SmartStadium AI](https://smartstadium-ai.streamlit.app/) | `Python` `Groq AI` `Streamlit` | GenAI · FIFA 2026 · **Live!** |
| 🚨 AI Road Accident Prevention | `Python` `OpenCV` `ML` | Real-time · < 1s alert |
| 💡 Smart Street Lighting | `Arduino` `IR Sensors` `Embedded C` | ~40% energy saved |
| 🌱 Smart Irrigation System | `IoT` `Soil Sensors` `MCU` | Automated · Remote monitor |
| 🤖 e-Yantra Robot Nav | `Python` `CoppeliaSim` | IIT Bombay Robotics |

</div>

<br/>

<!-- CONNECT -->
<div align="center">

### 📫 Let\'s Connect

<a href="mailto:shaikzaiba1306@gmail.com">
<img src="https://img.shields.io/badge/Email-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"/>
</a>
<a href="https://github.com/faraaz-13">
<img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub"/>
</a>
<a href="https://smartstadium-ai.streamlit.app/">
<img src="https://img.shields.io/badge/Live Project-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white" alt="Project"/>
</a>

<br/><br/>

*⭐ "Two paths, one trajectory — reaching beyond boundaries." 🛸*

</div>
'''

with open("/mnt/user-data/outputs/README.md", "w") as f:
    f.write(readme)
print("✅ README.md")
print("\n🎉 ALL FILES GENERATED!")
print("Upload all 4 SVG files + README.md to your faraaz-13 GitHub profile repo!")
