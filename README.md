[nova_code (1).html](https://github.com/user-attachments/files/34702589/nova_code.1.html)
<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Nova Code</title>
<style>
  html, body { margin: 0; height: 100%; background: transparent; }
  body {
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    gap: 26px; font-family: Consolas, "Courier New", monospace; color: #5F2FD4;
    cursor: pointer; user-select: none;
  }
  .screen { position: relative; max-width: 94vw; }
  canvas {
    display: block; max-width: 94vw; height: auto;
    filter: drop-shadow(0 0 10px rgba(95, 47, 212, .6));
  }
</style>
</head>
<body>
<div class="screen"><canvas id="c"></canvas></div>

<script>
const NOVA_ART = [
  "███╗   ██╗ ██████╗ ██╗   ██╗ █████╗ ",
  "████╗  ██║██╔═══██╗██║   ██║██╔══██╗",
  "██╔██╗ ██║██║   ██║██║   ██║███████║",
  "██║╚██╗██║██║   ██║╚██╗ ██╔╝██╔══██║",
  "██║ ╚████║╚██████╔╝ ╚████╔╝ ██║  ██║",
  "╚═╝  ╚═══╝ ╚═════╝   ╚═══╝  ╚═╝  ╚═╝",
];
const CODE_ART = [
  " ██████╗ ██████╗ ██████╗ ███████╗",
  "██╔════╝██╔═══██╗██╔══██╗██╔════╝",
  "██║     ██║   ██║██║  ██║█████╗  ",
  "██║     ██║   ██║██║  ██║██╔══╝  ",
  "╚██████╗╚██████╔╝██████╔╝███████╗",
  " ╚═════╝ ╚═════╝ ╚═════╝ ╚══════╝",
];

const WC = Math.max(...[...NOVA_ART, ...CODE_ART].map(l => l.length));
const center = lines => {
  const bw = Math.max(...lines.map(l => l.length));
  const left = Math.floor((WC - bw) / 2);
  return lines.map(l => (" ".repeat(left) + l).padEnd(WC, " "));
};
const ART = [...center(NOVA_ART), " ".repeat(WC), ...center(CODE_ART)];
const HC = ART.length;

const ALPHA = "NOVACODE0123456789ABCDEFNOVA";
const GRAIN = ART.map((_, cy) => [...Array(WC)].map((_, cx) => ((cx * 7 + cy * 13) % 5 - 2) * 0.015));

const CW = 18, CHH = 36; // розмір символу в пікселях (1:2, як у терміналі)
const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");
const dpr = window.devicePixelRatio || 1;
const PX = WC * CW, PY = HC * CHH;
canvas.width = PX * dpr; canvas.height = PY * dpr;
canvas.style.width = PX + "px";
ctx.scale(dpr, dpr);
ctx.font = `bold ${CHH - 10}px Consolas, "Courier New", monospace`;
ctx.textAlign = "center";
ctx.textBaseline = "middle";

// Малюємо символи рамки ╔ ╗ ╚ ╝ ═ ║ самі, щоб лінії ідеально стикувались
const G = 4.5, T = 3; // відстань між подвійними лініями та товщина
function hl(x, y, x0, x1, cy) { ctx.fillRect(x + x0, y + cy - T / 2, x1 - x0, T); }
function vl(x, y, cx, y0, y1) { ctx.fillRect(x + cx - T / 2, y + y0, T, y1 - y0); }
function drawBox(c, x, y) {
  const cx = CW / 2, cy = CHH / 2;
  switch (c) {
    case "═": hl(x, y, 0, CW, cy - G); hl(x, y, 0, CW, cy + G); break;
    case "║": vl(x, y, cx - G, 0, CHH); vl(x, y, cx + G, 0, CHH); break;
    case "╔":
      hl(x, y, cx - G, CW, cy - G); hl(x, y, cx + G, CW, cy + G);
      vl(x, y, cx - G, cy - G, CHH); vl(x, y, cx + G, cy + G, CHH); break;
    case "╗":
      hl(x, y, 0, cx + G, cy - G); hl(x, y, 0, cx - G, cy + G);
      vl(x, y, cx + G, cy - G, CHH); vl(x, y, cx - G, cy + G, CHH); break;
    case "╚":
      hl(x, y, cx - G, CW, cy + G); hl(x, y, cx + G, CW, cy - G);
      vl(x, y, cx - G, 0, cy + G); vl(x, y, cx + G, 0, cy - G); break;
    case "╝":
      hl(x, y, 0, cx + G, cy + G); hl(x, y, 0, cx - G, cy - G);
      vl(x, y, cx + G, 0, cy + G); vl(x, y, cx - G, 0, cy - G); break;
  }
}
function drawGlyph(c, x, y) {
  if (c === "█") ctx.fillRect(x, y, CW, CHH);
  else drawBox(c, x, y);
}

function draw(level, t, reveal) {
  ctx.clearRect(0, 0, PX, PY);
  for (let cy = 0; cy < HC; cy++) {
    const b = HC - 1 - cy;
    for (let cx = 0; cx < WC; cx++) {
      const c = ART[cy][cx];
      if (c === " ") continue;
      const x = cx * CW, y = cy * CHH;
      const surface = level + 0.9 * Math.sin(cx * 0.22 + t * 6);
      if (b + 1 <= surface) {
        // залито: справжній напис кольором #5F2FD4
        const k = Math.min(1, 0.80 + 0.20 * Math.sin(cx * 0.2 - t * 5 + cy * 0.3) + GRAIN[cy][cx]);
        const depth = (b + 1) / HC;
        if (c === "█") {
          ctx.fillStyle = `rgb(${(95 + 30 * depth) * k | 0},${(47 + 25 * depth) * k | 0},${Math.min(255, (212 + 25 * depth) * k) | 0})`;
        } else {
          ctx.fillStyle = `rgb(${(72 + 20 * depth) * k | 0},${(36 + 15 * depth) * k | 0},${Math.min(255, (165 + 25 * depth) * k) | 0})`;
        }
        drawGlyph(c, x, y);
      } else if (b + 0.5 <= surface) {
        ctx.fillStyle = "rgb(195,170,255)";
        drawGlyph(c, x, y);
      } else if (cx < reveal) {
        if (c === "█") {
          // всередині літер - букви та цифри, частина мерехтить
          const flick = (cx + cy) % 3 === 0 ? Math.floor(t * 10) : 0;
          ctx.fillStyle = (cx * 7 + cy * 3) % 5 === 0 ? "rgb(125,85,235)" : "rgb(66,33,148)";
          ctx.fillText(ALPHA[(cx * 31 + cy * 17 + flick) % ALPHA.length], x + CW / 2, y + CHH / 2 + 1);
        } else {
          ctx.fillStyle = "rgb(48,25,105)";
          drawBox(c, x, y);
        }
      }
    }
  }
}

let running = 0;

function play() {
  const id = ++running;
  let t = 0, reveal = 0, level = -2, phase = 0, wait = 0;

  function step() {
    if (id !== running) return;
    if (phase === 0) {            // 1) форма проявляється зліва направо
      reveal += 0.7;
      if (reveal > WC) { phase = 1; wait = 20; }
    } else if (phase === 1) {     // пауза
      if (--wait <= 0) phase = 2;
    } else if (phase === 2) {     // 2) заливка знизу вгору
      level += 0.1;
      if (level > HC + 2) phase = 3;
    }                             // 3) далі напис просто переливається
    t += 0.03;
    draw(level, t, reveal);
    requestAnimationFrame(step);
  }
  step();
}

document.body.addEventListener("click", play);
play();
</script>
</body>
</html>
